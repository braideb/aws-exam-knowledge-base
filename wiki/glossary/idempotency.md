---
title: Idempotency (idempotencia)
category: glossary
tags: [arquitectura, event-driven, serverless, sqs, lambda]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Amazon S3 Event Notifications - Amazon Simple Storage Service.md", "raw/doc oficial/Amazon RDS and Aurora High Availability Guide - Multi-AZ, Read Replicas, RDS Proxy, and Global Database Failover.md"]
updated: 2026-10-03
---

# Idempotency (idempotencia)

> **En una línea:** procesar el mismo mensaje **dos veces** produce el mismo resultado que procesarlo una.

## Definición

Propiedad de una operación que puede repetirse sin efectos secundarios acumulativos. Es un **requisito de diseño**, no una feature de AWS: casi toda la mensajería de AWS entrega **at least once**, así que los duplicados **van a pasar**.

Cómo se logra: deduplicar por un ID único del evento, usar escrituras condicionales, o hacer que la operación sea naturalmente repetible (`SET x = 5` en vez de `x = x + 1`).

## Dónde aparece

- [[S3]] — Event Notifications: entrega **at least once** → "diseñar consumidores idempotentes"
- [[EventBridge]] — reintentos y DLQ
- [[dva-development]] — arquitecturas event-driven, pilar del DVA-C02
- También en: [[horizontal-vs-vertical-scaling]] · [[eventual-consistency]] · [[serverless]] · [[stateless]] · [[RDS]]

## Dato de examen

- Enunciado *"el evento se procesó dos veces y se duplicó el cobro"* → la respuesta correcta es **hacer idempotente al consumidor**, no "configurar SQS FIFO" (que además solo garantiza exactly-once dentro de su ventana de deduplicación).
- Los eventos de S3 **normalmente llegan en segundos pero pueden demorar más de un minuto** → tampoco asumir orden.
- En un failover de base de datos, una escritura que dio timeout **puede o no haberse confirmado**. Para reintentarla sin duplicar, tiene que ser idempotente (o llevar una clave de deduplicación) ([[RDS]]).
- Relacionado en S3: el **conditional write** `If-None-Match` evita sobrescribir en escrituras concurrentes (donde por defecto rige **last-writer-wins**).

## Ver también

[[eventual-consistency]] · [[serverless]]
