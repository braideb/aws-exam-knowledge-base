---
title: Subscription Filter
category: glossary
tags: [cloudwatch-logs, logging, streaming, observabilidad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/07 Monitoring and logging/07.06 CloudWatch Logs — Subscriptions y Aggregation.md"]
updated: 2026-09-24
---

# Subscription Filter

> **En una línea:** la regla de un log group de CloudWatch Logs que reenvía sus eventos, en vivo, a otro servicio.

## Definición

Se define **por log group** con: un **pattern** (qué eventos), el **ARN del destino**, la **distribution** y el **IAM role** que usa CloudWatch Logs para escribir. Destinos: **Lambda**, **Kinesis Data Streams**, **Firehose** u **OpenSearch**. Máx **2 por log group**. No confundir con el [[metric-filter]]: ese convierte logs en **métricas**; el subscription filter **entrega los logs** a otro lado.

## Dónde aparece

- [[CloudWatchLogs]] — subscriptions y log aggregation multi-cuenta
- También en: [[dva-troubleshooting]] · [[metric-filter]] · [[near-real-time]]

## Dato de examen

- Real time → Lambda / Kinesis Data Streams. [[near-real-time|Near real time]] a S3 → Firehose.
- Centralizar logs de varias cuentas → subscription filters hacia un **destination** en la cuenta central ([[cross-account]]).
- Llevar logs a S3 **ya** → subscription + Firehose, no `CreateExportTask` (hasta 12 h).

## Ver también

[[metric-filter]] · [[near-real-time]] · [[cross-account]]
