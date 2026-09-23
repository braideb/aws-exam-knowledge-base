---
title: EventBridge
category: service
tags: [eventbridge, eventos, event-driven, automatizacion, cloudwatch-events]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.11 CloudWatch — Basics.md", "raw/doc oficial/Using EventBridge - Amazon Simple Storage Service.md"]
updated: 2026-09-19
---

# EventBridge

## ¿Qué es?

El **hub de eventos** de AWS — la evolución de CloudWatch Events (mismo backend, API compatible). Recibe eventos de servicios AWS y los rutea a destinos según **reglas**; también genera eventos **programados** (cron).

## Casos de uso

- Reaccionar a cambios de estado ("una instancia se detuvo" → Lambda).
- Arquitecturas **event-driven** (pilar de DVA-C02) — con consumidores [[idempotency|idempotentes]].
- Tareas programadas sin servidores (cron [[serverless|serverless]]).

## Características clave

- **Default para todo lo nuevo** frente a S3 Event Notifications y CloudWatch Events clásico.
- Con S3: habilitando EventBridge en el bucket, **todos** los tipos de evento van al bus (Object Created/Deleted, Storage Class Changed, Tags Added/Deleted, ACL Updated, Restore Expired…) y se filtran con **reglas ricas** (prefijo, sufijo, tamaño, metadata…). Ojo: la mayoría de esos tipos también existen en S3 Events clásico — la ventaja de EventBridge no es la lista de eventos sino el **filtrado, la fiabilidad y los destinos**.
- Más destinos que S3 Events (que solo tiene SNS/SQS/Lambda): Step Functions, Kinesis, SQS **FIFO**, otra cuenta, etc., con reintentos y DLQ.

**S3 Event Notifications vs. EventBridge** — las cinco dimensiones que se preguntan al compararlos:

| | **S3 Event Notifications** | **EventBridge** |
|---|---|---|
| Destinos | Solo SNS, SQS, Lambda | **15+** servicios (Step Functions, Kinesis…) |
| Tipos de evento | Conjunto acotado | **Todos** los de S3, más granulares |
| Filtrado | Por prefijo y sufijo de la key | **Avanzado**, por cualquier campo (tamaño, tipo, metadata) |
| Múltiples destinos | Limitado | Muchos para el mismo evento, con reglas independientes |
| Activación | Ya está listo | Hay que **activar EventBridge en el bucket** primero |

> Regla práctica: para el patrón simple de siempre (subida → Lambda), las event notifications clásicas alcanzan y son más directas. Para filtrado fino, múltiples consumidores o destinos que no sean los tres clásicos → **EventBridge**. En diseño nuevo, AWS lo empuja por defecto.

## Integración con otros servicios

- [[S3]] — fuente de eventos de objetos (alternativa moderna a Event Notifications).
- [[CloudWatch]] — hermano de monitoreo; EventBridge se encarga de *reaccionar*.
- [[CloudTrail]] — para eventos de API que no emiten evento nativo, EventBridge puede matchear sobre lo que CloudTrail captura.

## Gotchas y trampas del examen

- "Detectar y reaccionar **en tiempo casi real** a un cambio" → **EventBridge** (no [[CloudTrail]], que demora ~15 min).
- "SQS FIFO como destino de eventos S3" → S3 Events no lo soporta; **EventBridge sí** rutea a FIFO.
- CloudWatch Events y EventBridge son el mismo servicio; el examen usa ambos nombres.

> ⚠️ Página inicial — se ampliará cuando el curso cubra EventBridge en profundidad (buses custom, schema registry, pipes).
