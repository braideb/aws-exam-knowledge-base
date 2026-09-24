---
title: Delete Marker
category: glossary
tags: [s3, versioning, storage, borrado]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/What does Amazon S3 replicate - Amazon Simple Storage Service.md", "raw/notas curso mejorado/04 S3/04.03 S3 Object Versioning y MFA Delete.md", "raw/notas curso mejorado/04 S3/04.09 S3 Lifecycle Configuration.md"]
updated: 2026-09-24
---

# Delete Marker

> **En una línea:** un borrado **lógico y reversible** en un bucket con versioning.

## Definición

En un bucket de [[S3]] con versioning habilitado, un `DELETE` **sin version ID** no borra nada: crea una versión especial vacía —el delete marker— que pasa a ser la versión *current*. El objeto "desaparece" de los listados pero **todas sus versiones siguen ahí y siguen costando**.

- Borrar el **delete marker** → el objeto reaparece.
- `DELETE` **con** version ID → destrucción **permanente** de esa versión.

## Dónde aparece

- [[S3]] — sección Versioning y MFA Delete
- [[s3-storage-classes]] — lifecycle expira noncurrent versions y borra delete markers huérfanos
- También en: [[durability]] · [[worm]]

## Dato de examen

- Gotcha de lifecycle: si un bucket **sin** versioning tenía una regla de **expiración** y habilitás versioning, esa regla ahora solo **crea delete markers** — hay que agregar una regla de **noncurrent expiration** para que vuelva a borrar de verdad.
- Con [[worm|Object Lock]]: `DELETE` con version ID sobre versión bloqueada → **403**; `DELETE` simple → **200 + delete marker** (el lock no lo impide).
- La replicación de delete markers es **opt-in**, y un `DELETE` **con version ID en el source no se replica** nunca.

## Ver también

[[worm]] · [[durability]] · [[object-storage]]
