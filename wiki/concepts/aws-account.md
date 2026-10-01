---
title: AWS Account y Root User
category: concept
tags: [fundamentos, cuenta, root-user, mfa, seguridad, multi-account]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.01 Cuenta de AWS (AWS Account).md", "raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.02 Multi-Factor Authentication (MFA).md", "raw/doc oficial/AWS account root user - AWS Identity and Access Management.md", "raw/doc oficial/AWS Multi-factor authentication in IAM - AWS Identity and Access Management.md"]
updated: 2026-10-01
---

# AWS Account y Root User

## Definición

Una **AWS Account** es un contenedor de **identidades (users)** y **recursos**. Se crea con un nombre, un **email único** (único a nivel **global**: el mismo mail no sirve para dos cuentas) y un método de pago. Recibe un **Account ID de 12 dígitos** (`123456789012`) que la identifica en los [[arn|ARNs]] y en la URL de login. La cuenta es la unidad natural de aislamiento: límite de seguridad, de facturación y de cuotas de servicio.

> [!warning] Cuenta ≠ usuario. La cuenta es el contenedor; los usuarios viven adentro.

Dos detalles prácticos del curso:
- **Plus addressing** para el mail: `micorreo+prod@gmail.com` y `micorreo+dev@gmail.com` llegan a la misma casilla pero AWS las toma como direcciones distintas — es la forma habitual de abrir varias cuentas sin crear casillas nuevas.
- **Account alias**: la URL de login por defecto usa el ID numérico (`https://123456789012.signin.aws.amazon.com/console`), que nadie recuerda. Se puede definir un alias **único a nivel global** → `https://miempresa-prod.signin.aws.amazon.com/console`.

Las identidades que creás **arrancan sin ningún permiso**: todo se concede explícitamente.

## Account Root User

- **Uno solo por cuenta**, creado con el email de la cuenta.
- **Control total, irrestringible** dentro de su cuenta (las IAM policies no lo limitan; los [[Organizations|SCPs]] sí pueden limitarlo, pero solo en member accounts).
- Regla de acceso por defecto: **todo denegado, excepto para el root user**.

**Buenas prácticas** (pregunta típica de examen):
1. Activarle **MFA** de inmediato.
2. **No usarlo** para el día a día → crear un IAM admin y trabajar con ese.
3. **No crearle access keys.**
4. Reservarlo solo para tareas que lo exigen: cerrar la cuenta, cambiar **nombre de cuenta / email raíz / password del root**, cambiar método de pago o plan de soporte, activar **MFA Delete** en S3, **restaurar los permisos del único IAM admin bloqueado**, **corregir una bucket policy (o de SQS) que deniega el acceso a todos los principals**, habilitar acceso IAM a Billing, ver ciertas **facturas e información fiscal**, registrarse como seller del RI Marketplace.
5. Configurar los **contactos alternativos** de la cuenta (billing, operations, security) — las alertas críticas no deben depender de una sola casilla.
6. Poner una **alarma de [[CloudTrail]] + [[CloudWatch]] sobre el login del root**: si el root se loguea y no fuiste vos, es un incidente de seguridad.

> Frente a los tipos de policy ([[iam-policy-evaluation]]): al root **no** se le puede attachear identity policy ni [[permissions-boundary|permissions boundary]], pero **sí** puede ser [[principal|`Principal`]] en resource policies, y **sí** lo limitan SCPs/RCPs (solo en member accounts de una organización).

> **Centralized root access** ([[Organizations]]): la management account puede **eliminar las credenciales root de las member accounts**; las cuentas nuevas creadas en la org ya **nacen sin credenciales root**. Es la evolución de "protegé el root": directamente no existe.

## Multi-Factor Authentication (MFA)

Un **factor** es una pieza de evidencia de identidad. Más factores = más seguridad.

| Factor | Qué es | Ejemplo |
|---|---|---|
| **Knowledge** | Algo que sabés | password, PIN |
| **Possession** | Algo que tenés | dispositivo MFA, token |
| **Inherent** | Algo que sos | huella, rostro |
| **Location** | Dónde estás | red corporativa |

![[Pasted image 20260613233140.png]]

Cómo funciona en AWS (TOTP): AWS genera una **secret key** → QR → la app del teléfono la guarda → la app genera códigos que rotan cada ~30s. **Se envía el código, nunca la key.**

> Datos de la doc oficial: hasta **8 dispositivos MFA por usuario**; se soportan virtual MFA (app), hardware TOTP, **security keys FIDO/passkeys**. El SMS ya no se soporta.

Matiz sobre el factor **Location**: en AWS *no* se implementa como MFA, pero existe en la capa de **autorización** — `aws:SourceIp` y `aws:SourceVpce` en las [[iam-policy-evaluation|policies]] restringen desde dónde se puede operar.

### Si se pierde el dispositivo MFA

| Identidad | Cómo se recupera |
|---|---|
| **IAM user** | Cualquier admin de la cuenta le desactiva el MFA y lo reconfigura — trámite de dos minutos |
| **Root user** | No hay administrador por encima → **proceso de recuperación de AWS** (verifica mail y teléfono de la cuenta). Lleva tiempo y exige contactos actualizados |

De ahí la práctica de registrar **más de un dispositivo** en el root (el teléfono y una llave física guardada en otro lado).

### Exigir MFA por policy (dato de examen)

El MFA se **exige**, no solo se ofrece. La condition key es **`aws:MultiFactorAuthPresent`**:

```json
{
  "Effect": "Deny",
  "Action": "ec2:TerminateInstances",
  "Resource": "*",
  "Condition": { "BoolIfExists": { "aws:MultiFactorAuthPresent": "false" } }
}
```

Enunciado típico: *"¿cómo garantizo que ciertas acciones solo se ejecuten con MFA?"* → **una condition en la policy**, no una configuración del usuario. La variante **`aws:MultiFactorAuthAge`** exige que la autenticación haya sido hace menos de X segundos.

### MFA y la CLI — la trampa

**La CLI no pide MFA automáticamente**: el MFA aplica a la consola, y las access keys siguen funcionando sin él. Para que la CLI lo respete hay que pedir credenciales temporales:

```bash
aws sts get-session-token \
  --serial-number arn:aws:iam::123456789012:mfa/mi-usuario \
  --token-code 123456
```

…y usar el trío que devuelve (key + secret + **session token**). Sin la condition `aws:MultiFactorAuthPresent`, un atacante con la access key robada **opera sin MFA aunque el usuario lo tenga configurado**.

## Por qué usar múltiples cuentas

Las cuentas contienen el **[[blast-radius|blast radius]]** de errores y exploits. Recomendación: cuentas separadas por entorno (`DEV`, `TEST`, `PROD`), equipo o cliente — gestionadas con [[Organizations]].

| Motivo | Por qué |
|---|---|
| **Aislamiento de seguridad** | Una credencial comprometida en `DEV` no toca `PROD`. Es una separación **más fuerte que cualquier policy** |
| **Separación de costos** | Cada cuenta factura aparte → saber cuánto gasta cada equipo es trivial, sin depender de tags bien puestos |
| **Límites y cuotas** | Los service quotas son **por cuenta y por region** → un equipo no consume la cuota de otro |
| **Compliance** | Una cuenta con requisitos regulatorios estrictos sin arrastrar al resto |
| **Radio de error** | Un `terraform destroy` en la cuenta equivocada duele mucho menos si esa cuenta solo tiene pruebas |

![[Pasted image 20260613230446.png]]

## Preguntas de examen frecuentes

- "¿Qué hacer primero al crear una cuenta?" → MFA en root + crear IAM admin para uso diario.
- "¿Quién puede activar MFA Delete en S3?" → solo credenciales **root** (vía CLI/API).
- "¿Cómo limito al root user?" → dentro de una cuenta suelta, imposible; en una organización, con **SCPs** (solo member accounts).
- "Exigir MFA para operaciones destructivas" → **Deny + `BoolIfExists` sobre `aws:MultiFactorAuthPresent: false`**.
- "El usuario tiene MFA pero la CLI no lo pide" → correcto, es esperado: hay que usar `sts get-session-token` y exigirlo por policy.
- "Aislar entornos al máximo" → **cuentas separadas**, más fuerte que cualquier policy. Recordar que los **quotas son por cuenta y region**.

## Demos del curso

- [Creando un AWS Account](https://learn.cantrill.io/courses/1101194/lectures/63958659)
- [Adding MFA General Account Root User](https://learn.cantrill.io/courses/1101194/lectures/63972849)
