---
title: SLA (Service Level Agreement)
category: glossary
tags: [disponibilidad, route53, s3, resiliencia]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-08-12
---

# SLA (Service Level Agreement)

> **En una línea:** el compromiso contractual de AWS sobre disponibilidad o tiempos de un servicio — a veces viene incluido (Route 53), a veces se puede comprar (S3 RTC).

## Definición

El SLA es el nivel de servicio que AWS garantiza contractualmente, típicamente expresado en % de disponibilidad o en un tiempo máximo para completar una operación, con crédito de servicio si no se cumple. No es lo mismo que la [[availability|disponibilidad]] "de diseño" de un servicio: el SLA es el compromiso legal sobre esa disponibilidad.

## Dónde aparece

- [[Route53]] — el único servicio AWS con **SLA del 100%**, consecuencia directa de ser [[globally-resilient]]
- [[S3]] — Replication Time Control (RTC): SLA de **15 minutos** para que la réplica ocurra, pagando extra sobre la replicación estándar
- [[availability]], [[globally-resilient]]

## Dato de examen

- "¿Cuál es el único servicio AWS con SLA del 100%?" → **Route 53**.
- "Necesito garantía de que la replicación de S3 ocurra dentro de X minutos" → **RTC**, no CRR/SRR estándar (que es mejor esfuerzo, sin garantía de tiempo).

## Ver también

[[availability]] · [[Route53]] · [[globally-resilient]]
