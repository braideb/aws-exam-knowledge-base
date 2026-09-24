---
title: Throughput
category: glossary
tags: [storage, ebs, performance, red]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.04 Storage Refresh.md"]
updated: 2026-09-24
---

# Throughput

> **En una línea:** la **cantidad de datos por segundo** que mueve un volumen o una interfaz, normalmente en MB/s.

## Definición

Es el producto del tamaño de bloque por las [[iops|IOPS]]. Un volumen puede tener muchas IOPS y poco throughput (bloques chicos, mucha operación suelta) o al revés (bloques grandes leídos de corrido). Los perfiles de acceso **aleatorio** se miden mejor en IOPS y los **secuenciales** en throughput — por eso los HDD `st1`/`sc1` publican su rendimiento en MB/s por TB.

## Dónde aparece

- [[storage-types]] — la relación entre las tres métricas
- [[ebs-volume-types]] — 250 MB/s (gp2) vs 1.000 MB/s (gp3, io1, io2) vs 4.000 MB/s (io2 Block Express)
- También en: [[ec2-cheat-sheet]] · [[CloudWatch]] · [[EBS]] · [[S3]] · [[burst-credit]] · [[ebs-optimized]] · [[ec2-instance-types]] · [[instance-store-vs-ebs]] · [[iops]] · [[prefix]]

## Dato de examen

- La diferencia que más cae entre los dos SSD de propósito general: **gp2 tope 250 MB/s, gp3 tope 1.000 MB/s**. Si el escenario pide throughput alto sin pagar IOPS aprovisionadas → **gp3**.
- Palabras como "streaming", "secuencial", "logs" o "big data" apuntan a throughput, no a IOPS → **st1**.

## Ver también

[[iops]] · [[ebs-optimized]] · [[burst-credit]]
