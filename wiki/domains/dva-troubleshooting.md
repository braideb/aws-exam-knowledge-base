---
title: "DVA-C02 · Dominio 4: Troubleshooting and Optimization"
category: domain
tags: [dva-c02, troubleshooting, optimization, monitoring, logging]
exam: [DVA-C02]
sources: ["https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html"]
updated: 2026-09-22
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
| Fallas de conectividad: route tables, security groups, NACLs ([[ephemeral-port\|ephemeral ports]]), NAT, DNS de la VPC | [[VPC]], [[security-groups-vs-nacls]] | ✅ |
| **Diagnosticar tráfico rechazado con evidencia** (`action = REJECT`) | [[vpc-flow-logs]] | ✅ fuerte |
| Instancia que no responde: System vs Instance status check, auto-recovery | [[EC2]] | ✅ |
| Volumen que "se puso lento": [[burst-credit\|créditos]] agotados de gp2 | [[ebs-volume-types]] | ✅ |
| Volumen restaurado que rinde poco al principio | [[lazy-restore]], [[EBS]] | ✅ |
| IOPS aprovisionadas que no se alcanzan (tope por instancia) | [[ebs-optimized]], [[instance-store-vs-ebs]] | ✅ |
| Lambda en VPC que falla con `ENILimitReached` o pierde internet | [[lambda-in-vpc]] | ✅ |
| Optimización de costo de cómputo | [[ec2-purchase-options]] | ✅ |

## Servicios más importantes para este dominio

[[CloudWatch]], [[CloudWatchLogs]], [[CloudTrail]], [[EventBridge]] — todos con página. Se suman [[EC2]] y [[EBS]] como objeto de diagnóstico.

> **El razonamiento de troubleshooting que más rinde:** los [[vpc-flow-logs|flow logs]] dicen *que* se bloqueó, no *quién*. Si se ve el **request ACCEPT y la respuesta REJECT**, corta la **NACL** (le falta la regla outbound de ephemeral ports); si se ve **solo el request REJECT**, puede ser el SG o la NACL de entrada. Y para ver el **contenido** de los paquetes no sirven los flow logs: hace falta [[traffic-mirroring|Traffic Mirroring]].

> ⚠️ **Huecos pendientes de ingest**: **X-Ray** (tracing distribuido — el gran ausente y muy preguntado en DVA), CloudWatch **Logs Insights**, ServiceLens, análisis de errores de Lambda/API Gateway ([[throttling]], cold starts, códigos 4xx/5xx).
