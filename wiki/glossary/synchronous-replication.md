---
title: Replicación síncrona (synchronous replication)
category: glossary
tags: [replicacion, rds, aurora, durabilidad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.06 RDS Multi-AZ Instance.md", "raw/notas curso mejorado/10 Databases (SQL)/10.09 RDS Read Replicas.md", "raw/notas curso mejorado/10 Databases (SQL)/10.13 Aurora — Architecture.md"]
updated: 2026-10-03
---

# Replicación síncrona (synchronous replication)

> **En una línea:** la escritura se considera hecha (**committed**) recién cuando llegó **también** a la copia.

## Definición

La primary escribe y espera la confirmación de la réplica antes de responder. La copia tiene **exactamente** los mismos datos ([[rpo|RPO]] = 0), a cambio de algo más de latencia por escritura. Por eso se usa **entre AZs cercanas** y no entre regiones lejanas. Lo contrario es la [[asynchronous-replication|replicación asíncrona]].

## Dónde aparece

- [[RDS]] — Multi-AZ instance: primary → standby, a nivel storage
- [[Aurora]] — la primary escribe en 6 nodos de storage repartidos en 3 AZs
- [[rds-ha-options]] · [[ha-ft-dr]] · [[global-infrastructure]]

## Dato de examen

En RDS (sin contar Aurora): **síncrono = Multi-AZ**, **asíncrono = read replicas**. El Multi-AZ **cluster** es "semisíncrono": confirma cuando responde **al menos uno** de los dos readers.

## Ver también

[[asynchronous-replication]] · [[replication-lag]] · [[multi-az]]
