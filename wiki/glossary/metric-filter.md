---
title: Metric Filter
category: glossary
tags: [cloudwatch-logs, cloudwatch, observabilidad, monitoring]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-09-23
---

# Metric Filter

> **En una línea:** un patrón que escanea CloudWatch Logs y convierte las líneas que coinciden en una métrica — lo que convierte a Logs de depósito pasivo en monitoreo activo.

## Definición

Regla configurada sobre un log group que busca un patrón (por ejemplo, la palabra `ERROR`) en cada evento entrante y, al coincidir, incrementa una métrica de CloudWatch. Esa métrica se comporta como cualquier otra: puede tener una alarm que dispare una notificación. Es el puente entre [[CloudWatchLogs]] (texto) y [[CloudWatch]] (métricas y alarms).

## Dónde aparece

- [[CloudWatchLogs]] — sección propia "Metric Filters"
- [[CloudWatch]], [[CloudTrail]] — metric filters + alarms sobre eventos de auditoría
- [[dimension]], [[high-cardinality]] — las dimensions extraídas por un metric filter crean una variación nueva por cada par único; usar dimensions en un metric filter impide configurar el *default value*
- [[dva-troubleshooting]], [[observability-costs]]
- [[vpc-flow-logs]] — para alarmar sobre rechazos cuando el destino es CloudWatch Logs
- También en: [[VPC]]

## Dato de examen

- "Alertar cuando aparece X en los logs" → **metric filter + alarm + SNS**.
- Cada combinación única de dimension generada por un metric filter se factura aparte — cuidado con la [[high-cardinality|alta cardinalidad]].

## Ver también

[[CloudWatchLogs]] · [[dimension]] · [[high-cardinality]]
