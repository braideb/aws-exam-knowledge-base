---
title: "DVA-C02 · Dominio 4: Troubleshooting and Optimization"
category: domain
tags: [dva-c02, troubleshooting, optimization, monitoring, logging]
exam: [DVA-C02]
sources: ["https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html"]
updated: 2026-08-12
---

# DVA-C02 · Dominio 4 — Troubleshooting and Optimization

## Peso en el examen

**18%**

## Resumen de lo que el examen evalúa

Diagnosticar y resolver problemas de aplicaciones: observabilidad (métricas, logs, trazas), análisis de fallos, y optimización de rendimiento y costos.

## Task statements oficiales (exam guide)

**Task 1 — Root cause analysis**: debuggear código, interpretar métricas/logs/**traces**, consultar logs, métricas custom (**CloudWatch Embedded Metric Format**), dashboards e insights, troubleshooting de fallos de deployment, debug de integraciones entre servicios.

**Task 2 — Instrumentar código para observabilidad**: logging vs monitoring vs **observability**, estrategia de logging efectiva, emitir métricas custom desde el código, **annotations para tracing** (X-Ray), alertas por acciones específicas, structured logging, health checks y readiness probes.

**Task 3 — Optimizar aplicaciones**: definir **concurrency**, profiling, memoria/cómputo mínimos, subscription filter policies para messaging, cache por request headers, **caching a nivel de aplicación**, optimizar uso de recursos, identificar cuellos de botella desde los logs.

## Temas clave — cobertura actual

| Tema | Página | Estado |
|---|---|---|
| Métricas, dimensions, alarms | [[CloudWatch]] | ✅ fuerte |
| Logs centralizados, [[metric-filter\|metric filters]] | [[CloudWatchLogs]] | ✅ fuerte |
| Auditoría de API, quién-hizo-qué | [[CloudTrail]] | ✅ fuerte |
| Reaccionar a eventos (remediación) | [[EventBridge]] | ⚠️ inicial |
| Optimización de performance en S3 (multipart, TA) | [[S3]] | ✅ |
| Optimización de costos de storage (lifecycle, classes) | [[s3-storage-classes]] | ✅ |
| Optimización de costos de observabilidad (ingesta de logs, cardinalidad) | [[observability-costs]] | ✅ |
| Resiliencia y recuperación | [[ha-ft-dr]] | ✅ |

## Servicios más importantes para este dominio

[[CloudWatch]], [[CloudWatchLogs]], [[CloudTrail]], [[EventBridge]] — todos con página.

> ⚠️ **Huecos pendientes de ingest**: **X-Ray** (tracing distribuido — el gran ausente y muy preguntado en DVA), CloudWatch **Logs Insights**, ServiceLens, análisis de errores de Lambda/API Gateway ([[throttling]], cold starts, códigos 4xx/5xx).
