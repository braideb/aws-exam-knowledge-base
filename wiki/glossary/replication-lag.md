---
title: Replication lag
category: glossary
tags: [replicacion, rds, aurora, troubleshooting]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.09 RDS Read Replicas.md", "raw/doc oficial/Amazon RDS and Aurora High Availability Guide - Multi-AZ, Read Replicas, RDS Proxy, and Global Database Failover.md"]
updated: 2026-10-03
---

# Replication lag

> **En una línea:** cuánto va **atrasada** una réplica respecto de la primary.

## Definición

En la [[asynchronous-replication|replicación asíncrona]], la réplica aplica los cambios después que la primary. El retraso (segundos, a veces más) crece con las escrituras pesadas, las transacciones largas, la distancia entre regiones y las réplicas en **cascada** (réplica de réplica). Se mide en CloudWatch: `ReplicaLag` (RDS), `AuroraReplicaLag` (típicamente < 100 ms) y `AuroraGlobalDBReplicationLag` ¹.

¹ Complemento de la doc oficial.

## Dónde aparece

- [[RDS]] — read replicas y cascada
- [[Aurora]] — replicas y Global Database
- [[rds-ha-options]]
- También en: [[ha-ft-dr]] · [[dva-troubleshooting]] · [[asynchronous-replication]]

## Dato de examen

- El lag es el **[[rpo|RPO]] real** si promovés una réplica en una emergencia.
- También **alarga el failover**: la réplica tiene que aplicar lo pendiente antes de aceptar escrituras.
- *"Los usuarios leen datos desactualizados desde la réplica"* → lag: es normal en replicación asíncrona.

## Ver también

[[asynchronous-replication]] · [[rpo]] · [[eventual-consistency]]
