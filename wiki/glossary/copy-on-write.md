---
title: Copy-on-write
category: glossary
tags: [aurora, clone, storage]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.14 Aurora — Restore, Clone y Backtrack.md", "raw/doc oficial/Cloning a volume for an Amazon Aurora DB cluster.md", "raw/doc oficial/Amazon Aurora Fast Database Cloning.md"]
updated: 2026-10-03
---

# Copy-on-write

> **En una línea:** una copia que **comparte** los datos del original y solo duplica una página cuando alguno de los dos la **modifica**.

## Definición

Al crear la copia no se mueve ningún dato: las dos apuntan a las mismas páginas. Cuando el origen o la copia cambian una página, recién ahí se crea una versión propia. Por eso la copia es **instantánea**, su tiempo de creación **no depende del tamaño** y solo pagás el storage de lo que cambió. Es el mecanismo del **fast clone** de [[Aurora]] (hasta 15 clones copy-on-write por origen, en la misma región ¹).

¹ Complemento de la doc oficial.

## Dónde aparece

- [[Aurora]] — fast clone
- [[databases-cheat-sheet]]

## Dato de examen

*"Copia de una base de producción de 2 TB para test, en minutos y sin duplicar el storage"* → **Aurora clone**, no un restore de snapshot.

## Ver también

[[point-in-time-recovery]] · [[lazy-restore]]
