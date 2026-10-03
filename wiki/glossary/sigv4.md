---
title: SigV4 (Signature Version 4)
category: glossary
tags: [iam, seguridad, api, s3, rds]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/IAM database authentication for MariaDB, MySQL, and PostgreSQL - Amazon Relational Database Service.md", "raw/notas curso mejorado/04 S3/04.11 S3 Presigned URLs.md"]
updated: 2026-10-03
---

# SigV4 (Signature Version 4)

> **En una línea:** el algoritmo con el que se **firma** cada request a la API de AWS usando la secret key, **sin enviarla**.

## Definición

El cliente (CLI, SDK) calcula localmente una firma criptográfica de la request con la secret access key, y manda solo la firma más el access key ID. AWS recalcula la firma y, si coincide y no expiró, autentica. La misma firma, puesta en query params, es lo que hace funcionar una [[presigned-url|presigned URL]] de S3 y el **token de IAM DB auth** de RDS (válido 15 minutos).

## Dónde aparece

- [[IAM]] — la secret key nunca viaja en la request
- [[S3]] — presigned URLs
- [[RDS]] — `generate-db-auth-token`
- También en: [[presigned-url]]

## Dato de examen

- La firma tiene **vencimiento**: presigned URL (hasta 7 días con un IAM user) y token de RDS (15 min).
- Si las credenciales que firmaron expiran antes (las de un rol), la URL o el token dejan de servir antes de tiempo.

## Ver también

[[presigned-url]] · [[temporary-credentials]] · [[principal]]
