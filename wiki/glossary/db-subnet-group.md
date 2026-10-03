---
title: DB subnet group
category: glossary
tags: [rds, aurora, vpc, subnets]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.03 RDS — Architecture.md"]
updated: 2026-10-03
---

# DB subnet group

> **En una línea:** la lista de subnets de una VPC donde RDS puede ubicar las instancias de una base.

## Definición

Lo creás antes de lanzar una instancia de [[RDS]] o un cluster de [[Aurora]]. Tiene que cubrir **al menos dos AZs**, porque con Multi-AZ la primary y la standby (o los readers) quedan en **AZs distintas**, en subnets elegidas de esta lista. Si las subnets son públicas, la base **puede** hacerse pública (mala práctica). Best practice del curso: **un DB subnet group por implementación**.

## Dónde aparece

- [[RDS]] — arquitectura dentro de la VPC
- [[databases-cheat-sheet]]

## Dato de examen

La base queda en las subnets del subnet group, pero **quién llega** a ella lo decide el **security group** de la instancia, no el subnet group.

## Ver también

[[multi-az]] · [[public-subnet]] · [[cidr]]
