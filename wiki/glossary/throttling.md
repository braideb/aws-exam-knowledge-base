---
title: Throttling
category: glossary
tags: [kms, limites, performance, s3]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Understanding Amazon DNS - Amazon Virtual Private Cloud.md", "raw/notas curso mejorado/04 S3/04.06 S3 Object Encryption.md", "raw/notas curso mejorado/04 S3/04.07 S3 Bucket Keys.md"]
updated: 2026-09-24
---

# Throttling

> **En una línea:** cuando un servicio empieza a rechazar o demorar tus llamadas porque superaste su límite de requests por segundo.

## Definición

Respuesta de un servicio AWS cuando el volumen de llamadas supera su límite de rate (requests/segundo). No es un error de permisos ni de código: es un límite de servicio ajustable (soft limit) que se resuelve con reintentos con backoff, reduciendo el volumen de llamadas, o — en el caso puntual de KMS — cacheando el resultado en vez de repetir la llamada.

## Dónde aparece

- [[data-encryption-key]], [[s3-encryption]] — SSE-KMS llama a KMS **por cada objeto** (5.500–50.000 req/s según region); **S3 Bucket Keys** reduce hasta 99% esas llamadas
- [[dva-troubleshooting]] — throttling de Lambda/API Gateway como causa de errores 4xx/5xx
- También en: [[VPC]] · [[XRay]]

## Dato de examen

- "Bucket con millones de objetos SSE-KMS empieza a tirar errores de throttling en KMS" → la respuesta esperada es **S3 Bucket Keys**, no simplemente pedir un límite más alto.

## Ver también

[[data-encryption-key]] · [[s3-encryption]]
