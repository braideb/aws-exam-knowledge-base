---
title: RTO (Recovery Time Objective)
category: glossary
tags: [dr, resiliencia, backups, rds]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.13 HA vs. FT vs. DR.md", "raw/notas curso mejorado/10 Databases (SQL)/10.08 RDS Backups y Restore.md", "raw/notas curso mejorado/10 Databases (SQL)/10.09 RDS Read Replicas.md"]
updated: 2026-10-03
---

# RTO (Recovery Time Objective)

> **En una línea:** cuánto **tiempo** podés estar caído hasta que el servicio vuelva a funcionar.

## Definición

Mira **hacia adelante**: desde la falla hasta la recuperación. Depende de **qué tan lista** está la alternativa: restaurar un backup tarda horas en una base grande, promover una réplica tarda minutos y un [[failover]] automático, segundos. Se mide junto con el [[rpo|RPO]].

## Dónde aparece

- [[ha-ft-dr]] — definición y estrategias de DR (de Backup & Restore a Active/Active)
- [[RDS]] — el restore es lento (mal RTO); una read replica promovida da un RTO bajo
- [[rds-automated-backups-vs-snapshots]]

## Dato de examen

- *"Recuperarse en minutos"* → **réplica promovible**, Multi-AZ o Global Database, **no** restaurar un snapshot.
- Un restore de RDS crea una **instancia nueva**: al RTO hay que sumarle reconfigurar la app con el endpoint nuevo.

## Ver también

[[rpo]] · [[failover]] · [[lazy-restore]]
