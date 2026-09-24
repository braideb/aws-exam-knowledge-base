---
title: Custom Metric
category: glossary
tags: [cloudwatch, metrics, observabilidad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/07 Monitoring and logging/07.01 CloudWatch — Architecture Concepts.md", "raw/notas curso mejorado/07 Monitoring and logging/07.02 CloudWatch Data (Namespace, Datapoint, Metric, Dimensions).md"]
updated: 2026-09-24
---

# Custom Metric

> **En una línea:** una métrica de CloudWatch que publicás vos (con el agent o con `PutMetricData`), en vez de que AWS la genere por su cuenta.

## Definición

Todo lo que no publica un servicio de AWS de forma nativa: memoria de una EC2 (vía CloudWatch Agent), métricas de una app, de un servidor on-premises o las que genera un [[metric-filter]]. Vive en un **namespace propio**, que no puede empezar con `AWS/`. Es la única que admite **high resolution** (1 s) y **se cobra**.

## Dónde aparece

- [[CloudWatch]] — arquitectura (agent/API) y costos
- [[observability-costs]] — cada combinación métrica + [[dimension]] se factura aparte
- [[CloudWatchLogs]] — los metric filters publican custom metrics
- También en: [[dva-troubleshooting]] · [[high-resolution-metric]] · [[metric-filter]] · [[percentile]]

## Dato de examen

- "Métrica de memoria de EC2" → custom metric vía **CloudWatch Agent**.
- AWS **no agrega** custom metrics a través de dimensions distintas: si querés un total, publicalo vos.
- Para reaccionar en menos de 1 minuto hace falta una custom metric de [[high-resolution-metric|alta resolución]].

## Ver también

[[dimension]] · [[high-resolution-metric]] · [[high-cardinality]]
