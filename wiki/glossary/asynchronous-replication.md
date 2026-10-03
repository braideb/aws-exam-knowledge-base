---
title: Replicación asíncrona (asynchronous replication)
category: glossary
tags: [replicacion, rds, read-replicas, aurora]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.09 RDS Read Replicas.md", "raw/notas curso mejorado/10 Databases (SQL)/10.17 Aurora Global Database.md"]
updated: 2026-10-03
---

# Replicación asíncrona (asynchronous replication)

> **En una línea:** la escritura se confirma en la primary y **después** se copia a la réplica.

## Definición

La primary no espera a la réplica: responde enseguida y replica en segundo plano. Las escrituras no se frenan y la réplica puede estar lejos (otra región), pero va **un poco atrasada** ([[replication-lag]]). Si la primary muere, se pierde lo que no llegó a copiarse ([[rpo|RPO]] > 0). Lo contrario es la [[synchronous-replication|replicación síncrona]].

## Dónde aparece

- [[RDS]] — read replicas (misma región o cross-region)
- [[Aurora]] — Global Database (a nivel storage, < 1 s entre regiones)
- [[rds-ha-options]]

## Dato de examen

- *"Asíncrono"* en una opción de RDS → **read replicas**.
- Una réplica asíncrona **puede devolver datos viejos**: la app tiene que tolerarlo.

## Ver también

[[synchronous-replication]] · [[replication-lag]] · [[eventual-consistency]]
