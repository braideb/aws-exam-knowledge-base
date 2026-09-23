---
title: Evaluación de IAM Policies
category: concept
tags: [iam, policies, seguridad, deny, allow, permissions-boundary, scp]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.01 IAM Identity Policies.md", "raw/doc oficial/Policies and permissions in AWS Identity and Access Management - AWS Identity and Access Management.md", "raw/doc oficial/Policy evaluation logic - AWS Identity and Access Management.md", "raw/doc oficial/Permissions boundaries for IAM entities - AWS Identity and Access Management.md"]
updated: 2026-09-19
---

# Evaluación de IAM Policies

## Definición

Una **policy** es un documento JSON con statements que **conceden (Allow)** o **deniegan (Deny)** acciones sobre recursos. Por sí sola no hace nada: tiene efecto al estar **attached** a una identidad o recurso.

## Anatomía de un statement

```json
{
    "Sid": "DenyCatBucket",
    "Effect": "Deny",
    "Action": ["s3:*"],
    "Resource": ["arn:aws:s3:::catgifs", "arn:aws:s3:::catgifs/*"]
}
```

| Campo | Qué hace |
|---|---|
| `Sid` | Etiqueta opcional para humanos |
| `Effect` | `Allow` o `Deny` |
| `Action` | Operaciones (`s3:*`, `s3:GetObject`) |
| `Resource` | [[arn\|ARNs]] afectados |
| [[principal\|`Principal`]] | **Solo en resource policies** — a quién aplica |
| `Condition` | Lógica extra (IP, MFA, tags…) |

> `"Version": "2012-10-17"` es la versión del **lenguaje** de policies, no de tu documento. Nunca cambia.

### El campo `Condition` en detalle

```json
"Condition": { "OPERADOR": { "clave": "valor" } }
```

| Uso | Condition |
|---|---|
| Solo desde la red de la oficina | `"IpAddress": {"aws:SourceIp": ["203.0.113.0/24"]}` |
| Solo si se autenticó con MFA | `"BoolIfExists": {"aws:MultiFactorAuthPresent": "true"}` |
| Solo en ciertas regiones | `"StringEquals": {"aws:RequestedRegion": ["us-east-1","sa-east-1"]}` |
| Solo recursos etiquetados `dev` | `"StringEquals": {"aws:ResourceTag/Environment": "dev"}` |

Operadores más usados: `StringEquals`, `StringLike` (acepta comodines), `ArnLike`, `IpAddress`, `NumericLessThan`, `DateGreaterThan`, `Bool`.

> Los que terminan en **`IfExists`** **no fallan si la clave no está en la request** — importante para no bloquear sin querer llamadas de servicios internos de AWS, donde parte del contexto viene redactado.

### ABAC — permisos por tags

La última fila de la tabla es la base del **[[abac|ABAC]]** (Attribute-Based Access Control): en vez de enumerar recursos uno por uno, se conceden permisos **por tag**. Escala muchísimo mejor que el RBAC clásico — una sola policy cubre recursos que todavía no existen, siempre que lleven el tag correcto.

### Variables de policy

Se resuelven en tiempo de evaluación. La más usada es `${aws:username}`:

```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::empresa-home/${aws:username}/*"
}
```

**Una sola policy que le da a cada usuario acceso únicamente a su propia carpeta.** Sin variables habría que escribir una policy por persona.

### `NotAction` y `NotResource`

Las versiones negadas de los campos. Se usan poco pero aparecen:

```json
{ "Effect": "Deny", "NotAction": ["s3:*", "cloudwatch:*"], "Resource": "*" }
```

Se lee "denegar todo excepto S3 y CloudWatch". **Son peligrosas**: incluyen implícitamente los servicios que AWS lance en el futuro, así que es fácil terminar con permisos más amplios (o más restrictivos) de lo que uno cree. Evitarlas salvo motivo claro.

## La regla que hay que memorizar

```
1. ¿Explicit DENY?  → DENEGADO. Fin.
2. ¿Explicit ALLOW? → PERMITIDO.
3. Nada             → Default DENY (implícito).
```

El DENY explícito gana **siempre**, sin importar el orden de los statements.

![[Pasted image 20260705155709.png]]

## Cómo se combinan los distintos tipos de policy

| Tipos combinados | Operación |
|---|---|
| Identity policy + **Resource policy** (misma cuenta) | **Unión** (basta que una permita) |
| Identity policy + Resource policy (**[[cross-account]]**) | Se necesitan **ambas** |
| Identity policy + **[[permissions-boundary\|Permissions boundary]]** | **Intersección** |
| Identity policy + **SCP** ([[Organizations]]) | **Intersección** |
| + **Session policy** (al asumir rol) | **Intersección** con la sesión |

Y sobre todo eso: cualquier Deny explícito mata el acceso.

### La cadena completa, como método de diagnóstico

La regla de tres pasos alcanza para la mayoría de las preguntas. La versión completa, **en orden**, es la que se usa para debuggear un "esto debería funcionar y no funciona":

1. **Deny explícito** en cualquier capa → denegado, se termina la evaluación.
2. **SCP** (si la cuenta está en una organización) → tiene que permitirlo.
3. **Resource policy** → si permite explícitamente, puede alcanzar por sí sola (misma cuenta).
4. **Permissions boundary** → si existe, la acción tiene que estar dentro del techo.
5. **Session policy** → si las credenciales vienen de STS con una, también tiene que permitirlo.
6. **Identity policy** → tiene que permitirlo.
7. Si nada permitió → **[[implicit-deny|deny implícito]]**.

> La forma corta: **cualquier capa puede denegar; todas las capas relevantes tienen que permitir.**

Dos herramientas para no hacerlo a ojo:
- **IAM Policy Simulator**: se elige identidad + acción + recurso y dice si se permite **y qué policy tomó la decisión**.
- **IAM Access Analyzer** ([[IAM]]): valida policies mientras las escribís y genera policies a partir de la actividad real de [[CloudTrail]].

La doc oficial enumera **nueve tipos** de policy. Además de los de la tabla: **RCPs** (techo para resource policies, primo de los SCPs), **VPC endpoint policies** (boundary sobre el tráfico que cruza un endpoint de [[VPC]]; el default permite todo), **ACLs** (el único tipo **no-JSON**, solo controla acceso **cross-account**) y **AWS RAM resource shares**. Detalle fino: las resource-based policies son **siempre inline** — no existen "managed resource policies".

## Permissions boundaries en detalle

Una **permissions boundary** es una managed policy (AWS o customer) que fija el **techo** de lo que las identity policies de un user/rol pueden conceder. **No concede nada por sí sola**: efectivo = identity ∩ boundary. Si además hay SCP: **las tres capas deben permitir** (identity ∩ boundary ∩ SCP). Con session policies pasa lo mismo: intersección de todo.

- **Caso de uso estrella (examen): delegación segura.** Un admin delega la creación de users a alguien, exigiendo por policy que todo user creado lleve cierta boundary — condition **`iam:PermissionsBoundary`** sobre `iam:CreateUser`/`iam:AttachUserPolicy`, más Denies a `iam:DeleteUserPermissionsBoundary` y a editar la policy de boundary. Así el delegado no puede crear identidades más poderosas que el techo.
- **Matiz con resource policies** (fino, cae en pro-level): una resource policy que da acceso **directamente al ARN de un user** (misma cuenta) **no queda limitada por el implicit deny de la boundary** — la boundary solo recorta lo que conceden las *identity* policies. Lo mismo con permisos otorgados al ARN de una **role session**. Un **explicit Deny en la boundary sí gana siempre**, sea cual sea el origen del Allow.
- La boundary aplica a **users y roles**, no a grupos ni a service-linked roles (ver [[IAM]]).

## Session policies

Policy opcional que se pasa **al crear la sesión** (`AssumeRole`/`GetFederationToken`) para recortarla aún más: la sesión obtiene la **intersección** entre las policies de la identidad y la session policy. Útil para emitir credenciales temporales con menos permisos que el rol completo.

## Managed vs. Inline

| | Managed | Inline |
|---|---|---|
| Se define | Una vez, objeto independiente | JSON pegado en cada identidad |
| Cambios | Un solo lugar, se propaga | Hay que editar cada copia |
| Versionado | Sí (hasta 5, con rollback) | No |
| Cuándo | **Default** | Excepciones de una sola identidad |

Subtipos de managed: **AWS Managed** (las mantiene AWS, tienden a ser demasiado amplias) y **Customer Managed** (las tuyas, preferidas para producción).

Dos características de las **AWS Managed** que conviene tener presentes:
- **No se pueden editar.** Si querés una variante, la copiás a una customer managed.
- **AWS las actualiza sola** cuando salen servicios nuevos → conveniencia y riesgo a la vez: los permisos de tus identidades **pueden ampliarse sin que hagas nada**. Para [[least-privilege|mínimo privilegio]] estricto, customer managed.

Una subcategoría útil: las **job function policies** de AWS (`DataScientist`, `NetworkAdministrator`, `Billing`, `SupportUser`), pensadas como punto de partida para roles típicos de una organización.

Ventaja secundaria de las **inline**: **viven y mueren con la identidad**. Si borrás el user, la policy se va con él y no queda basura. Con managed policies hay que acordarse de limpiar las que quedaron sin usar.

![[Pasted image 20260705160500.png]]

## Preguntas de examen frecuentes

- Allow amplio + Deny puntual sobre el mismo recurso → gana el **Deny**.
- "La identity policy lo permite pero el SCP no" → **denegado** (intersección).
- ¿Cómo identificar una resource policy? → tiene campo **`Principal`**.
- Permissions boundary define el **techo**, no concede permisos (igual que los SCPs).
- "Delegar creación de users sin permitir escalación de privilegios" → boundary obligatoria vía condition `iam:PermissionsBoundary`.
- "Dar acceso solo a los recursos del entorno `dev` sin listarlos uno por uno" → **[[abac|ABAC]]** con `aws:ResourceTag`.
- "Cada usuario accede solo a su carpeta del bucket, con una sola policy" → variable **`${aws:username}`** en el `Resource`.
- "Restringir a ciertas regiones" → condition `aws:RequestedRegion` (y exceptuar servicios globales).
- "¿Qué policy está bloqueando esta acción?" → **IAM Policy Simulator**.
- Cuidado con los operadores **sin** `IfExists` en Denies por red: si la clave no viene en la request, la condition no matchea y el Deny no aplica (o al revés, bloquea servicios de AWS).

## Demos del curso

- [Simple Identity Permissions in AWS](https://learn.cantrill.io/courses/1101194/lectures/25335803)
