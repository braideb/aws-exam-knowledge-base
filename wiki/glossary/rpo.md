---
title: RPO (Recovery Point Objective)
category: glossary
tags: [dr, resiliencia, backups, rds]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.13 HA vs. FT vs. DR.md", "raw/notas curso mejorado/10 Databases (SQL)/10.08 RDS Backups y Restore.md", "raw/notas curso mejorado/10 Databases (SQL)/10.09 RDS Read Replicas.md"]
updated: 2026-10-03
---

# RPO (Recovery Point Objective)

> **En una línea:** cuántos **datos** (medidos en tiempo) estás dispuesto a perder si algo falla.

## Definición

Mira **hacia atrás**: si el último punto recuperable es de hace 1 hora, el RPO es 1 hora. Lo fija la **frecuencia** de los backups o el tipo de replicación: la replicación [[synchronous-replication|síncrona]] da RPO ≈ 0 y la [[asynchronous-replication|asíncrona]] da lo que valga el [[replication-lag|lag]]. Va de la mano del [[rto|RTO]], pero mide otra cosa.

| Mecanismo (RDS/Aurora) | RPO típico |
|---|---|
| Snapshot manual diario | Hasta 24 h |
| Automated backups + transaction logs | ~5 min |
| Read replica / Global Database | Segundos (el lag) |
| Multi-AZ | 0 |

## Dónde aparece

- [[ha-ft-dr]] — definición, junto con RTO, y las 4 estrategias de DR
- [[RDS]] — PITR con RPO de ~5 minutos
- [[rds-automated-backups-vs-snapshots]] · [[rds-ha-options]]

## Dato de examen

- "Cuántos datos se pueden perder" → **RPO**. "Cuánto tiempo caído" → **RTO**.
- **Snapshots y backups mejoran el RPO, no el RTO** (restaurar es lento).

## Ver también

[[rto]] · [[point-in-time-recovery]] · [[replication-lag]]
