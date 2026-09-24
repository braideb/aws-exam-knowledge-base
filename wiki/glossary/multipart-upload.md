---
title: Multipart Upload
category: glossary
tags: [s3, performance, upload, storage]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Uploading and copying objects using multipart upload in Amazon S3 - Amazon Simple Storage Service.md", "raw/doc oficial/Using server-side encryption with Amazon S3 managed keys (SSE-S3) - Amazon Simple Storage Service.md", "raw/notas curso mejorado/04 S3/04.04 S3 Performance Optimization.md"]
updated: 2026-09-24
---

# Multipart Upload

> **En una línea:** subir un objeto **partido en trozos paralelos**, con reintento por trozo.

## Definición

En vez de un único stream, [[S3]] recibe partes independientes y las ensambla. Ventajas: velocidad = **suma de los streams**, y si falla una parte se reintenta **solo esa**, no el objeto entero.

| Dato | Valor |
|---|---|
| Tamaño de parte | **5 MB – 5 GB** (la última puede ser menor) |
| Máx. partes | **10.000** |
| Obligatorio a partir de | **5 GB** (límite del single PUT) |
| Lo activan las herramientas desde | ~100 MB |

## Dónde aparece

- [[S3]] — sección Performance
- También en: [[dva-development]] · [[s3-encryption]] · [[etag]] · [[object-storage]] · [[prefix]] · [[presigned-url]]

## Dato de examen

- **Permisos**: iniciar/subir/completar usan `s3:PutObject`; **abortar** requiere `s3:AbortMultipartUpload` y listar partes `s3:ListMultipartUploadParts`.
- **Costo invisible**: los multipart **incompletos se siguen facturando** → lifecycle rule `AbortIncompleteMultipartUpload`.
- El **[[etag|ETag]] de un objeto multipart NO es el MD5** del contenido (es un checksum de checksums) → rompe verificaciones ingenuas de integridad.
- Con SSE-KMS hace falta **`kms:GenerateDataKey` y `kms:Decrypt`** (uno al iniciar, el otro al subir partes).

## Ver también

[[object-storage]] · [[prefix]] · [[envelope-encryption]]
