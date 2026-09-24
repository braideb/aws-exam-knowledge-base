---
title: High-Resolution Metric
category: glossary
tags: [cloudwatch, metrics, alarms, observabilidad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/07 Monitoring and logging/07.03 CloudWatch — Resolution, Retention y Statistics.md", "raw/notas curso mejorado/07 Monitoring and logging/07.04 CloudWatch Alarms.md"]
updated: 2026-09-24
---

# High-Resolution Metric

> **En una línea:** una [[custom-metric|custom metric]] de CloudWatch con granularidad de **1 segundo** en vez de los 60 s estándar.

## Definición

Se publica con `StorageResolution = 1`. Permite leer los datos en períodos de 1, 5, 10 o 30 s (además de múltiplos de 60) y crear **high resolution alarms** de **10 o 30 s**. Cuesta más y el detalle sub-minuto se retiene solo **3 horas**; después se agrega a 1 minuto.

| | Standard | High resolution |
|---|---|---|
| Granularidad | 60 s | 1 s |
| Alarm mínima | 60 s | 10 s |
| Retención del detalle | 15 días (1 min) | 3 horas |

## Dónde aparece

- [[CloudWatch]] — resolution, retención y alarms
- También en: [[custom-metric]]

## Dato de examen

- **Detailed monitoring de EC2 ≠ high resolution**: detailed es 1 minuto; bajar de ahí requiere una custom metric.
- "Alarma que reaccione en segundos" → high-resolution metric + high-resolution alarm.

## Ver también

[[custom-metric]] · [[dimension]]
