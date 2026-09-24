---
title: Durability (durabilidad)
category: glossary
tags: [s3, storage, resiliencia, sla]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Understanding and managing Amazon S3 storage classes - Amazon Simple Storage Service.md", "raw/notas curso mejorado/04 S3/04.08 S3 Object Storage Classes.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.08 S3 Buckets — Basics.md"]
updated: 2026-09-24
---

# Durability (durabilidad)

> **En una línea:** la probabilidad de que el dato **no se pierda**. Distinta de poder leerlo ahora.

## Definición

Mide la pérdida permanente de datos. En [[S3]] es de **11 nueves** (99.999999999%) — y es igual en **todas** las storage classes, incluida One Zone-IA.

**Durabilidad ≠ [[availability|disponibilidad]]:**
- Durabilidad = *"no se pierde"*.
- Disponibilidad = *"puedo llegar a él ahora"*.

## Dónde aparece

- [[S3]] — "durabilidad 11 nueves vs. disponibilidad 99.99% en Standard"
- [[s3-storage-classes]] — todas las clases comparten los 11 nueves; cambian disponibilidad, costo y latencia
- [[ephemeral-storage]] — el extremo opuesto: almacenamiento que no promete durar
- También en: [[availability]] · [[delete-marker]] · [[etag]] · [[eventual-consistency]] · [[object-storage]] · [[worm]]

## Dato de examen

- Trampa clásica: **One Zone-IA también tiene 11 nueves de durabilidad**… *dentro de su única AZ*. Si esa AZ se destruye, los datos se pierden igual → nunca para datos **irreemplazables**.
- La durabilidad de S3 se apoya en replicar en **≥3 AZs** ([[region-resilient]]); no protege contra un borrado tuyo → para eso están **versioning** y [[worm|Object Lock]].

## Ver también

[[availability]] · [[region-resilient]] · [[delete-marker]]
