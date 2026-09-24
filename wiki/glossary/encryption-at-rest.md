---
title: Encryption at Rest / in Transit
category: glossary
tags: [cifrado, seguridad, tls, sse]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/04 S3/04.06 S3 Object Encryption.md", "raw/doc oficial/AWS KMS keys - AWS Key Management Service.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.13 EBS Encryption.md"]
updated: 2026-09-24
---

# Encryption at Rest / in Transit

> **En una línea:** **at rest** = datos guardados en disco; **in transit** = datos viajando por la red.

## Definición

| | **At rest** | **In transit** |
|---|---|---|
| Protege | El dato almacenado | El dato en movimiento |
| Cómo | SSE-S3 / SSE-KMS / SSE-C / client-side ([[s3-encryption]]) | **TLS/HTTPS** |
| Contra qué | Acceso al medio físico, admin sin permisos de clave | Interceptación en la red |

En [[S3]] el cifrado in transit ocurre **siempre** (HTTPS); las decisiones interesantes están del lado de at rest. Desde 2023 el SSE es **obligatorio**: no hay objetos sin cifrar, solo elegís el *cómo* (default **SSE-S3**).

## Dónde aparece

- [[s3-encryption]] — contexto y tabla comparativa de los cuatro métodos
- [[KMS]] — cifrado en reposo de S3/EBS/RDS/DynamoDB
- [[shared-responsibility-model]] — el cifrado client-side y server-side es responsabilidad **del cliente**
- [[EBS]] — cifrado de volúmenes y snapshots con AES-256, sin impacto de rendimiento
- También en: [[dva-security]] · [[data-encryption-key]] · [[envelope-encryption]]

## Dato de examen

- **Forzar HTTPS** en un bucket → bucket policy con `Deny` + condition **`aws:SecureTransport: false`** (opcionalmente `s3:TlsVersion` para exigir TLS mínimo).
- "Los buckets no se cifran; **los objetos sí**" — el default bucket encryption solo fija el método para objetos **nuevos**.
- La **metadata del objeto no se cifra**, solo los datos.

## Ver también

[[envelope-encryption]] · [[role-separation]] · [[data-encryption-key]]
