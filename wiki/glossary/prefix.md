---
title: Prefix (prefijo)
category: glossary
tags: [s3, performance, storage, policies]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Bucket policy examples using condition keys - Amazon Simple Storage Service.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.08 S3 Buckets — Basics.md", "raw/doc oficial/What is Amazon S3 - Amazon Simple Storage Service.md"]
updated: 2026-09-24
---

# Prefix (prefijo)

> **En una línea:** la parte inicial de una key de S3 — **no es una carpeta**, pero determina la performance.

## Definición

En [[S3]] la estructura es **plana**: `logs/2026/07/22/app.log` es una única key que contiene barras. Todo lo anterior al nombre final es el **prefijo**, y las "carpetas" que muestra la consola son una ilusión construida a partir de él.

Aun así el prefijo es real para dos cosas: **escalar [[throughput]]** y **acotar permisos**.

## Dónde aparece

- [[S3]] — estructura plana, Performance, condition key `s3:prefix`
- [[s3-storage-classes]] — reglas de lifecycle aplicadas a prefijos o tags
- También en: [[multipart-upload]] · [[object-storage]]

## Dato de examen

- **Throughput por prefijo**: **3.500** PUT/COPY/POST/DELETE y **5.500** GET/HEAD por segundo **por prefijo**. Se escala repartiendo la carga entre **varios prefijos** — por eso los datasets se estructuran como `logs/2026/07/22/…`.
- **`s3:prefix`** como condition limita `ListBucket` a un prefijo → así se da acceso "a una carpeta" sin exponer el bucket entero.
- Un prefijo **no** es una unidad de facturación ni de permisos por sí solo: sin una policy que lo restrinja, cualquiera con acceso al bucket lo ve.

## Ver también

[[object-storage]] · [[multipart-upload]] · [[least-privilege]]
