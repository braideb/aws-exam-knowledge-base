---
title: SSM Parameter Store
category: service
tags: [ssm, systems-manager, parameter-store, configuracion, secretos, kms]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/09 Advanced EC2/09.04 SSM Parameter Store.md", "raw/notas curso mejorado/09 Advanced EC2/09.05 Demostración - Parameter Store.md", "raw/notas curso mejorado/09 Advanced EC2/09.06 Logging en EC2 con CloudWatch Agent.md", "raw/doc oficial/AWS Systems Manager Parameter Store - AWS Systems Manager.md"]
updated: 2026-10-03
---

# SSM Parameter Store

## ¿Qué es?

Parte de **AWS Systems Manager (SSM)**. Guarda **parámetros** (nombre + valor) de configuración y secretos, en texto plano o **cifrados con [[KMS]]**, con versionado y jerarquías. Es un **servicio público**: se usa con la CLI o las APIs desde cualquier lugar con acceso a los public endpoints de AWS, y el acceso se controla con [[IAM]].

## Casos de uso

- **Configuración de aplicaciones**: connection strings, hostnames, puertos, feature flags, configuraciones enteras.
- **Secretos sin rotación**: contraseñas, códigos de licencia (con SecureString).
- **Bootstrapping de EC2**: el user data o la app leen la configuración al arrancar con el instance role, en lugar de llevar secretos en el user data ([[ec2-bootstrapping]]).
- **Configuración del CloudWatch Agent**: el agente la guarda y la lee como parámetro ([[CloudWatchLogs]]).
- **AMI ID más reciente**: los **parámetros públicos** de AWS, que usa [[CloudFormation]] para no hardcodear AMIs.

## Características clave

### Tipos

| Tipo | Contenido |
|---|---|
| **String** | Texto plano |
| **StringList** | Lista separada por comas |
| **SecureString** | Texto **cifrado con KMS** |

### SecureString y KMS

Para leer el valor descifrado se pide con **`--with-decryption`**, y hacen falta **dos permisos**: sobre el parámetro (`ssm:GetParameter*`) y sobre la **KMS key** que lo cifra (`kms:Decrypt`). Sin el segundo, la lectura falla aunque la policy de SSM la permita.

### Jerarquías y versionado

```
/wordpress/DBUser
/wordpress/DBPassword          (SecureString)
/my-cat-app/dbstring
/my-cat-app/dbpassword         (SecureString)
```

- Un parámetro se lee por su **nombre completo** (`get-parameters --names /wordpress/DBPassword`) o se lee **toda la rama** de una vez (`get-parameters-by-path --path /wordpress/`).
- La jerarquía sirve para separar por aplicación, por entorno o por equipo, y se combina con IAM: se puede dar permiso solo sobre `/dev-team/*`.
- Cada cambio genera una **versión nueva** del parámetro (como el versionado de objetos de S3). Se guardan las **últimas 100** ¹.

### Tiers

| Tier | Costo | Cantidad | Tamaño del valor | Extras |
|---|---|---|---|---|
| **Standard** | Gratis | Hasta **10.000** parámetros | **4 KB** | — |
| **Advanced** | De pago | Hasta **100.000** ¹ | **8 KB** | **Parameter policies** (por ejemplo, expiración); **compartir con otras cuentas** ¹ |

Un parámetro Standard se puede **pasar a Advanced, pero no volver** ¹. Por defecto el throughput es bajo: para muchas lecturas por segundo hay un modo **high-throughput** de pago (si no, [[throttling]]) ¹.

¹ Complemento de la doc oficial de Parameter Store. La doc recomienda además **Secrets Manager** para credenciales de bases de datos, API keys y tokens, y deja SecureString para configuración sensible.

### Integraciones

- **Parámetros públicos**: los crea AWS. El clásico es el ID de la AMI más reciente de un SO en una región.
- **Referencia a [[SecretsManager|Secrets Manager]]**: con el prefijo `/aws/reference/secretsmanager/<secreto>` se lee un secreto de Secrets Manager a través de la API de Parameter Store.
- Lo usan de forma nativa CloudFormation, EC2 (la CLI desde la instancia), Lambda, [[ECS]] (secrets en la task definition) y el CloudWatch Agent.

## Integración con otros servicios

- [[KMS]]: cifra los SecureString. Hace falta permiso sobre la key.
- [[IAM]]: todo acceso se autentica y autoriza. Lo correcto es un [[instance-profile|instance role]] o un [[execution-role]], no access keys.
- [[EC2]]: la instancia lee su configuración al arrancar ([[ec2-bootstrapping]]).
- [[CloudWatchLogs]]: el CloudWatch Agent guarda su config como parámetro (`ssm:AmazonCloudWatch-linux`).
- [[ECS]]: el task execution role lee los secrets referenciados en la task definition.
- [[CloudFormation]]: parámetros públicos para resolver AMI IDs.

## Gotchas y trampas del examen

- **Rotación automática** de credenciales (por ejemplo, de [[RDS]]) → **Secrets Manager**, no Parameter Store. Ver [[parameter-store-vs-secrets-manager]].
- *"`AccessDenied` al leer un SecureString con `--with-decryption`"* → falta **`kms:Decrypt`** sobre la key, no el permiso de SSM.
- *"Leer toda la configuración de una app en una llamada"* → **`get-parameters-by-path`** sobre su rama.
- *"Valor de 6 KB"* o *"que el parámetro expire"* → tier **Advanced**.
- *"Usar la AMI más reciente sin hardcodear el ID"* → **parámetro público** de SSM.
- No guardes secretos en el **user data**: ponelos en Parameter Store y leelos con el role.

## Demos del curso

- [Parameter Store](https://learn.cantrill.io/courses/1101194/lectures/27895415): crear `/my-cat-app/…`, `/my-dog-app/…`, `/rate-my-lizard/…` y leerlos con `get-parameters`, `get-parameters-by-path` y `--with-decryption`.

> 📖 Lectura profunda: [[09.04 SSM Parameter Store]] · [[09.05 Demostración - Parameter Store]]
