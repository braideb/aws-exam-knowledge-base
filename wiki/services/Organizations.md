---
title: AWS Organizations
category: service
tags: [organizations, scp, multi-account, consolidated-billing, gobernanza]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.10 AWS Organizations.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.11 Service Control Policies (SCP).md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.14 Precios.md", "raw/doc oficial/Terminology and concepts for AWS Organizations.md", "raw/doc oficial/Service control policies (SCPs).md", "raw/doc oficial/SCP evaluation - AWS Organizations.md"]
updated: 2026-07-25
---

# AWS Organizations

## ¿Qué es?

Servicio para **administrar múltiples cuentas AWS** como una sola organización: facturación consolidada, gestión centralizada de identidades y **restricciones globales (SCPs)**. Sin él, 30 cuentas = 30 juegos de credenciales y 30 facturas.

## Características clave

### Estructura

```
Organization
└── Organization Root          ← contenedor superior (NO es una cuenta)
    ├── Member Account
    └── Organizational Unit (OU)   ← anidables
        └── Member Accounts
```

Datos de la doc oficial: OUs anidables hasta **5 niveles de profundidad** (root incluido); la management account **no se puede cambiar** una vez creada la org; las invitaciones son **handshakes** (aceptar/rechazar). **Delegated administrator**: la management account puede delegar la administración de policies o servicios integrados a una member account — otra razón para mantenerla vacía (ver [[aws-account]]).

> Matiz sutil: la organización **no vive "dentro" de la management account**. Esa cuenta solo se usa para *crearla*; una vez creada, la organización es una entidad aparte.

**Crear una cuenta vs. invitarla** — la diferencia se pregunta:

| | **Creada desde la organización** | **Invitada** |
|---|---|---|
| `OrganizationAccountAccessRole` | Se crea **automáticamente** | Hay que **crearlo a mano** |
| Password del root user | **No se define ninguna** → hay que hacer *forgot password* para usarla | Ya la tiene |
| Salir de la organización | Requiere completar información de contacto y facturación primero | Igual |

**Regla de diseño de las OUs:** no se organizan por departamento sino **por lo que se les va a aplicar**. Si dos cuentas necesitan las mismas restricciones, van en la misma OU.

> [!danger] Tres "root" distintos — fuente infinita de confusión:
> - **Organization Root** = contenedor jerárquico.
> - **Management Account** = la cuenta que creó la org (👑, también "payer account"; antes "master account").
> - **Account Root User** = el usuario todopoderoso de cada cuenta individual.

![[Pasted image 20260707005920.png]]

### Consolidated Billing

Las member accounts pierden su método de pago; todo se factura a la **management account** (por eso también se la llama **payer account**). Beneficio real: el consumo se **suma**, así que:

- **Descuentos por volumen**: los tiers de precio se calculan sobre el **total sumado**, no por cuenta. Cinco cuentas que individualmente no alcanzan el tier barato de S3, sumadas sí llegan — y **todas** pagan el precio menor.
- **Reserved Instances y Savings Plans compartidos**: si una cuenta compró capacidad reservada y no la usa toda, otra cuenta de la org aprovecha el descuento.
- Posibilidad de **pagos por adelantado** para descuentos grandes.

> El reparto de descuentos entre cuentas se puede **desactivar por cuenta**, si querés que cada equipo vea su costo real sin subsidios cruzados.

> **Organizations y los SCPs son gratis** — igual que [[IAM]] y STS. Gobernar 30 cuentas no tiene costo propio; se paga solo el consumo de cada una ([[observability-costs]]).

### Gestión de identidades

Con Organizations no hacen falta IAM users en cada cuenta: una **cuenta de login** (*identity account*, con IAM users o federación con AD) + **switch role** hacia las demás. "Switch role" = `sts:AssumeRole` sobre un rol de la otra cuenta, con botón lindo.

El flujo real:
1. La persona se loguea en la **Login-Account** → credenciales válidas **solo** ahí.
2. Hace "Switch Role" indicando cuenta destino + nombre del rol (ej. `Developer`).
3. Por detrás se dispara `sts:AssumeRole` contra `arn:aws:iam::PROD_ID:role/Developer`.
4. Ese rol necesita una **[[trust-policy]] que confíe en la Login-Account** (o en un user/role puntual de ella).
5. STS devuelve [[temporary-credentials|credenciales temporales]] nuevas, con los permisos del rol, típicamente por 1 h.
6. Opera en Prod **sin haber tenido nunca un usuario propio ahí**.

Es el mismo mecanismo que usa Lambda con su execution role ([[IAM]]); la única diferencia es que ahí el `Principal` de la trust policy era un servicio y acá es otra cuenta.

![[Pasted image 20260707010344.png]]

## Service Control Policies (SCPs)

Documento JSON attachable al **Root de la org, a una OU o a una cuenta**; se **hereda hacia abajo**.

**Lo más importante: los SCPs NO conceden permisos.** Son un **techo** (account [[permissions-boundary|permissions boundary]]). El acceso efectivo = **intersección** entre identity policies y SCPs ([[iam-policy-evaluation]]).

> Analogía: el SCP es el reglamento del edificio ("prohibido el subsuelo"); la identity policy es tu contrato ("Juan puede ir al subsuelo"). Juan **no entra**: necesita que ambos lo permitan.

![[Pasted image 20260707024117.png]]

> [!danger] La management account **NUNCA es afectada por SCPs** — por eso la buena práctica es mantenerla **vacía**, sin recursos ni usuarios de trabajo.

Los SCPs **sí limitan al root user de las member accounts** — es el **único mecanismo de AWS capaz de limitar a un root user**. Ninguna IAM policy puede. Por eso son la herramienta de gobierno por excelencia.

Requieren la org en modo **"all features"**; pasar de *consolidated billing only* exige que **todas las cuentas miembro lo aprueben**.

### SCPs que aparecen en casi toda organización real

```json
// 1. Nadie apaga la auditoría — probablemente el SCP más importante que existe:
//    sin él, un atacante con permisos de admin apaga CloudTrail y se vuelve invisible.
{ "Effect": "Deny",
  "Action": ["cloudtrail:StopLogging", "cloudtrail:DeleteTrail"],
  "Resource": "*" }

// 2. Restricción de regiones — hay que exceptuar los servicios globales
//    o se rompe IAM (técnicamente se invocan contra us-east-1).
{ "Effect": "Deny",
  "NotAction": ["iam:*", "s3:*", "cloudfront:*", "route53:*", "support:*"],
  "Resource": "*",
  "Condition": { "StringNotEquals": { "aws:RequestedRegion": ["us-east-1", "sa-east-1"] } } }

// 3. Que una cuenta no pueda escaparse de la organización (y de sus controles).
{ "Effect": "Deny", "Action": ["organizations:LeaveOrganization"], "Resource": "*" }
```

Los SCPs aceptan condiciones; la más usada para excepciones es **`aws:PrincipalArn`** ("denegar a todos menos al rol de automatización").

Otro ejemplo típico: **limitar el tamaño máximo de instancia [[EC2]]**. Así, aunque un dev tenga `AdministratorAccess` en su cuenta, no puede lanzar una instancia de $30/hora "para probar" — y **ni el root user de esa cuenta puede saltárselo**.

### Límites (memorizar)

| Límite | Valor |
|---|---|
| SCPs adjuntos por entidad (root/OU/cuenta) | **5** |
| Tamaño de un SCP | **5.120 caracteres** |
| Profundidad de anidamiento de OUs | **5 niveles** |

El límite de tamaño se alcanza rápido al enumerar servicios — otra razón por la que la **deny list** resulta más práctica.

### Cómo diagnosticar un bloqueo por SCP

El error es un **`AccessDenied` genérico que NO menciona el SCP** (por diseño, para no filtrar la estructura de la organización). Las herramientas:

- ⚠️ **El IAM Policy Simulator NO evalúa SCPs** — no sirve para esto.
- En la consola de Organizations, cada cuenta muestra sus **políticas efectivas** heredadas.
- Prueba rápida: si la acción **funciona desde la management account** y falla en la member account, con casi total seguridad es un SCP (la management account está exenta).

### Deny List vs. Allow List

| | Deny List (default) | Allow List |
|---|---|---|
| Base | `FullAWSAccess` (`Allow` `*` sobre `*`, que AWS adjunta solo) + Denies puntuales | Se quita `FullAWSAccess`, se enumera lo permitido → el resto cae en **Default Deny** |
| Administración | **Baja** ✅ recomendada | Alta |
| Riesgo | Servicio nuevo queda permitido | Bloquear algo necesario |

Gotcha fino (doc oficial): en allow list hace falta un **Allow en CADA nivel** del árbol (root → OU → cuenta); si quitás `FullAWSAccess` de un nivel sin reemplazo, todo lo de abajo queda bloqueado.

### Cómo se evalúan los SCPs en el árbol

Las dos reglas (doc oficial, con escenarios de examen):

1. **Allow**: para que un permiso llegue a una cuenta debe haber un Allow explícito **en cada nivel** del camino directo root → OU(s) → cuenta. Deny-by-default: lo no listado queda denegado. `FullAWSAccess` se adjunta automáticamente a cada root/OU/cuenta al crearse — por eso el modo deny list "funciona solo".
2. **Deny**: alcanza con un Deny **en cualquier nivel** del camino para bloquear a todo lo que cuelga debajo — un Deny en el root afecta a toda la organización aunque una OU inferior tenga un Allow explícito para lo mismo.

Consecuencia práctica: un Allow en una OU **nunca "recupera"** lo denegado (o no permitido) más arriba — la evaluación entre niveles es siempre **intersección**.

- Recomendación oficial: **no** attachear SCPs al root sin probar antes en una OU con pocas cuentas; usar **service last accessed data** de IAM para detectar qué servicios se usan realmente antes de restringir.
- ⚠️ Si deshabilitás el policy type SCP en el root, **todos los SCPs se desattachean de todo**; al rehabilitarlo, vuelve solo `FullAWSAccess` — los attachments anteriores **se pierden** (hay que rearmarlos a mano).

## Integración con otros servicios

- [[IAM]] / [[iam-policy-evaluation]] — intersección SCP ∩ identity policies.
- [[CloudTrail]] — **organizational trail**: auditoría de todas las cuentas en un solo lugar.
- Otras policies de la org: **RCPs** (techo para resource policies), Tag/Backup/declarative policies.

## Gotchas y trampas del examen

- SCP en escenario "aunque el usuario tenga AdministratorAccess, no puede X" → correcto, el SCP es techo.
- Los SCPs no afectan a la management account **ni a los service-linked roles**.
- SCPs requieren la org en modo **"all features"** (no solo consolidated billing).
- Restringir regiones por SCP → siempre exceptuar servicios globales (IAM, STS, CloudFront) o se rompe todo.
- Los SCPs **no afectan a principals de cuentas EXTERNAS a la org**: si una bucket policy de la cuenta A da acceso a users de una cuenta B externa, el SCP de A no los limita (solo limita a las identidades **de** las cuentas miembro). Sí aplican a los **delegated administrators** (siguen siendo member accounts).
- Con permissions boundary + SCP + identity policy, **las tres** deben permitir la acción ([[iam-policy-evaluation]]).
- "Impedir que alguien apague la auditoría" → **SCP con Deny sobre `cloudtrail:StopLogging`/`DeleteTrail`**, no permisos IAM.
- "AccessDenied que no dice por qué" en una member account → probar la misma acción desde la **management account**; si ahí anda, es un SCP.
- El **Policy Simulator no ve SCPs** — distractor si la pregunta ofrece esa herramienta para diagnosticar un bloqueo organizacional.
- Otras cosas que un SCP no puede restringir: registrarse al plan Enterprise support como root, trusted signer de CloudFront.

## Demos del curso

- [AWS Organizations](https://learn.cantrill.io/courses/1101194/lectures/25362688)
- [Using Service Control Policies](https://learn.cantrill.io/courses/1101194/lectures/25362691)
