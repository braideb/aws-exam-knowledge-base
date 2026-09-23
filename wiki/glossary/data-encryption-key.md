---
title: DEK (Data Encryption Key)
category: glossary
tags: [kms, cifrado, dek, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-09-23
---

# DEK — Data Encryption Key

> **En una línea:** la clave que **sí** cifra tus datos; la KMS Key solo la protege.

## Definición

Clave generada por `GenerateDataKey` a partir de una KMS Key. Vuelve en **dos versiones**:

| Versión | Qué se hace con ella |
|---|---|
| **Plaintext** | Se usa para cifrar/descifrar y se **descarta inmediatamente** |
| **Ciphertext** (cifrada con la KMS Key) | Se **guarda junto a los datos** |

Es la pieza central de la [[envelope-encryption]]. [[S3]] con SSE-KMS genera **una DEK por objeto**.

## Dónde aparece

- [[KMS]] — sección "DEKs y envelope encryption"
- [[s3-encryption]] — S3 Bucket Keys: S3 fabrica las DEKs **localmente** a partir de una bucket key temporal
- [[EBS]] — DEK única por volumen, en claro solo en la memoria del EC2 host
- También en: [[dva-development]]

## Dato de examen

- "Reducir el costo y el [[throttling]] de KMS en un bucket con millones de objetos" → **S3 Bucket Keys** (menos llamadas a KMS). **No es retroactivo**: solo objetos nuevos.
- Cada `GenerateDataKey` y cada `Decrypt` queda registrado en [[CloudTrail]] — así se audita **quién descifró qué y cuándo**.

## Ver también

[[envelope-encryption]] · [[encryption-context]] · [[encryption-at-rest]]
