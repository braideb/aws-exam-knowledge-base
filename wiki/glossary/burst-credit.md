---
title: Burst credit
category: glossary
tags: [ebs, ec2, performance, gp2]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.06 EBS Volume Types - General Purpose SSD (gp2 y gp3).md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.08 EBS Volume Types - HDD (st1 y sc1).md", "raw/notas curso mejorado/11 Advanced Storage/11.02 EFS — Performance, Throughput y Storage Classes.md", "raw/notas curso mejorado/11 Advanced Storage/11.07 FSx for Lustre — Storage, Performance y Arquitectura.md"]
updated: 2026-10-05
---

# Burst credit

> **En una línea:** una **ficha prepaga de rendimiento**: mientras te queden, el volumen (o la instancia) va más rápido que su baseline; cuando se acaban, caés al baseline.

## Definición

El modelo del "balde": se rellena a un ritmo fijo y cada operación gasta fichas. En **gp2** un crédito son **3 por segundo por GB** (mínimo 100), el balde arranca lleno con **5,4 millones** de créditos y permite ráfagas de hasta **3.000 [[iops|IOPS]]**. Los HDD `st1`/`sc1` y las instancias de la familia **T** usan la misma mecánica con otros números.

## Dónde aparece

- [[ebs-volume-types]] — gp2, st1 y sc1 funcionan con créditos; **gp3 no**
- [[ec2-instance-types]] — las instancias T son el *burst pool* de cómputo
- [[dva-troubleshooting]] — "el volumen se puso lento" suele ser el balde vacío
- [[EFS]] — el throughput mode **Bursting**: créditos de throughput que escalan con el tamaño del file system
- [[FSx]] — FSx for Lustre: burst hasta 1.300 MB/s por TiB
- También en: [[CloudWatch]] · [[EBS]] · [[ec2-cheat-sheet]] · [[iops]] · [[throughput]]

## Dato de examen

- **A partir de 1 TB el sistema de créditos de gp2 deja de importar**: el baseline (1.000 × 3 = 3.000 IOPS) ya iguala al tope de burst.
- Síntoma típico: un volumen chico que anduvo bien un rato y **cayó a ~100 IOPS**. Es gp2 con el balde en cero → agrandar el volumen o migrar a **gp3**, que da 3.000 IOPS fijas sin créditos.

## Ver también

[[iops]] · [[throughput]] · [[ebs-volume-types]]
