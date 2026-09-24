---
title: Dimension (CloudWatch)
category: glossary
tags: [cloudwatch, metrics, observabilidad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.11 CloudWatch — Basics.md", "raw/notas curso mejorado/07 Monitoring and logging/07.02 CloudWatch Data (Namespace, Datapoint, Metric, Dimensions).md"]
updated: 2026-09-24
---

# Dimension (CloudWatch)

> **En una línea:** el par nombre/valor que dice **de dónde vino** cada datapoint.

## Definición

Dentro de una métrica de [[CloudWatch]], las dimensions separan las fuentes: la métrica es `CPUUtilization`, y la dimension `InstanceId=i-0abc123` indica de qué instancia. Sin ellas no se podría distinguir la CPU de una instancia puntual del promedio de todas las `t3.small`.

La jerarquía completa:

```
Namespace (AWS/EC2)
└── Metric (CPUUtilization)      ← serie temporal
    └── Datapoint (timestamp+value)
        + Dimensions (InstanceId=i-xxx)
```

## Dónde aparece

- [[CloudWatch]] — jerarquía de conceptos; **máx 30 dimensions** por métrica
- [[CloudWatchLogs]] — dimensions extraídas por un [[metric-filter]]
- [[observability-costs]] — cada combinación única se factura aparte
- [[custom-metric]] — AWS no agrega custom metrics a través de dimensions
- También en: [[XRay]] · [[high-cardinality]] · [[high-resolution-metric]] · [[percentile]]

## Dato de examen

- Una métrica se identifica por **Namespace + MetricName + Dimensions**: cambiar una sola dimension es **otra métrica**. Al publicar con `PutMetricData`, mandar siempre las mismas dimensions.
- La **agregación** por dimensions (ej. `CPUUtilization` por `InstanceType` o `AutoScalingGroupName`) la ofrece AWS **solo en ciertas métricas nativas**, no en las [[custom-metric|custom]].
- Un **datapoint no es un servidor**: es una medición. La fuente se identifica con las dimensions — pregunta recurrente.
- Cada combinación única métrica + dimensions es **una métrica facturada** → cuidado con la [[high-cardinality|alta cardinalidad]].
- Las **request metrics** de [[S3]] (opt-in, pagas, 1 min) pueden filtrarse por access point usando dimensions.

## Ver también

[[high-cardinality]] · [[observability-costs]] · [[custom-metric]]
