---
title: Near Real Time
category: glossary
tags: [streaming, observabilidad, latencia]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/07 Monitoring and logging/07.06 CloudWatch Logs — Subscriptions y Aggregation.md", "raw/notas curso mejorado/07 Monitoring and logging/07.08 VPC Flow Logs.md"]
updated: 2026-09-24
---

# Near Real Time

> **En una línea:** "casi en tiempo real" — los datos llegan con una demora corta (segundos a un minuto) por buffering, no evento por evento.

## Definición

El examen usa la distinción como pista: **real time** = cada evento se procesa apenas llega (Lambda, Kinesis Data Streams); **near real time** = se junta un buffer y se entrega en lote (el ejemplo canónico es **Firehose**, que en el curso bufferiza 60 s). Por debajo de ambos quedan los procesos que tardan minutos u horas y **no** son real time en ningún sentido.

| Latencia | Ejemplos |
|---|---|
| Real time | Subscription → Lambda, Kinesis Data Streams, [[EventBridge]] |
| Near real time | Firehose |
| No real time | [[vpc-flow-logs\|VPC Flow Logs]] (1–10 min), [[CloudTrail]] (~15 min), `CreateExportTask` (hasta 12 h) |

## Dónde aparece

- [[CloudWatchLogs]] — destinos de subscriptions
- [[vpc-flow-logs]] — destino Firehose
- [[subscription-filter]]

## Dato de examen

- Si la pregunta dice "near real time" y "S3" → casi siempre **Firehose**. Si dice "real time" y "procesar cada evento" → **Lambda** o **Kinesis Data Streams**.

## Ver también

[[subscription-filter]]
