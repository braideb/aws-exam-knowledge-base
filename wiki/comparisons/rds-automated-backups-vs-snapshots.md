---
title: RDS — Automated backups vs Snapshots manuales
category: comparison
tags: [rds, backups, snapshots, pitr, rpo, rto, dr]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.08 RDS Backups y Restore.md", "raw/doc oficial/Restoring a DB instance to a specified time for Amazon RDS - Amazon Relational Database Service.md", "raw/doc oficial/Sharing encrypted snapshots for Amazon RDS - Amazon Relational Database Service.md", "raw/doc oficial/Encrypting Amazon RDS resources - Amazon Relational Database Service.md", "raw/doc oficial/RDS Cross-Region Read Replicas – DR & Scaling.md"]
updated: 2026-10-03
---

# RDS — Automated backups vs Snapshots manuales

Los dos se guardan en [[S3]] gestionado por AWS (no lo ves en tu consola de S3), son **incrementales** y cubren **toda la instancia**. Cambian en **quién los dispara, cuánto duran y a qué punto podés volver**.

## Tabla comparativa

| | **Automated backups** | **Snapshots manuales** |
|---|---|---|
| Disparo | Automático: 1 por día en la **backup window** | Manual, script o AWS Backup |
| Transaction logs | ✅ a S3 **cada 5 minutos** | ❌ |
| Restore a… | **Cualquier punto** dentro de la retención ([[point-in-time-recovery\|PITR]]) | Solo al **momento del snapshot** |
| [[rpo\|RPO]] | **~5 minutos** | El intervalo entre snapshots |
| Retención | **0–35 días** (0 = deshabilitado); se borran solos | **Indefinida**, hasta que los borres |
| ¿Sobreviven al borrar la instancia? | Si los retenés, igual **caducan** | ✅ Sí |
| Compartir con otra cuenta | ❌ (primero copiarlos a snapshot manual) ¹ | ✅ (cifrado: con **customer managed key**) |
| Copia cross-region | Replicación de backups cross-region (opt-in) | Copia de snapshot (KMS key de la región destino) |

¹ Complemento: no viene del curso ni de un clipping curado.

En los dos casos, **restaurar crea una instancia nueva con un endpoint nuevo** y es **lento** (mal [[rto|RTO]]). La instancia restaurada además carga los bloques desde S3 en segundo plano ([[lazy-restore]]), según la doc oficial de PITR.

## Cuándo usar X vs Y

| Escenario | Respuesta |
|---|---|
| Volver a 10 minutos antes de un `DROP TABLE` | **Automated backups** → PITR |
| Conservar una copia **más de 35 días** (compliance, auditoría) | **Snapshot manual** |
| Borrar la instancia sin perder los datos | **Snapshot final** al borrar |
| Llevar la base a otra cuenta | **Snapshot manual** compartido (+ key compartida si está cifrado) |
| Pasar una base **sin cifrar** a **cifrada** | Snapshot → **copia cifrada** → restore |
| DR en otra región con RPO de minutos | **Replicación cross-region de automated backups** |
| RTO de minutos, no de horas | Ninguno de los dos: **read replica** promovible o Multi-AZ ([[rds-ha-options]]) |

## Trampa típica del examen

- *"Los backups se borraron solos al eliminar la instancia, aunque elegí retenerlos"* → los automated backups retenidos **igual caducan** según su retención. Lo que dura para siempre es el **snapshot final**.
- *"Busco el bucket de los backups de RDS en S3"* → no existe en tu cuenta: es S3 **gestionado por AWS**.
- *"El backup no afecta el rendimiento"* → solo con **Multi-AZ**, porque se toma desde la standby. En single-AZ hay una breve pausa de I/O.
- *"Restaurar sin cambiar la configuración de la app"* → con restore **no se puede** (endpoint nuevo). En Aurora MySQL, **backtrack**.

> 📖 Lectura profunda: [[10.08 RDS Backups y Restore]]
