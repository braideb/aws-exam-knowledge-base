---
title: Envelope Encryption
category: glossary
tags: [kms, cifrado, dek, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Envelope Encryption

> **En una línea:** cifrás los datos con una clave y **cifrás esa clave** con otra clave.

## Definición

Patrón base de todo el cifrado en AWS. Como una KMS Key solo cifra hasta **4 KB**, no cifra tus datos: cifra la clave que sí lo hace.

1. `GenerateDataKey` → [[KMS]] devuelve una [[data-encryption-key|DEK]] en **plaintext** y la **misma DEK cifrada**.
2. Cifrás los datos con la DEK plaintext.
3. **Descartás la plaintext.**
4. Guardás la **DEK cifrada junto a los datos**.
5. Para descifrar: mandás la DEK cifrada a KMS (`Decrypt`) → recuperás la plaintext → descifrás → la descartás.

## Dónde aparece

- [[KMS]] — sección "DEKs y envelope encryption"
- [[s3-encryption]] — SSE-KMS genera **una DEK por objeto**; los Bucket Keys optimizan ese patrón

## Dato de examen

- "KMS Key para cifrar un archivo de 1 GB" → ❌, límite de **4 KB**. La respuesta es envelope encryption con DEKs.
- **KMS nunca guarda las DEKs** ni ve tus datos grandes: solo genera y descifra claves.
- Subir a un bucket SSE-KMS requiere **`kms:GenerateDataKey`**; leer requiere **`kms:Decrypt`** — un "Access Denied al subir" casi siempre es el primero.

## Ver también

[[data-encryption-key]] · [[encryption-context]] · [[encryption-at-rest]] · [[role-separation]]
