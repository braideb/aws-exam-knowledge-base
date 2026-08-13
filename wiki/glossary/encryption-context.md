---
title: Encryption Context
category: glossary
tags: [kms, cifrado, seguridad, aad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Encryption Context

> **En una línea:** pares clave-valor **no secretos** que quedan atados criptográficamente a la operación.

## Definición

Datos autenticados adicionales (**AAD**) que se pasan al cifrar con [[KMS]]. **El mismo context debe presentarse al descifrar**, o la operación falla. No van cifrados —son visibles— pero cualquier alteración rompe el descifrado.

Doble utilidad:
1. **Integridad**: liga el ciphertext a un contexto concreto.
2. **Auditoría y autorización**: aparece en [[CloudTrail]] y se puede exigir por condition en key policies y **grant constraints**.

## Dónde aparece

- [[s3-encryption]] — S3 usa por defecto el **ARN del objeto** (o el **del bucket** si hay Bucket Keys)
- [[KMS]] — grant constraints por encryption context (**solo claves simétricas**)

## Dato de examen

- "¿Cómo restrinjo un grant a un recurso puntual?" → **encryption context** o **`SourceArn`**.
- No es un secreto: no se ponen datos sensibles adentro (queda en texto plano en los logs).

## Ver también

[[envelope-encryption]] · [[data-encryption-key]]
