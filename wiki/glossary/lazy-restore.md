---
title: Lazy restore
category: glossary
tags: [ebs, snapshots, performance]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.11 EBS Snapshots, Restore y Fast Snapshot Restore (FSR).md"]
updated: 2026-10-03
---

# Lazy restore

> **En una línea:** un volumen creado desde un snapshot está **disponible al instante pero todavía vacío por detrás**: los bloques se traen de S3 a medida que se los pide.

## Definición

[[EBS]] va copiando el snapshot en segundo plano. Si la aplicación lee un bloque que todavía no llegó, EBS lo busca en ese momento en S3 — funciona, pero con **latencia mucho más alta**. De ahí el bajón de rendimiento de los primeros minutos. Un volumen creado **en blanco** no tiene este problema: rinde al máximo desde el segundo cero.

## Dónde aparece

- [[EBS]] — restore de snapshots
- [[dva-troubleshooting]] — "el volumen restaurado anda lento"
- También en: [[ec2-cheat-sheet]] · [[ha-ft-dr]] · [[RDS]] · [[rds-automated-backups-vs-snapshots]]

## Dato de examen

Dos formas de evitarlo, y las dos aparecen como opciones:
- **Forzar la lectura de todos los bloques** antes de poner el volumen en producción (`dd` en Linux). Gratis, pero lleva tiempo.
- **Fast Snapshot Restore (FSR)**, que hace la restauración instantánea. Cuesta extra y el límite es de **50 por región**, contando **cada par snapshot + AZ** como uno.

## Ver también

[[EBS]] · [[iops]]
