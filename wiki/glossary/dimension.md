---
title: Dimension (CloudWatch)
category: glossary
tags: [cloudwatch, metrics, observabilidad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
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

## Dato de examen

- Un **datapoint no es un servidor**: es una medición. La fuente se identifica con las dimensions — pregunta recurrente.
- Cada combinación única métrica + dimensions es **una métrica facturada** → cuidado con la [[high-cardinality|alta cardinalidad]].
- Las **request metrics** de [[S3]] (opt-in, pagas, 1 min) pueden filtrarse por access point usando dimensions.

## Ver también

[[high-cardinality]] · [[observability-costs]]
