---
title: IAM (Identity and Access Management)
category: service
tags: [iam, seguridad, users, groups, roles, sts, federacion, access-keys]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.03 IAM — Conceptos básicos.md", "raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.04 IAM Access Keys.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.02 IAM Users.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.04 Restricciones y datos útiles de IAM.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.05 IAM Groups.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.06 IAM Roles.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.07 Cuándo usar IAM Roles - los cinco escenarios.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.08 Service-Linked Roles.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.09 Security Token Service (STS).md", "raw/doc oficial/IAM users - AWS Identity and Access Management.md", "raw/doc oficial/IAM roles - AWS Identity and Access Management.md", "raw/doc oficial/Temporary security credentials in IAM - AWS Identity and Access Management.md", "raw/doc oficial/Using AWS Identity and Access Management Access Analyzer - AWS Identity and Access Management.md"]
updated: 2026-09-23
---

# IAM — Identity and Access Management

## ¿Qué es?

Servicio de **identidades y permisos** de AWS. Cada cuenta tiene su propia base de IAM, independiente de las demás. Es **global** (no se elige region), **[[globally-resilient]]**, **gratis**, y soporta federación y MFA. Lo que IAM autoriza, la cuenta lo permite.

Sus tres funciones: **Identity Provider** (crear/modificar/borrar identidades) · **Authentication** (verificar quién sos) · **Authorization** (decidir qué podés hacer).

> ⚠️ **IAM es [[eventual-consistency|eventually consistent]]** — al ser global, replica a todas las regiones. Crear un rol o cambiar una policy tarda segundos en propagarse: un script que crea un rol y lo usa **inmediatamente** puede fallar la primera vez y funcionar al reintentar. No es un bug, es la propagación.

## Casos de uso

- Crear identidades restringibles (el [[aws-account|root user]] no se puede restringir; todo lo demás sí).
- Dar acceso a humanos, aplicaciones y servicios con **mínimo privilegio** ([[least-privilege]]) — se construye **al revés de la intuición**: arrancar **sin permisos** y agregar según aparecen los *access denied*, en vez de arrancar con `AdministratorAccess` y prometerse recortarlo (nunca pasa).
- [[federation|Federar identidades externas]] (Active Directory, Google/Facebook) mediante roles.

## Características clave

### Principal → Authentication → Authorization

1. **[[principal|Principal]]**: quien hace la solicitud (aún no se sabe quién es).
2. **Authentication**: se autentica con **username+password** (consola) o **access keys** (CLI/API) → Authenticated Identity.
3. **Authorization**: IAM evalúa las [[iam-policy-evaluation|policies]] asociadas y decide.

![[Pasted image 20260705161353.png]]

### Las tres entidades

| Entidad | Qué es | Cuándo |
|---|---|---|
| **User** | Un humano o app concreta | Cantidad de identidades **conocida** |
| **Group** | Contenedor de users | Asignar policies a muchos users |
| **Role** | Identidad que se **asume** temporalmente | Cantidad **incierta** o servicios AWS |

**Permisos efectivos de un user** = **unión** de sus policies inline + sus managed + las policies de **todos** los grupos a los que pertenece. Y sobre esa unión, cualquier **Deny explícito gana**. Ejemplo: Ana está en `Desarrolladores` (`s3:*`) y en `Contratistas` (Deny sobre el bucket de finanzas) → puede todo en S3 **excepto** ese bucket, sin importar el orden.

**Groups — trampas de examen**: no se puede loguear a un grupo, no hay grupo "todos" por defecto, **no se anidan** (jerarquía plana; para simularla hay que duplicar policies o poner a la persona en varios grupos, hasta 10), y **no son identidades** → no pueden ir como `Principal` en una resource policy **ni** en una trust policy.

> **El patrón para "que un grupo pueda asumir un rol"** (porque no se puede listar el grupo en la trust policy): se le da al **grupo** una policy que permite `sts:AssumeRole` sobre el ARN del rol, y la **trust policy del rol confía en la cuenta**. Así, agregar a alguien al grupo le habilita el rol.

Cuando el modelo de grupos planos se vuelve un problema, las dos salidas son **roles con federación** (los grupos reales viven en el directorio corporativo) o **IAM Identity Center** (grupos + *permission sets*, pensado para escala).

Detalle de **password policy**: si activás expiración de passwords, hay que dar al user permiso de **`iam:ChangePassword` sobre sí mismo**, o queda encerrado cuando le vence. Y **un user por persona**: si dos humanos comparten credenciales, [[CloudTrail]] deja de servir para saber quién hizo qué.

### IAM Roles

> "IAM Roles are assumed… you become that role." El rol representa un **nivel de acceso**, no una persona.

| Aspecto | IAM User | IAM Role |
|---|---|---|
| Cuántos lo usan | **Un solo principal** | Número **desconocido/ilimitado** |
| Duración | **Largo plazo** — vos *sos* ese user | **Corto plazo** — vos *te convertís* en el rol |
| Credenciales | Permanentes (password, access keys) | **Temporales**, expiran y se renuevan |

> Analogía del curso: el **user** es tu credencial de empleado (tu foto, tu nombre, solo vos la usás). El **rol** es el **casco naranja de visitante** colgado en la entrada: no tiene nombre, cualquiera autorizado se lo pone, y lo devuelve al terminar.

Dos policies por rol:
- **[[trust-policy|Trust policy]]** → quién puede asumirlo (identidades de la cuenta, otras cuentas, servicios AWS, identidades federadas). Es **un muro alrededor del rol**: si el principal no está permitido, `AssumeRole` **falla**, punto.
- **Permissions policy** → qué puede hacer quien lo asume.

Para no confundirlas: la **trust policy mira hacia afuera** (quién entra), la **permissions policy mira hacia adentro** (qué puede tocar).

```json
// Trust policy que confía en el SERVICIO EC2
{ "Effect": "Allow",
  "Principal": { "Service": "ec2.amazonaws.com" },
  "Action": "sts:AssumeRole" }

// Trust policy que confía en OTRA CUENTA
{ "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::111122223333:root" },
  "Action": "sts:AssumeRole" }
```

El `:root` del segundo ejemplo **no es el Account Root User**: significa "la cuenta 111122223333 completa", o sea que se **delega en esa cuenta** la decisión de qué identidades suyas pueden asumir el rol ([[arn]]).

> ⚠️ **Los roles SÍ se pueden referenciar en resource policies** (a diferencia de los grupos). Consecuencia: si un rol puede acceder a un bucket —por su permissions policy o por la resource policy del bucket— entonces **cualquiera que asuma ese rol accede a ese recurso**.

Al asumir un rol, **STS** (`sts:AssumeRole`) entrega **[[temporary-credentials|credenciales temporales]]** (AccessKeyId, SecretAccessKey, **SessionToken**, Expiration) que caducan y se renuevan.

Datos de STS de la doc oficial: es un servicio **global** y **gratis** (endpoint default `sts.amazonaws.com`) pero con **endpoints regionales** opcionales para bajar latencia — las credenciales **funcionan globalmente** sin importar dónde se emitieron. Las temporales no se almacenan con el user: se **generan on-demand**, y expiran solas (no hay que revocarlas una a una). Para apps móviles la recomendación oficial es **Cognito** (soporta los mismos IdPs + acceso guest no autenticado).

**La familia de operaciones de STS** — cada escenario tiene la suya:

| Operación | Quién la llama | Para qué |
|---|---|---|
| **`AssumeRole`** | Una identidad de AWS (user o role) | El caso general: misma cuenta, [[cross-account]], servicios |
| **`AssumeRoleWithSAML`** | Identidad de un directorio corporativo | [[federation\|Federación]] empresarial (AD FS, Okta, Entra ID) |
| **`AssumeRoleWithWebIdentity`** | Identidad de un proveedor OIDC | Login social, Cognito, y **los pods de EKS** |
| **`GetSessionToken`** | Un IAM user | Credenciales temporales **de sí mismo**, típicamente para cumplir MFA |
| **`GetFederationToken`** | Un IAM user | Federar usuarios **sin rol**, con permisos acotados |
| **`GetCallerIdentity`** | Cualquiera | "¿Quién soy?" — **no requiere permisos** |

Las tres primeras son las que hay que reconocer en el examen. `GetCallerIdentity` es la de uso diario: verificar con qué identidad estás operando **antes** de ejecutar algo destructivo.

> **Se puede pedir menos de lo que el rol permite**: al asumir se pasa una **session policy** (`--policy`), y los permisos efectivos son la **intersección**. Nunca amplía, solo recorta. Si el script solo necesita leer un bucket, pedí credenciales que solo puedan eso — si se compromete, el daño está acotado ([[iam-policy-evaluation]]).
>
> ```bash
> aws sts assume-role --role-arn arn:aws:iam::123456789012:role/MiRol \
>   --role-session-name juan-backup --policy file://solo-lectura.json
> ```

![[Pasted image 20260705173934.png]]

**Los 5 escenarios de uso de roles:**
1. **Servicio AWS actúa por vos** (Lambda execution role, [[instance-profile|instance role]] de EC2) — nunca hardcodear access keys.
2. **[[break-glass|Break glass]]**: acceso de emergencia temporal y auditado.
3. **Federación corporativa** (AD/SAML): supera el límite de 5.000 users, SSO.
4. **Web Identity Federation** (Google/Facebook/Cognito): millones de usuarios de una app asumen un rol.
5. **Cross-account**: el partner crea un rol en su cuenta que confía en la tuya.

Notas de cada uno que caen en el examen:

- **(1) Es siempre el mismo patrón con nombres distintos**: *execution role* en Lambda, *[[instance-profile|instance profile]]* en EC2, *task role* en ECS, *service role* en CodeBuild o [[CloudFormation]]. Y **la aplicación no cambia nada**: los SDK recorren su cadena de credenciales y terminan tomando las del rol — migrar de access keys a rol suele ser **borrar la configuración de credenciales** y adjuntar el rol.
- **(2) Break glass no es una feature de AWS**, es un **patrón**: rol con permisos elevados + trust policy que dice quién lo asume + fricción deliberada (MFA) + una **alerta automática** (Slack/PagerDuty) cada vez que alguien lo asume, para que "romper el vidrio" no pase desapercibido. Ventaja sobre dar permisos altos permanentes: el acceso **se vence solo** y [[CloudTrail]] registra tanto el `AssumeRole` como todo lo que se hizo después.
- **(5) Object Owner**: cuando tus usuarios suben objetos al bucket del partner **asumiendo un rol de la cuenta del partner**, los objetos quedan siendo **propiedad del partner** — así se evita el problema clásico de "subí un archivo al bucket de otro y ni el dueño puede leerlo" (ver [[S3]]).

![[Pasted image 20260705181813.png]]

### Service-Linked Roles

Los [[service-linked-role|service-linked roles]] son roles que **AWS predefine** para servicios tan complejos que armar la policy a mano sería frágil (ej: Auto Scaling necesita lanzar instancias, engancharlas a un ELB, leer métricas de [[CloudWatch]]…). AWS dice "yo ya sé qué necesito, tomá este rol y no lo edites".

Cómo se reconocen: viven bajo el path reservado **`/aws-service-role/`** y el nombre empieza con **`AWSServiceRoleFor…`**

```
arn:aws:iam::123456789012:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling
```

Servicios que los usan: Auto Scaling, ELB, RDS, EKS, [[Organizations]], GuardDuty, Config, Trusted Advisor.

| Aspecto | Role normal (service role) | Service-linked role |
|---|---|---|
| Quién define los permisos | **Vos** | El servicio de AWS |
| ¿Editar permissions policy? | Sí | **No** (la mantiene AWS) |
| ¿Editar trust policy? | Sí | **No** |
| ¿Borrarlo? | Cuando quieras | **Solo si el servicio ya no lo usa** |
| Quién lo crea | Vos | El servicio, normalmente automático |
| Si el servicio agrega funciones | Lo actualizás vos | **AWS lo actualiza solo** |

Esa última fila es la ventaja práctica real. Para permitir su creación: `iam:CreateServiceLinkedRole` con condition **`iam:AWSServiceName`**.

> ⚠️ **No adivines el nombre del servicio ni el prefijo del rol**: el formato **varía entre servicios y es case sensitive**. Se busca en la doc del servicio puntual.

Relacionado: `iam:PassRole` = permiso para *entregarle* un rol a un servicio (control anti escalación de privilegios).

### Access Keys

- Credenciales de **largo plazo** para CLI/API. **No rotan solas.**
- Máximo **2 por user** (para rotar sin downtime: crear nueva → migrar → desactivar vieja → borrar).
- La **Secret Access Key se muestra una sola vez**.
- Regla de oro: si algo corre **dentro de AWS**, usar **roles**, no access keys.
- Otros tipos de credencial de un user: SSH keys (CodeCommit) y server certificates (usar **ACM** salvo en regions que ACM no soporta). Configuración de uso en la CLI: ver [[aws-cli]].
- Un user creado por **CLI/API nace sin ninguna credencial**; desde la consola hay que elegir al menos password o access keys. Desactivar la password (consola) **no** afecta sus access keys ni sus permisos.
- Alternativa reciente a las access keys para personas: **`aws login`** — la CLI/SDK se autentica con las credenciales de consola del user (requiere el permiso `SignInLocalDevelopmentAccess`).
- Si una app usa credenciales de un IAM user (*service account*): **nunca embeber las keys en el código** — los SDK/CLI las leen de ubicaciones conocidas; mejor aún, un rol.

**El prefijo del Access Key ID delata el tipo de credencial** (dato de examen fácil de reconocer):

| Prefijo | Qué es |
|---|---|
| **`AKIA…`** | Access key de **largo plazo** de un IAM user |
| **`ASIA…`** | Credenciales **temporales** emitidas por STS (un rol) |

Ver un `AKIA` dentro de una instancia [[EC2]] es por sí solo una señal de mala configuración: ahí debería haber un [[instance-profile|rol]].

> **La secret key nunca viaja en la request.** Se usa localmente para firmar el pedido con **SigV4**; lo que se envía es la firma. Por eso interceptar una request de AWS no revela la credencial — y por eso un **reloj muy desfasado** en el cliente rompe la firma.

| | **Access keys (largo plazo)** | **[[temporary-credentials\|Temporales]] (STS)** |
|---|---|---|
| Vencen | No, hasta que las borres | Sí (minutos u horas) |
| Partes | 2: key ID + secret | 3: + **session token** |
| Prefijo | `AKIA` | `ASIA` |
| Quién las tiene | IAM users | Roles, federación, MFA con `get-session-token` |
| Riesgo si se filtran | **Alto**: sirven hasta que alguien las revoque | Bajo: caducan solas |

Las access keys se justifican cada vez menos: código en AWS → **rol**; personas → **IAM Identity Center**; servidores **fuera** de AWS → **IAM Roles Anywhere** (credenciales temporales con certificados X.509).

### Si una access key se filtra (orden correcto)

1. **Desactivarla** — no borrarla todavía: desactivada deja de funcionar pero **conserva la evidencia**.
2. Revisar [[CloudTrail]]: qué se hizo con ella y desde qué IPs.
3. Crear una key nueva y actualizar los sistemas legítimos.
4. **Borrar** la comprometida.
5. Buscar **puertas traseras**: usuarios, roles o recursos creados por el atacante.

El error más caro y más común es **subirlas a Git**: hay bots escaneando GitHub que encuentran una key en minutos y levantan instancias para minar cripto. Le siguen hardcodearlas en una AMI (quedan en cada copia y cada backup), crearlas para el **root user** y no rotarlas nunca.

### Auditoría de credenciales

- **Credential report**: CSV con todos los users y el estado/antigüedad de passwords, access keys y MFA — la herramienta de auditoría estándar. Es la vía rápida para encontrar **keys viejas y usuarios sin MFA**.
- **Access Advisor** (pestaña *Last Accessed* de cada identidad): muestra **qué servicios usó realmente** esa identidad y cuándo → sirve para recortar permisos sobrantes **con evidencia**. No confundir con Access Analyzer.
- **Password policy** configurable por cuenta (complejidad, permitir que cada user cambie la suya).
- **IAM Access Analyzer** — cuatro capacidades que caen en el examen:
	1. **External access**: detecta recursos compartidos **fuera de la zone of trust** (cuenta u organización) analizando resource policies — S3, roles, KMS keys, Lambda, SQS, SNS, secrets, snapshots… Genera *findings* que se resuelven o archivan (uso legítimo). Es **por region** (crear un analyzer en cada una); reanaliza policies nuevas en ~30 min.
	2. **Unused access** (pago, por rol/user analizado): roles sin uso, access keys y passwords sin uso, acciones nunca ejecutadas. No depende de la region; excluye service-linked roles.
	3. **Policy validation** y custom policy checks: valida sintaxis y best practices al escribir policies (los warnings del editor de la consola son esto).
	4. **Policy generation**: genera una policy de mínimo privilegio a partir de la actividad real en [[CloudTrail]].

### Límites (memorizar)

| Límite | Valor |
|---|---|
| Users por cuenta | **5.000** |
| Groups por cuenta | 300 |
| Groups por user | 10 |
| Duración sesión STS | 15 min – 12 h (role chaining: máx **1 h**; pedir `DurationSeconds` > 3600 en chaining **falla**) |

Como IAM es **global**, estos límites son **por cuenta, no por region**: no podés tener 5.000 users en `us-east-1` y otros 5.000 en `eu-west-1`.

> El límite de **5.000** no es un número para memorizar y olvidar: es **el argumento detrás de la mitad de los escenarios de roles del examen**. Cada vez que el enunciado diga "20.000 empleados", "millones de usuarios de una app" o "una cantidad desconocida de accesos", la pista es que la respuesta **no** son IAM users.

## Integración con otros servicios

- [[ec2-instance-metadata]] — el [[instance-profile]] entrega las credenciales del role a la instancia a través del IMDS (`169.254.169.254`). Es el motivo por el que nunca hay que poner access keys en una instancia — y por el que conviene exigir **IMDSv2**.
- [[execution-role]] — el equivalente serverless: el role que Lambda asume para correr tu código ([[lambda-in-vpc]]).

- [[Organizations]] — multi-cuenta con roles + `sts:AssumeRole` ("switch role").
- [[KMS]] — key policies + IAM policies para acceso a claves.
- [[S3]] — bucket policies (resource) vs. identity policies.
- [[CloudTrail]] — audita cada AssumeRole y llamada de API.

## Gotchas y trampas del examen

- Más de 5.000 identidades → **roles + federación**, nunca "pedir aumento de límite".
- En una **trust policy no se permite wildcard (`*`) en el ARN del `Principal`**; y a un **service-linked role no se le puede aplicar [[permissions-boundary|permissions boundary]]**.
- Identidad externa (AD, Google) **no puede usarse directamente**: solo puede **asumir un rol**.
- Un rol de terceros (SaaS) debe exigir **[[external-id|External ID]]** en la trust policy (evita [[confused-deputy]]).
- Las credenciales temporales ya emitidas siguen vivas al quitar permisos → revocar con policy `aws:TokenIssueTime`.
- No generar [[presigned-url|presigned URLs]] de S3 con un rol (expiran con la sesión).
- "Access key `AKIA…` hardcodeada en una EC2" → el error es la key en sí: la respuesta es **instance role**.
- "El script crea un rol y falla al usarlo inmediatamente" → **eventual consistency** de IAM, reintentar con backoff.
- "¿Qué permisos sobran en esta identidad?" → **Access Advisor** (uso real); "¿qué policy debería tener?" → **Access Analyzer policy generation**.

## Demos del curso

- [Creating IAMADMIN user & adding MFA](https://learn.cantrill.io/courses/1101194/lectures/63973711)
- [Creating Access Keys and setting up AWS CLI](https://learn.cantrill.io/courses/1101194/lectures/24949221)
- [Permissions control using IAM Groups](https://learn.cantrill.io/courses/1101194/lectures/25335805)
