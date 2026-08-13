---
title: Cifrado en S3 (Client-Side vs SSE-C vs SSE-S3 vs SSE-KMS)
category: comparison
tags: [s3, kms, cifrado, sse, encryption, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/04 S3/04.06 S3 Object Encryption.md", "raw/notas curso mejorado/04 S3/04.07 S3 Bucket Keys.md", "raw/notas curso mejorado/04 S3/04.05 KMS (Key Management Service).md", "raw/doc oficial/Using server-side encryption with Amazon S3 managed keys (SSE-S3) - Amazon Simple Storage Service.md", "raw/doc oficial/Using server-side encryption with AWS KMS keys (SSE-KMS) - Amazon Simple Storage Service.md"]
updated: 2026-07-25
---

# Comparativa: Cifrado de objetos en S3

> Une la página de [[S3]] con la de [[KMS]]: cómo se cifra cada objeto y quién controla las claves.

## Contexto

- El cifrado **in transit** (HTTPS) ocurre siempre; acá hablamos de **[[encryption-at-rest|at rest]]**.
- **Los buckets no se cifran; los objetos sí** — cada objeto puede tener config distinta. El "default bucket encryption" solo define el método por defecto para objetos nuevos.
- SSE es **obligatorio** desde 2023: ya no hay objetos sin cifrar; elegís el *cómo*. Default: **SSE-S3**.

## Los dos componentes del SSE

Todo cifrado server-side tiene **dos** partes, y cada método reparte la responsabilidad distinto:

1. **El proceso de cifrado/descifrado** — plaintext + clave + algoritmo → ciphertext. Es **simétrico**: la misma clave en ambas direcciones.
2. **La generación y gestión de las claves.**

## Tabla comparativa

| Método | Gestión de claves | Proceso de cifrado | Extras |
|---|---|---|---|
| **Client-Side** | VOS | VOS | AWS **nunca** ve el plaintext |
| **SSE-C** | VOS (mandás la clave en cada PUT/GET) | S3 | S3 guarda solo un hash de la clave |
| **SSE-S3** (AES-256, default) | S3 | S3 | Cero control de claves, **sin role separation** |
| **SSE-KMS** | [[KMS]] | S3 | Rotación ✅ [[role-separation\|Role separation]] ✅ Auditoría CloudTrail ✅ |

Qué viaja por el túnel HTTPS: en **client-side** ya va ciphertext; en los tres SSE va el **plaintext original** (dentro de HTTPS) y S3 lo ve momentáneamente al recibirlo. De ahí que solo client-side garantice que AWS nunca vio los datos en claro — a costa de que **si perdés la clave, perdés los datos**, y AWS no puede ayudarte porque nunca la tuvo.

**Detalles de cada uno que caen:**

- **SSE-C**: ¿por qué elegirlo? Con millones de objetos el consumo de **CPU** del cifrado es enorme; SSE-C te lo **transfiere a AWS** conservando el control de las claves. Como S3 no guarda la clave, **para bajar el objeto hay que mandar la misma clave con la que se cifró** — si la perdés, es irrecuperable. Y **exige HTTPS siempre** (la clave no puede viajar en claro).
- **SSE-S3**: S3 mantiene una **S3 Key** interna (la crea, administra y rota él, vos tenés cero control). Por cada objeto genera una **object key** única, cifra el objeto con ella, cifra la object key con la S3 Key, y guarda la object key cifrada junto al objeto. Sus **tres límites**: no sirve en entornos fuertemente regulados, no permite controlar la rotación y **no da role separation**.
- **SSE-KMS**: para **leer** un objeto hacen falta **dos permisos a la vez** — `s3:GetObject` **y** `kms:Decrypt` sobre esa clave. Tener uno solo no alcanza: eso es exactamente lo que habilita el role separation.

![[Pasted image 20260715223422.png]]

## Cuándo usar cada uno

- "AWS jamás debe poder ver los datos" → **Client-Side** (S3 solo almacena ciphertext).
- "Controlo mis claves pero no quiero gastar CPU cifrando" → **SSE-C**.
- "Que AWS se encargue de todo, sin requisitos regulatorios" → **SSE-S3**.
- "Role separation / rotación controlada / auditoría / regulado (finanzas, salud)" → **SSE-KMS** con **customer managed key**.

### Detalles finos de SSE-KMS (doc oficial — caen en DVA)

- **Permisos exactos**: subir requiere **`kms:GenerateDataKey`**; leer requiere **`kms:Decrypt`**. Multipart necesita **ambos** (GenerateDataKey al iniciar, Decrypt al subir partes). "Access Denied subiendo a un bucket SSE-KMS" casi siempre es GenerateDataKey faltante.
- **[[encryption-context|Encryption context]]**: pares clave-valor usados como datos autenticados adicionales (AAD); S3 usa por defecto el **ARN del objeto** (o el **del bucket** si hay Bucket Keys). El mismo context debe presentarse al descifrar — y aparece en CloudTrail.
- Solo admite **KMS keys simétricas**, y la key debe estar en la **misma region** que el bucket.
- Para exigir una key específica en uploads: condition `s3:x-amz-server-side-encryption-aws-kms-key-id` — en [[cross-account]] usar el **ARN completo** (un alias se resuelve en la cuenta del *requester*).
- GET de un objeto SSE-KMS mandando headers de cifrado → **HTTP 400**.
- La **metadata del objeto no se cifra** (solo los datos); SSE-S3 usa **AES-256-GCM**. Existe también **DSSE-KMS** (doble capa) como cuarto método server-side.

### Por qué SSE-KMS y no SSE-S3 (el punto del examen)

Con SSE-S3, un admin full de S3 **puede leer los datos** (las claves viven en S3). Con SSE-KMS, si no tiene permisos sobre la KMS Key, **no puede descifrar** — administrar el recurso y leer los datos quedan separados. Además cada `Decrypt` queda en [[CloudTrail]].

![[Pasted image 20260715223341.png]]

## S3 Bucket Keys (optimización de SSE-KMS)

Problema: SSE-KMS llama a KMS **por cada objeto** (costo + [[throttling]] de 5.500–50.000 req/s según region). Solución: una **bucket key temporal** generada por la KMS Key, con la que **S3 fabrica las [[data-encryption-key|DEKs]] localmente**.

- Menos llamadas a KMS → menor costo, más escala. AWS habla de hasta **99% menos llamadas**: en vez de una por objeto, una cada tanto para renovar la bucket key.
- **No retroactivo** (solo objetos nuevos). Para re-cifrar lo existente → **Batch Operations** con Copy in-place ([[S3]]).
- CloudTrail pasa a mostrar el ARN del **bucket** (menos eventos KMS). Si tu auditoría dependía de ver **una llamada a KMS por objeto**, esto la cambia — es la única razón real para no activarlas.
- **Funciona con replicación** (SRR y CRR): la configuración de cifrado del objeto se mantiene.

> **El punto del [[etag|ETag]] (caso especial de replicación).** Si replicás un objeto que estaba en **plaintext** hacia un bucket **con bucket keys habilitadas**, el objeto se cifra **recién en el destino** → los bytes no son los mismos → **el ETag cambia** entre origen y destino. Importa si tenés herramientas que comparan ETags para verificar que la réplica esté "igual": van a reportar diferencia aunque la réplica sea correcta.
>
> La otra trampa del ETag: para un objeto de un solo PUT, el ETag **es** el MD5 del contenido. Para uno de [[multipart-upload|multipart]], es un **hash de los hashes de las partes con sufijo `-N`** (N = cantidad de partes) → comparar contra el MD5 del archivo original **siempre da distinto**.

**Cuándo activarlas:** prácticamente siempre que uses SSE-KMS con volumen medio o alto. El ahorro es grande y no hay contras relevantes más allá del cambio en los logs.

## Trampa típica del examen

- SSE-C ≠ client-side: en SSE-C **S3 cifra** (vos solo aportás la clave).
- "Exigir que todo upload venga cifrado con KMS" → bucket policy con condition `s3:x-amz-server-side-encryption`.
- La AWS managed key `aws/s3` no sirve para acceso cross-account → customer managed key.
- Requisito de que nadie de AWS vea plaintext → **solo client-side** cumple.

## Demos del curso

- [Object Encryption and Role Separation](https://learn.cantrill.io/courses/1101194/lectures/25997332)
