---
title: Role Separation
category: glossary
tags: [kms, seguridad, iam, cifrado, compliance]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Role Separation (separación de roles)

> **En una línea:** quien **administra** el recurso no es necesariamente quien puede **leer los datos**.

## Definición

Separar el permiso de gestionar un recurso del permiso de descifrar su contenido. Lo hace posible [[KMS]]: los permisos sobre la clave viven en la **key policy**, independiente de las policies del servicio que guarda los datos.

Caso canónico: un administrador con acceso total a [[S3]] **no puede leer** objetos cifrados con **SSE-KMS** si no tiene permisos sobre la KMS Key.

## Dónde aparece

- [[KMS]] — casos de uso y gotchas
- [[s3-encryption]] — "Por qué SSE-KMS y no SSE-S3": con SSE-S3 las claves viven en S3, así que el admin **sí** lee los datos

## Dato de examen

- Enunciado con *"el equipo de operaciones administra el bucket pero no debe ver el contenido"* → **SSE-KMS con customer managed key**, nunca SSE-S3.
- SSE-S3 **no ofrece role separation**; tampoco rotación controlada ni auditoría por clave.

## Ver también

[[envelope-encryption]] · [[least-privilege]] · [[control-plane]] · [[data-plane]]
