---
title: Topology-aware
category: glossary
tags: [arquitectura, resiliencia, placement-groups, replicacion]
exam: [SAA-C03, DVA-C02]
sources: ["raw/notas curso mejorado/09 Advanced EC2/09.11 Partition Placement Groups.md"]
updated: 2026-09-30
---

# Topology-aware

> **En una línea:** una aplicación que **sabe dónde está físicamente cada nodo** y usa esa información para decidir dónde replicar.

## Definición

Una app distribuida que no conoce la topología podría poner las tres copias de un dato en hardware que falla junto. Una app topology-aware recibe la ubicación de cada nodo y replica en [[fault-domain|fault domains]] distintos. Ejemplos típicos: **HDFS, HBase, Cassandra**.

## Dónde aparece

- [[EC2]]: resumen de placement groups
- [[placement-groups]]: los partition placement groups exponen a qué partición pertenece cada instancia, justamente para estas apps

## Dato de examen

- "HDFS / HBase / Cassandra a gran escala en EC2" → **partition placement group**.

## Ver también

[[fault-domain]] · [[blast-radius]]
