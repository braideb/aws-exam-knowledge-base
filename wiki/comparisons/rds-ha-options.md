---
title: RDS — Multi-AZ vs Read Replicas vs Aurora (opciones de HA y escalado)
category: comparison
tags: [rds, aurora, multi-az, read-replicas, global-database, ha, dr, rpo, rto]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.06 RDS Multi-AZ Instance.md", "raw/notas curso mejorado/10 Databases (SQL)/10.07 RDS Multi-AZ Cluster.md", "raw/notas curso mejorado/10 Databases (SQL)/10.09 RDS Read Replicas.md", "raw/notas curso mejorado/10 Databases (SQL)/10.13 Aurora — Architecture.md", "raw/notas curso mejorado/10 Databases (SQL)/10.17 Aurora Global Database.md", "raw/doc oficial/Amazon RDS and Aurora High Availability Guide - Multi-AZ, Read Replicas, RDS Proxy, and Global Database Failover.md", "raw/doc oficial/RDS Multi-AZ Deployment – Failover, Sync Replication & HA Guide.md", "raw/doc oficial/RDS Cross-Region Read Replicas – DR & Scaling.md"]
updated: 2026-10-03
---

# RDS — Multi-AZ vs Read Replicas vs Aurora

Cinco mecanismos con nombres parecidos que resuelven **fallas distintas**. La pregunta de examen casi siempre es: *¿esto escala lecturas, da failover automático, o las dos cosas?*

## Tabla comparativa

| | **[[RDS]] Multi-AZ instance** | **RDS Multi-AZ cluster** | **RDS read replica** | **[[Aurora]] replicas** | **Aurora Global Database** |
|---|---|---|---|---|---|
| Réplicas | 1 standby | 2 readers | Hasta 15 ¹ | Hasta 15 | 1 + 10 regiones (16 replicas c/u) |
| ¿Sirve lecturas? | ❌ | ✅ | ✅ | ✅ | ✅ (secundarias read-only) |
| Replicación | [[synchronous-replication\|Síncrona]] (storage) | Semisíncrona (≥1 reader confirma) | [[asynchronous-replication\|Asíncrona]] | Storage compartido (6 copias); lag < 100 ms | Storage, cross-region, **< 1 s** |
| [[failover\|Failover]] automático | ✅ **60–120 s** (DNS) | ✅ **~35 s** | ❌ **promote manual** | ✅ **~30 s** | Failover gestionado (~1 min) o switchover |
| Alcance | Misma región | Misma región (3 AZs) | Misma región o **cross-region** | Misma región | **Multi-región** |
| Motores | Todos | MySQL, PostgreSQL | Todos | Aurora MySQL/PostgreSQL | Aurora MySQL/PostgreSQL |
| [[rpo\|RPO]] ante falla | 0 | ~0 | Lo que tenga de [[replication-lag\|lag]] | 0 dentro de la región | ~1 s |
| Protege contra **corrupción** | ❌ | ❌ | ❌ (la replica) | ❌ | ❌ |

¹ El curso dice 5; la doc actual dice 15 combinadas (ver [[RDS]]).

**Contra corrupción o borrados**, ninguna réplica sirve: hace falta **PITR** o snapshots ([[rds-automated-backups-vs-snapshots]]) o, en Aurora MySQL, **backtrack**.

## Cuándo usar X vs Y

| Escenario | Respuesta |
|---|---|
| HA ante caída de una AZ, cualquier motor, no hace falta leer de la réplica | **Multi-AZ instance** |
| HA + escalar lecturas en MySQL/PostgreSQL, failover más rápido, sin migrar a Aurora | **Multi-AZ cluster** |
| Escalar lecturas (reportes, analytics) o acercar lecturas a otra región | **Read replicas** (cross-region si hace falta) |
| DR cross-region para Oracle/SQL Server/MySQL en RDS | **Cross-region read replica** + promote manual (o backups cross-region) |
| Muchas réplicas de lectura que también sean target de failover | **Aurora** |
| DR multi-región con RPO de segundos y failover gestionado | **Aurora Global Database** |
| Failover más corto para los clientes y menos conexiones | **RDS Proxy** delante de cualquiera de los anteriores |

## Trampa típica del examen

- *"La primary está saturada de lecturas y la standby de Multi-AZ no hace nada"* → la standby de **Multi-AZ instance no se lee**. Agregar read replicas, pasar a Multi-AZ cluster o a Aurora.
- *"Failover automático a otra región"* con RDS clásico → **no existe**: las cross-region read replicas se **promueven a mano**. Automático multi-región = **Aurora Global Database**.
- *"¿Multi-AZ protege si cae la región?"* → **No**. Solo protege ante la caída de una AZ.
- *"Síncrono"* en una opción de RDS clásico = Multi-AZ. *"Asíncrono"* = read replicas.
- Una read replica promovida **corta la replicación** y queda como instancia independiente, con su **propio endpoint**.

> 📖 Lectura profunda: [[10.06 RDS Multi-AZ Instance]] · [[10.07 RDS Multi-AZ Cluster]] · [[10.09 RDS Read Replicas]] · [[10.13 Aurora — Architecture]] · [[10.17 Aurora Global Database]]
