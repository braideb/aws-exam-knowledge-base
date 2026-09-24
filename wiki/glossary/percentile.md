---
title: Percentile (p95, p99)
category: glossary
tags: [cloudwatch, metrics, statistics, observabilidad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/07 Monitoring and logging/07.03 CloudWatch — Resolution, Retention y Statistics.md"]
updated: 2026-09-24
---

# Percentile (p95, p99)

> **En una línea:** el valor por debajo del cual cae un porcentaje dado de los datapoints — p95 = "el 95% de las requests fue igual o más rápido que esto".

## Definición

Una statistic de CloudWatch, igual que Average, Min, Max o Sum. Sirve cuando el **promedio esconde la cola**: si el 5% de las requests tarda 10 s, el Average puede verse sano, pero el p95 lo muestra. El curso menciona p95 y p97.5; en la práctica se usan p90, p95, p99.

## Dónde aparece

- [[CloudWatch]] — statistics
- [[CloudWatchLogs]] — percentiles sobre métricas de metric filters (solo si nunca publican valores negativos)
- También en: [[dva-troubleshooting]]

## Dato de examen

- "Alertar cuando **algunos** usuarios sufren latencia alta aunque el promedio esté bien" → alarm sobre **p95/p99**, no Average.

## Ver también

[[custom-metric]] · [[dimension]]
