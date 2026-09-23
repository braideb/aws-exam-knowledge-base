---
title: IOPS
category: glossary
tags: [storage, ebs, performance]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.04 Storage Refresh.md"]
updated: 2026-09-23
---

# IOPS

> **En una línea:** **operaciones de entrada/salida por segundo** — cuántas lecturas o escrituras aguanta un volumen en un segundo.

## Definición

Cada operación mueve un bloque de tamaño fijo, y ese tamaño **depende del tipo de disco**: **16 KB** en los SSD de EBS y **1 MB** en los HDD. De ahí sale la fórmula que relaciona las tres métricas de rendimiento:

> **IO block size × IOPS = [[throughput]]**

Con 16 KB y 100 IOPS son ~1,6 MB/s; con 1 MB y 500 IOPS son 500 MB/s. Por eso el mismo número de IOPS significa cosas muy distintas según el tipo de volumen.

## Dónde aparece

- [[storage-types]] — la fórmula y las tres variables
- [[ebs-volume-types]] — el tope de IOPS de cada tipo
- [[instance-store-vs-ebs]] — la escalera de decisión
- También en: [[EBS]] · [[ec2-cheat-sheet]]

## Dato de examen

- Un volumen tiene un tope de IOPS **y** uno de throughput por separado: podés chocar con uno antes que con el otro.
- Números a tener: **gp2/gp3 → 16.000**, **io1/io2 → 64.000**, **io2 Block Express → 256.000**, tope por instancia **~260.000**, y por encima de eso solo queda el instance store.

## Ver también

[[throughput]] · [[burst-credit]] · [[ebs-optimized]]
