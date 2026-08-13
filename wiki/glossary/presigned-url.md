---
title: Presigned URL
category: glossary
tags: [s3, iam, temporary-credentials, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-08-12
---

# Presigned URL

> **En una línea:** una URL con la firma y las credenciales de quien la generó incluidas en la query string — quien la tiene puede acceder (o subir), sin sus propias credenciales, hasta que expire.

## Definición

Es la URL normal de un objeto S3 más query params con las credenciales del generador, un timestamp, la validez y una firma criptográfica SigV4. El servicio recalcula la firma en cada uso: no se puede editar la URL para apuntar a otro objeto o extender su vida, y el generador no necesita estar online cuando se usa, porque toda la autorización viaja dentro de la URL. Sirve tanto para GET (descarga) como para PUT (upload directo al bucket, el patrón estándar de formularios de carga en apps serverless).

## Dónde aparece

- [[S3]] — sección completa "Presigned URLs": mecánica, expiraciones por origen, gotchas
- [[CloudFront]] — comparación con Signed URLs/Cookies (alcance: un objeto de S3 vs. contenido detrás del CDN)
- [[IAM]], [[temporary-credentials]] — por qué no conviene generarlas con credenciales de un rol
- [[dva-development]] — dominio 1 del examen DVA-C02

## Dato de examen

- La vida real de una presigned URL generada desde un rol (Lambda, EC2 instance role) es **el menor** entre su propia expiración y la de las credenciales que la firmaron — con roles de ~1h, "se vencen antes de tiempo" aunque se pidió más tiempo.
- Expiración máxima según quién la genera: consola 1 min–12h; CLI/SDK hasta 7 días; con credenciales de instance role ~6h; con AssumeRole ~1h.

## Ver también

[[S3]] · [[temporary-credentials]] · [[multipart-upload]]
