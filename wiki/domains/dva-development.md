---
title: "DVA-C02 · Dominio 1: Development with AWS Services"
category: domain
tags: [dva-c02, development, serverless, apis, sdk]
exam: [DVA-C02]
sources: ["https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html"]
updated: 2026-08-12
---

# DVA-C02 · Dominio 1 — Development with AWS Services

## Peso en el examen

**32%** — el dominio más pesado del examen.

## Resumen de lo que el examen evalúa

Desarrollar código para aplicaciones hospedadas en AWS: arquitecturas event-driven, uso de APIs/SDK/CLI, patrones [[serverless]] (Lambda), y uso de data stores desde el código (DynamoDB, S3, ElastiCache).

## Task statements oficiales (exam guide)

**Task 1 — Desarrollar código para apps en AWS**: patrones de arquitectura (event-driven, microservicios, fanout, coreografía/orquestación), stateful vs stateless, acoplamiento fuerte vs débil, sync vs async, crear/extender APIs (validación, transformaciones), unit tests (SAM), messaging, SDKs/APIs de AWS, streaming, EventBridge para event-driven, resiliencia ante terceros (retries, circuit breakers). Incluye **Amazon Q Developer** como asistente.

**Task 2 — Desarrollar código para Lambda**: acceso a recursos privados en VPC, configuración (memoria, concurrency, timeout, runtime, handler, layers, extensions, triggers, destinations), ciclo de vida de eventos y errores (Destinations, DLQ), testing, integraciones, tuning de performance, procesamiento near real-time.

**Task 3 — Usar data stores**: partition keys de alta cardinalidad, modelos de consistencia (strong vs eventual), query vs scan, keys e índices de DynamoDB, serialización, lifecycle de datos, caching, stores especializados (OpenSearch).

## Temas clave — cobertura actual

| Tema | Página | Estado |
|---|---|---|
| S3 desde el código: [[multipart-upload\|multipart]], [[presigned-url\|presigned URLs]], CORS | [[S3]] | ✅ |
| Arquitecturas event-driven con S3 Events / EventBridge | [[S3]], [[EventBridge]] | ✅ (EventBridge inicial) |
| Uso de CLI, perfiles y credenciales | [[aws-cli]], [[IAM]] | ✅ |
| Cifrado desde la app ([[envelope-encryption\|envelope encryption]], [[data-encryption-key\|DEKs]]) | [[KMS]], [[s3-encryption]] | ✅ |

## Servicios más importantes para este dominio

Presentes en la wiki: [[S3]], [[EventBridge]], [[KMS]].

> ⚠️ **Huecos grandes pendientes de ingest** (el curso aún no los cubrió): **Lambda**, **API Gateway**, **DynamoDB**, **SQS/SNS**, **Step Functions**, **ElastiCache**, SDK patterns (reintentos, backoff, paginación). Son el corazón de este dominio — prioridad alta para próximos ingest.
