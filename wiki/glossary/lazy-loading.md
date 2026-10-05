---
title: Lazy loading
category: glossary
tags: [storage, performance, fsx, lustre, s3, caching]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/11 Advanced Storage/11.06 FSx for Lustre — Concepts.md"]
updated: 2026-10-05
---

# Lazy loading

> **En una línea:** **no traer los datos hasta que alguien los pide**: el primer acceso los carga y los siguientes ya los encuentran.

## Definición

Es el patrón contrario a precargar todo. Arrancar es instantáneo y barato, pero el **primer acceso a cada dato es lento**, porque recién en ese momento se lo va a buscar a la fuente. En [[FSx]] for Lustre, los objetos del bucket de S3 asociado **aparecen** en el file system desde el inicio, pero se copian recién cuando un cliente los lee por primera vez.

## Dónde aparece

- [[FSx]]: FSx for Lustre con repositorio S3
- [[efs-vs-fsx]]: la integración clave de Lustre
- También en: [[lazy-restore]], el mismo patrón aplicado a los volúmenes EBS restaurados desde un snapshot

## Dato de examen

- Lustre + S3: lazy load al **leer**, pero **sin sincronización** de vuelta. Para que los cambios lleguen a S3 hay que **exportarlos** (`hsm_archive`).
- *(A futuro)* Es también una estrategia de **caché** que el examen DVA pregunta con ElastiCache: lazy loading vs write-through.

## Ver también

[[lazy-restore]]
