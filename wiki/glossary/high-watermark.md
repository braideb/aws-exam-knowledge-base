---
title: High watermark (facturación de storage)
category: glossary
tags: [aurora, costos, storage]
exam: [SAA-C03]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.13 Aurora — Architecture.md", "raw/doc oficial/Amazon Aurora storage.md"]
updated: 2026-10-03
---

# High watermark (facturación de storage)

> **En una línea:** cobrar por el **máximo** de storage que llegaste a usar, aunque después hayas liberado espacio.

## Definición

Así facturaban las versiones viejas de [[Aurora]]: si llegaste a 50 GB y bajaste a 40 GB, seguías pagando 50 GB. Podías reusar el espacio liberado, pero para pagar menos había que **migrar a un cluster nuevo** (dump + restore).

> ⚠️ Outdated: en las versiones actuales de Aurora el cluster volume **se achica al borrar datos** (*dynamic resizing*) y pagás lo que usás. El curso ya avisa que el modelo cambió.

## Dónde aparece

- [[Aurora]] — facturación del storage

## Dato de examen

Si aparece en una pregunta vieja: *"bajar el costo de storage tras borrar muchos datos"* → cluster nuevo + migración. Con las versiones actuales alcanza con borrar.

## Ver también

[[copy-on-write]] · [[observability-costs]]
