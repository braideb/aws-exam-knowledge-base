---
title: KMS (Key Management Service)
category: service
tags: [kms, seguridad, cifrado, claves, dek, envelope-encryption, key-policy]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/04 S3/04.05 KMS (Key Management Service).md", "raw/doc oficial/AWS KMS keys - AWS Key Management Service.md", "raw/doc oficial/Rotate AWS KMS keys - AWS Key Management Service.md", "raw/doc oficial/Grants in AWS KMS - AWS Key Management Service.md", "raw/doc oficial/Multi-Region keys in AWS KMS - AWS Key Management Service.md"]
updated: 2026-09-24
---

# KMS — Key Management Service

## ¿Qué es?

Servicio de **gestión de claves criptográficas**. Casi todos los servicios AWS que cifran datos lo usan por detrás. Es **regional y público** (ocupa la AWS Public Zone — ver [[global-infrastructure]]): cada region tiene su KMS aislado. Maneja claves **simétricas y asimétricas**, y **las claves NUNCA salen de KMS** — toda operación pasa por su API. Cumple **FIPS 140-2 Level 2** (algunas funciones alcanzan Level 3, pero el servicio en general es L2).

Flujo básico de la API: **`CreateKey`** → **`Encrypt`** (le mandás hasta 4 KB de plaintext, te devuelve ciphertext) → **`Decrypt`**. Los datos van hacia KMS; la clave nunca sale.

## Casos de uso

- [[encryption-at-rest|Cifrado en reposo]] de S3/EBS/RDS/DynamoDB y demás.
- **[[role-separation|Role separation]]**: quien administra el recurso ≠ quien puede descifrar.
- Auditoría de uso de claves ([[CloudTrail]] registra cada operación).

## Características clave

### KMS Keys (ex-CMK)

Contenedor **lógico** del material criptográfico (ID, policy, estado, fecha). El material puede generarse en KMS o importarse.

Jerarquía interna (doc oficial): **Domain key** (AES-256-GCM, rotada **a diario** por KMS) → **HSM Backing Key** (HBK = una "versión" del material de tu KMS key; nunca sale del HSM en plaintext) → **Derived Encryption Keys** (una por operación) → tus DEKs/datos. Cada rotación anual crea un HBK nuevo y conserva los anteriores.

> [!warning] Límite de **4 KB**: una KMS Key solo cifra directamente hasta 4KB. Es intencional — están pensadas para cifrar **otras claves**, no tus datos.

### DEKs y envelope encryption

`GenerateDataKey` crea una **[[data-encryption-key|Data Encryption Key]]** a partir de una KMS Key y devuelve **dos versiones**: plaintext (usar ya y descartar) y ciphertext (la misma DEK cifrada con la KMS Key).

**KMS no cifra tus datos grandes ni guarda las DEKs.** Vos (o el servicio):
1. Cifrás los datos con la DEK plaintext.
2. **Descartás la plaintext.**
3. Guardás la **DEK cifrada junto a los datos**.
4. Para descifrar: mandás la DEK cifrada a KMS (`Decrypt`) → te devuelve la plaintext → descifrás → descartás.

Este patrón es **[[envelope-encryption]]** y es la base de todo el cifrado en AWS. [[S3]] con SSE-KMS genera **una DEK por objeto**.

![[Pasted image 20260715080938.png]]

### Tipos de clave

| Tipo | Quién la maneja | Configurable |
|---|---|---|
| **AWS Owned** | AWS, invisible, multi-cuenta | No (gratis) |
| **AWS Managed** (`aws/s3`…) | AWS, al integrar un servicio | Casi nada; rotación anual forzosa |
| **Customer Managed** | Vos | **Todo**: key policy, [[cross-account]], rotación |

- **Rotación**: cambia el material de respaldo; el material viejo **se conserva** → lo cifrado antes sigue descifrándose. Customer managed: opcional (~anual) + **rotación on-demand** (doc oficial; también disponible para claves simétricas con material **importado**). **Rotación manual únicamente** para claves asimétricas, HMAC y custom key stores.
- Período de rotación configurable con **`RotationPeriodInDays`** (default 365); la condition key `kms:RotationPeriodInDays` permite acotarlo por policy. Costo: se factura la 1ª y 2ª rotación del material, **con tope en la 2ª** (las siguientes son gratis). Cada rotación emite un evento a **[[EventBridge]]** y un `RotateKey` en [[CloudTrail]].
- **AWS owned keys son el default de los servicios nuevos desde 2021**; las AWS managed (`aws/…`) son el mecanismo legacy.
- **Aliases**: puntero con nombre a una clave; **por region**, no globales.
- Claves **aisladas a su region** (multi-region keys son la excepción, prefijo `mrk-`).
- Borrado: solo programado con espera de **7–30 días** (cancelable).

### Key Policies

Toda KMS Key tiene **exactamente una** key policy (resource policy). **KMS no confía en la cuenta por defecto** — a diferencia del resto de AWS, la confianza debe ser explícita (la misma lógica que una [[trust-policy|trust policy]]):

```json
{ "Sid": "Enable IAM User Permissions",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
  "Action": "kms:*", "Resource": "*" }
```

Cadena completa: **key policy confía en la cuenta → IAM policies conceden permisos puntuales** (`kms:Encrypt`, `kms:Decrypt`).

> ⚠️ **El escenario de la clave huérfana.** Si alguien reescribe la key policy y se olvida del statement de confianza a la cuenta, la clave queda inaccesible: **nadie —ni el root user— puede usarla ni volver a editar su policy**. La única salida es **abrir un caso con soporte de AWS**. No se toca ese statement base sin cuidado.

**Los permisos de KMS se dividen en dos familias**, y separarlas es la base del [[role-separation|role separation]]:

| Familia | Acciones típicas | Quién debería tenerla |
|---|---|---|
| **Uso de la clave** | `kms:Encrypt`, `kms:Decrypt`, `kms:GenerateDataKey`, `kms:ReEncrypt` | Las aplicaciones y usuarios que manejan datos |
| **Administración de la clave** | `kms:PutKeyPolicy`, `kms:ScheduleKeyDeletion`, `kms:EnableKeyRotation`, `kms:CreateGrant` | Los administradores de seguridad |

La gracia es que se dan **por separado**: alguien puede rotar la clave y cambiar su policy **sin poder descifrar datos**, y otro puede descifrar **sin poder administrarla**.

![[Pasted image 20260715082605.png]]

### Grants

Tercer mecanismo de autorización (además de key policy e IAM policies): un **grant** da permisos **temporales y solo de Allow** (jamás Deny) sobre **exactamente una** KMS key, sin tocar ninguna policy. Es como los usan los **servicios AWS** que cifran at rest: crean un grant en tu nombre, usan la clave y lo **retiran** al terminar.

- El grantee usa el permiso **sin mencionar el grant** (como si viniera de una policy). Se elimina con **retire** (lo hace el retiring [[principal]] del grant, "terminé de usarlo") o **revoke** (lo hace un admin, "te corto el acceso").
- **[[eventual-consistency|Eventual consistency]]**: un grant recién creado puede tardar segundos/minutos en propagarse → para usarlo YA está el **grant token** (string base64 no-secreto que devuelve **solo `CreateGrant`**; `ListGrants` da el grant ID, no el token).
- **Grant constraints**: por **[[encryption-context|encryption context]]** (solo claves simétricas) o por **`SourceArn`** (solo requests en nombre de un recurso concreto — obligatorio cuando el grantee es un service principal — es la defensa contra el [[confused-deputy|confused deputy]]).
- Solo permite **grant operations** (cifrar/descifrar, DescribeKey, crear/retirar grants…) y deben ser soportadas por el tipo de clave (una simétrica no puede grantear `Sign`). Grantee = cualquier principal IAM, **nunca un IAM group ni una organización**.
- ⚠️ `kms:CreateGrant` es tan sensible como `kms:PutKeyPolicy`: quien puede crear grants puede dar acceso a la clave a terceros (aunque quien recibió el permiso *vía otro grant* solo puede delegar lo que le fue granteado). Límite: **50.000 grants por clave**.

### Multi-Region Keys

Claves **relacionadas** en distintas regiones con el **mismo key ID y mismo material** (prefijo `mrk-`): cifrás en una region y descifrás en otra **sin llamada cross-region ni re-cifrado**. Casos de uso: **DR**, datos globales (ej. DynamoDB global tables con cifrado client-side), firmas distribuidas, apps activo-activo.

- **No son globales**: creás una **primary** y la **replicás** explícitamente a regiones elegidas (misma partición). AWS nunca replica por su cuenta. **No se puede convertir** una single-region key existente en multi-region (ni al revés).
- Cada réplica es una **clave completa e independiente**: key policy, grants, aliases, tags y estado enabled/disabled son **propiedades independientes**; key ID, material, spec/usage y rotación son **compartidas**.
- La **rotación (automática u on-demand) solo se configura en la primary**; KMS sincroniza el material a las réplicas. La primary **no se puede borrar hasta borrar todas las réplicas** (se puede promover una réplica a primary primero).
- Las **AWS managed keys son siempre single-region**; tampoco hay multi-region en custom key stores. Cada réplica cuenta y se factura como una clave más en su region.
- Gotcha: la mayoría de los servicios AWS las tratan como claves comunes — ej. **S3 CRR igual descifra y re-cifra** con la clave del destino aunque ambas sean réplicas relacionadas.

## Integración con otros servicios

- [[EBS]] — cada volumen cifrado recibe su propia [[data-encryption-key|DEK]] vía `GenerateDataKeyWithoutPlaintext`; la clave en claro solo vive en la memoria del EC2 host, nunca en disco. Un snapshot hereda la DEK del volumen.

- [[S3]] / [[s3-encryption]] — SSE-KMS, bucket keys.
- [[IAM]] — key policy + identity policies.
- [[CloudTrail]] — auditoría de cada `GenerateDataKey`/`Decrypt` (quién descifró qué y cuándo).

## Gotchas y trampas del examen

- "FIPS 140-2 **Level 3**" o HSM dedicado single-tenant → **CloudHSM**, no KMS. El nivel FIPS es el diferenciador que usa el examen.
- "Reescribí la key policy y ahora nadie puede usar la clave" → **caso con soporte de AWS**; no hay forma de arreglarlo desde la cuenta.
- "Que el equipo de seguridad administre la clave pero no lea los datos" → separar las **dos familias de permisos** (uso vs. administración).
- KMS Key ≠ cifrar datos grandes: >4KB → DEKs (envelope encryption).
- La AWS managed key `aws/s3` **no sirve cross-account** → customer managed key con ARN explícito.
- Role separation: admin de S3 sin permisos sobre la KMS Key **no puede leer** los objetos cifrados con SSE-KMS.
- Los aliases no son globales; cada region tiene el suyo.
- "Permiso temporal a la clave sin modificar policies" → **grant**; "AccessDenied justo después de `CreateGrant`" → eventual consistency, usar el **grant token**.
- "Cifrar en us-east-1 y descifrar en eu-west-1 sin llamadas cross-region" → **multi-region keys** (replicadas explícitamente; una single-region key no se puede convertir).

## Demos del curso

- [KMS — Encrypting the battleplans](https://learn.cantrill.io/courses/1101194/lectures/25997329)
