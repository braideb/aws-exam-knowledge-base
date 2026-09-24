---
title: Globally Resilient
category: glossary
tags: [resiliencia, infraestructura, global]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.03 Availability Zones (AZ).md", "raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.03 IAM — Conceptos básicos.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.14 Route 53 (R53) — Fundamentos.md"]
updated: 2026-09-24
---

# Globally Resilient

> **En una línea:** servicio que sigue funcionando aunque se caiga una **region entera**.

## Definición

Nivel máximo de resiliencia de un servicio AWS: sus datos y su plano de control están replicados **entre varias regiones**, así que la pérdida total de una region no lo afecta. Coincide casi siempre con los servicios **globales** (no elegís region al usarlos).

Ejemplos: [[IAM]], [[Route53]], [[CloudFront]], STS, Organizations.

## Dónde aparece

- [[global-infrastructure]] — tabla de los tres niveles de resiliencia
- [[IAM]] — "es global, globally resilient y gratis"
- [[Route53]] — única base de datos global, **[[sla|SLA]] del 100%**
- También en: [[az-resilient]] · [[edge-location]] · [[failover]] · [[region-resilient]]

## Dato de examen

- Lo global **no es gratis en auditoría**: [[CloudTrail]] registra los eventos de servicios globales (IAM, STS, CloudFront, Route 53) en **`us-east-1`**.
- Un escenario de **[[failover]] DNS multi-region** apunta a Route 53 justamente por ser globally resilient.

## Ver también

[[region-resilient]] · [[az-resilient]] · [[blast-radius]]
