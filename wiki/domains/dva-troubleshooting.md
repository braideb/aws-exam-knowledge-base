---
title: "DVA-C02 · Dominio 4: Troubleshooting and Optimization"
category: domain
tags: [dva-c02, troubleshooting, optimization, monitoring, logging]
exam: [DVA-C02]
sources: ["https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html"]
updated: 2026-10-03
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
| Resolution, retención, [[percentile\|percentiles]], alarms high resolution y "M out of N" | [[CloudWatch]] | ✅ fuerte |
| Métricas custom desde el código (`PutMetricData`, agent) | [[CloudWatch]], [[custom-metric]] | ✅ |
| **Tracing distribuido**: segments, service map, annotations vs metadata, daemon + IAM | [[XRay]] | ⚠️ solo curso, sin doc oficial |
| Sacar logs en vivo: [[subscription-filter\|subscription filters]], export a S3, agregación multi-cuenta | [[CloudWatchLogs]] | ✅ |
| Reaccionar a eventos (remediación) | [[EventBridge]] | ⚠️ inicial |
| Optimización de performance en S3 (multipart, TA) | [[S3]] | ✅ |
| Optimización de costos de storage (lifecycle, classes) | [[s3-storage-classes]] | ✅ |
| Optimización de costos de observabilidad (ingesta de logs, cardinalidad) | [[observability-costs]] | ✅ |
| Resiliencia y recuperación | [[ha-ft-dr]] | ✅ |
| Fallas de conectividad: route tables, security groups, NACLs ([[ephemeral-port\|ephemeral ports]]), NAT, DNS de la VPC | [[VPC]], [[security-groups-vs-nacls]] | ✅ |
| **Diagnosticar tráfico rechazado con evidencia** (`action = REJECT`) | [[vpc-flow-logs]] | ✅ fuerte |
| Instancia que no responde: System vs Instance status check, auto-recovery | [[EC2]] | ✅ |
| Instancia `running` con 2/2 checks pero mal configurada: **user data fallido** (EC2 no lo valida; solo corre en el primer launch) | [[ec2-bootstrapping]] | ✅ |
| Métricas de **memoria/disco** y logs del SO: CloudWatch Agent + rol + config en Parameter Store | [[CloudWatchLogs]], [[CloudWatch]] | ✅ |
| La CLI usa otras credenciales aunque la instancia tenga rol (keys en disco **pisan** al instance profile) | [[IAM]], [[aws-cli]] | ✅ |
| `AccessDenied` al leer un SecureString (falta `kms:Decrypt`) | [[SSMParameterStore]] | ✅ |
| Rendimiento de red entre instancias: [[enhanced-networking]], placement groups | [[placement-groups]] | ✅ |
| Volumen que "se puso lento": [[burst-credit\|créditos]] agotados de gp2 | [[ebs-volume-types]] | ✅ |
| Volumen restaurado que rinde poco al principio | [[lazy-restore]], [[EBS]] | ✅ |
| [[iops\|IOPS]] aprovisionadas que no se alcanzan (tope por instancia) | [[ebs-optimized]], [[instance-store-vs-ebs]] | ✅ |
| Lambda en VPC que falla con `ENILimitReached` o pierde internet | [[lambda-in-vpc]] | ✅ |
| Optimización de costo de cómputo | [[ec2-purchase-options]] | ✅ |
| **Task de ECS que no arranca**: no hace pull de la image o no manda logs (task execution role), falta capacidad en EC2 mode ([[capacity-provider]]), health checks que fallan con [[dynamic-port-mapping]] | [[ECS]], [[ecs-ec2-vs-fargate]] | ✅ solo curso |
| Tracing en containers: daemon de X-Ray como [[sidecar]] | [[XRay]], [[ECS]] | ✅ |
| Optimización de costo de containers: right-sizing de la task, Fargate Spot, `binpack` | [[ecs-ec2-vs-fargate]] | ✅ |
| **Base de datos**: `too many connections` desde Lambda (RDS Proxy), app que no vuelve tras un failover (cache de DNS), lecturas desactualizadas en una réplica ([[replication-lag]]), failover lento (lag o Aurora sin replicas) | [[RDS]], [[Aurora]], [[rds-ha-options]] | ✅ curso + doc oficial |
| Rotación de Secrets Manager que falla (la Lambda en la VPC sin endpoint ni NAT) | [[SecretsManager]], [[vpc-endpoints]] | ✅ |
| Optimización de costo de bases: Aurora Serverless v2 / auto-pause, I/O-Optimized vs Standard | [[Aurora]] | ✅ |

## Servicios más importantes para este dominio

[[CloudWatch]], [[CloudWatchLogs]], [[XRay]], [[CloudTrail]], [[EventBridge]] — todos con página. Se suman [[EC2]] y [[EBS]] como objeto de diagnóstico, y [[ECS]] para containers.

> **Las tres patas de la observabilidad:** métricas ([[CloudWatch]]: *cuánto*), logs ([[CloudWatchLogs]]: *qué pasó*) y trazas ([[XRay]]: *por dónde pasó la request y cuánto tardó cada tramo* — [[distributed-tracing]]). "Quién hizo qué" no es ninguna de las tres: es auditoría, [[CloudTrail]].

> **El razonamiento de troubleshooting que más rinde:** los [[vpc-flow-logs|flow logs]] dicen *que* se bloqueó, no *quién*. Si se ve el **request ACCEPT y la respuesta REJECT**, corta la **NACL** (le falta la regla outbound de ephemeral ports); si se ve **solo el request REJECT**, puede ser el SG o la NACL de entrada. Y para ver el **contenido** de los paquetes no sirven los flow logs: hace falta [[traffic-mirroring|Traffic Mirroring]].

> ⚠️ **Huecos pendientes de ingest**: clippings de doc oficial de **X-Ray** (la página sale solo del curso), CloudWatch **Logs Insights**, **Embedded Metric Format**, ServiceLens, análisis de errores de Lambda/API Gateway ([[throttling]], cold starts, códigos 4xx/5xx).
