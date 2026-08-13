---
title: WORM (Write Once, Read Many)
category: glossary
tags: [s3, object-lock, compliance, retencion, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# WORM — Write Once, Read Many

> **En una línea:** escrito una vez, **nadie lo puede modificar ni borrar** durante la retención.

## Definición

Modelo de almacenamiento inmutable exigido por normativas financieras y legales. En AWS lo implementa **S3 Object Lock**, que bloquea **versiones individuales** de objetos: ni overwrite ni delete.

| Mecanismo | Duración | ¿Bypass? |
|---|---|---|
| **Compliance** | Días/años | ❌ **Nadie**, ni el root user, hasta expirar |
| **Governance** | Días/años | ✅ con `s3:BypassGovernanceRetention` + header |
| **Legal Hold** | ON/OFF indefinido | ✅ con `s3:PutObjectLegalHold` |

## Dónde aparece

- [[S3]] — sección Object Lock (WORM)
- [[s3-storage-classes]] — Glacier Deep Archive para retención legal

## Dato de examen

- Object Lock **se habilita solo en buckets nuevos**, activa versioning y es **irreversible**.
- En modo **Compliance**, la única salida antes de que expire la retención es **cerrar la cuenta de AWS**. Si el escenario pide "poder corregir un error", la respuesta es **Governance**.
- `DELETE` con version ID sobre una versión bloqueada → **403**; `DELETE` simple → **200 + [[delete-marker]]** (el lock no lo impide).
- La bucket policy puede acotar los períodos con `s3:object-lock-remaining-retention-days`.

## Ver también

[[delete-marker]] · [[durability]] · [[least-privilege]]
