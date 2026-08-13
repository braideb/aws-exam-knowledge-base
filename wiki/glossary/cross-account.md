---
title: Cross-Account
category: glossary
tags: [iam, s3, kms, seguridad, roles]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-08-12
---

# Cross-Account

> **En una línea:** cuando el que accede a un recurso y el dueño del recurso son cuentas distintas de AWS — ahí la regla cambia: hacen falta permisos de las dos puntas.

## Definición

Patrón de acceso donde una identidad de la Cuenta A accede a un recurso de la Cuenta B. A diferencia del acceso same-account (donde alcanza con una identity policy **o** una resource policy — se unen), cross-account necesita **las dos puntas**: la cuenta dueña del recurso debe permitirlo explícitamente (bucket policy, key policy) **y** la cuenta que accede debe permitírselo a su propia identidad (IAM policy) — se intersectan. Es el mecanismo detrás de `AssumeRole` entre cuentas, bucket policies cross-account de S3, y key policies cross-account de KMS.

## Dónde aparece

- [[S3]] — bucket policies cross-account, replication cross-account, `RestrictPublicBuckets` corta todo el cross-account
- [[KMS]] — la AWS managed key `aws/s3` **nunca** sirve cross-account, hace falta customer managed key
- [[IAM]] — uno de los cinco escenarios estándar para usar roles
- [[iam-policy-evaluation]] — identity + resource policy: unión en misma cuenta, **intersección obligatoria** en cross-account
- [[confused-deputy]], [[trust-policy]] — el [[external-id|External ID]] como defensa en escenarios cross-account con terceros

## Dato de examen

- Identity policy + resource policy misma cuenta → alcanza con **una** (unión). Cross-account → se necesitan **ambas**.
- La AWS managed key `aws/s3` nunca sirve para cifrado cross-account → hace falta una customer managed key con key policy explícita.
- S3 Replication cross-account: por defecto los objetos replicados quedan con **owner de la cuenta source** — la cuenta destino no puede leer sus propios objetos replicados hasta cambiar el ownership explícitamente.

## Ver también

[[principal]] · [[trust-policy]] · [[confused-deputy]] · [[external-id]]
