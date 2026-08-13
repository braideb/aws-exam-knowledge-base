---
title: ETag
category: glossary
tags: [s3, integridad, multipart, http]
exam: [DVA-C02, SAA-C03]
sources: []
updated: 2026-07-25
---

# ETag

> **En una línea:** el identificador de contenido que devuelve S3 — **y que no siempre es el MD5**.

## Definición

Header HTTP que identifica una versión concreta del contenido de un objeto. En [[S3]] su valor depende de **cómo se subió**:

| Cómo se subió | ETag |
|---|---|
| **Single PUT** | **Es** el MD5 del contenido |
| **[[multipart-upload\|Multipart]]** | Un **hash de los hashes de las partes**, con sufijo **`-N`** (N = cantidad de partes) |

## Dónde aparece

- [[S3]] — Performance: el ETag de un objeto multipart no sirve para verificar integridad de forma ingenua
- [[s3-encryption]] — S3 Bucket Keys y el caso de replicación

## Dato de examen

- Comparar el ETag de un objeto **multipart** contra el MD5 del archivo original **siempre da distinto**, aunque el contenido sea idéntico. Es la fuente clásica de falsos positivos al verificar integridad. El `-N` al final es la pista de que fue multipart.
- **Caso de replicación**: si replicás un objeto en plaintext hacia un bucket con **Bucket Keys** habilitadas, el objeto se cifra **en el destino** → los bytes cambian → **el ETag cambia**. Las herramientas que comparan ETags entre origen y destino reportan diferencia aunque la réplica sea correcta.
- Para verificación real de integridad, S3 ofrece **checksums adicionales** (CRC32, CRC32C, SHA-1, SHA-256) que sí funcionan con multipart.

## Ver también

[[multipart-upload]] · [[durability]] · [[object-storage]]
