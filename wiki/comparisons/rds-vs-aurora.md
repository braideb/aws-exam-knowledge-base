---
title: RDS vs Aurora
category: comparison
tags: [rds, aurora, databases, storage, costos]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.03 RDS — Architecture.md", "raw/notas curso mejorado/10 Databases (SQL)/10.04 RDS — Costos.md", "raw/notas curso mejorado/10 Databases (SQL)/10.13 Aurora — Architecture.md", "raw/notas curso mejorado/10 Databases (SQL)/10.14 Aurora — Restore, Clone y Backtrack.md", "raw/doc oficial/Differences between Amazon Aurora and Amazon RDS.md", "raw/doc oficial/Amazon Aurora storage.md"]
updated: 2026-10-03
---

# RDS vs Aurora

[[Aurora]] es "parte de RDS" en la consola, pero tiene una arquitectura distinta: **storage compartido del cluster** en lugar de **un EBS por instancia**. Casi todas las diferencias salen de ahí.

## Tabla comparativa

| | **[[RDS]]** | **[[Aurora]]** |
|---|---|---|
| Motores | MySQL, MariaDB, PostgreSQL, **Oracle, SQL Server, Db2** | Solo **MySQL y PostgreSQL** compatibles |
| Storage | **EBS propio por instancia**; lo aprovisionás (GB, IOPS) | **Cluster volume compartido**, SSD, 6 copias en 3 AZs; crece solo (hasta 256 TiB ¹) |
| Réplicas | Standby (Multi-AZ, no se lee) + read replicas (async, endpoint propio) | Hasta **15 replicas** que leen **y** son target de failover |
| Failover | 60–120 s (Multi-AZ instance) · ~35 s (Multi-AZ cluster) | ~30 s |
| Endpoints | Un endpoint por instancia (cluster/reader en Multi-AZ cluster) | Cluster, reader, **custom**, instance (y global writer) |
| Restore | Instancia nueva | Cluster nuevo · **backtrack in-place** (MySQL) · **fast clone** |
| Serverless | No (según el curso) | **Aurora Serverless v2** (ACU, auto-pause) |
| Multi-región | Cross-region read replicas (promote manual) | **Global Database** (failover gestionado, < 1 s de lag) |
| Costo de storage | GB-mes **provisionado** | GB-mes **consumido** + I/O (Standard) o sin cargo de I/O (I/O-Optimized ¹) |
| Free tier | Sí (micro single-AZ) | El curso dice que **no** ² |

¹ Complemento de la doc oficial. ² Ver la nota en [[Aurora]].

## Cuándo usar X vs Y

- **RDS**: motor **Oracle / SQL Server / Db2**; cargas chicas donde el costo base de Aurora no se justifica; compatibilidad máxima con herramientas existentes.
- **Aurora**: MySQL/PostgreSQL que necesitan **más throughput** (AWS habla de hasta 6x MySQL/PostgreSQL estándar ¹), **failover rápido**, **muchas réplicas**, multi-región o **serverless**.
- Migrar de RDS MySQL/PostgreSQL a Aurora es, en general, un **snapshot restore**.

## Trampa típica del examen

- Confundir el **RDS Multi-AZ cluster** (1 writer + **2** readers, storage local por instancia) con **Aurora** (hasta 15 replicas, storage compartido).
- *"Agregar réplicas de lectura en segundos sin copiar datos"* → Aurora: las replicas se montan sobre el mismo volumen.
- *"Motor Oracle con failover automático"* → **RDS Multi-AZ**, no Aurora.
- *"Pagar solo por el storage usado, sin aprovisionarlo"* → Aurora.

> 📖 Lectura profunda: [[10.03 RDS — Architecture]] · [[10.13 Aurora — Architecture]]
