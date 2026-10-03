---
title: Point-in-time recovery (PITR)
category: glossary
tags: [backups, rds, aurora, dr]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.08 RDS Backups y Restore.md", "raw/doc oficial/Restoring a DB instance to a specified time for Amazon RDS - Amazon Relational Database Service.md"]
updated: 2026-10-03
---

# Point-in-time recovery (PITR)

> **En una línea:** restaurar la base a **cualquier momento** dentro del período de retención, no solo al de un snapshot.

## Definición

[[RDS]] lo arma con los **automated backups**: restaura el snapshot diario más cercano y **reproduce los transaction logs** (que sube a S3 cada 5 minutos) hasta la hora pedida. La granularidad es de ~5 minutos ([[rpo|RPO]] ≈ 5 min) y el último punto disponible es el `LatestRestorableTime`. Siempre crea una **instancia nueva**.

## Dónde aparece

- [[RDS]] — automated backups, retención de 0 a 35 días
- [[rds-automated-backups-vs-snapshots]]
- [[Aurora]] — mismo modelo, más el backtrack como alternativa in-place

## Dato de examen

- *"Volver a 5 minutos antes del error"* → **PITR**. Un snapshot manual solo vuelve al momento en que se tomó.
- PITR = **endpoint nuevo**. Para no cambiar de endpoint (Aurora MySQL) → **backtrack**.

## Ver también

[[rpo]] · [[rto]] · [[lazy-restore]]
