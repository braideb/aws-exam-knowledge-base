---
title: HPC (High-Performance Computing)
category: glossary
tags: [ec2, rendimiento, networking, placement-groups]
exam: [SAA-C03, DVA-C02]
sources: ["raw/notas curso mejorado/09 Advanced EC2/09.09 Cluster Placement Groups.md", "raw/notas curso mejorado/09 Advanced EC2/09.12 Enhanced Networking (SR-IOV, ENA, EFA).md", "raw/notas curso mejorado/11 Advanced Storage/11.06 FSx for Lustre — Concepts.md"]
updated: 2026-10-05
---

# HPC (High-Performance Computing)

> **En una línea:** cómputo que reparte un problema grande entre muchos nodos que **se comunican todo el tiempo**, así que la red entre ellos es el cuello de botella.

## Definición

Simulaciones, análisis científico, modelado. Lo que importa no es solo la CPU de cada nodo, sino la **latencia baja y constante** y el ancho de banda **entre** nodos. Muchas de estas apps usan **MPI** (Message Passing Interface) para comunicarse.

## Dónde aparece

- [[placement-groups]]: cluster placement group, el caso de uso típico
- [[FSx]]: FSx for Lustre, el file system de HPC
- También en: [[enhanced-networking]] (EFA) · [[EC2]] · [[ec2-cheat-sheet]] · [[efs-vs-fsx]] · [[ec2-instance-types]] · [[storage-types]] · [[virtualization]] · [[posix]]

## Dato de examen

- Escenario HPC → **cluster placement group** + [[enhanced-networking]]. Si menciona **MPI** o comunicación entre nodos de muy baja latencia → **EFA** (Elastic Fabric Adapter).
- Escenario HPC que pide un **file system** de muy alto rendimiento (cientos de GB/s), muchas veces sobre datos de S3 → **FSx for Lustre**.

## Ver también

[[enhanced-networking]] · [[throughput]]
