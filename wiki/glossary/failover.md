---
title: Failover
category: glossary
tags: [route53, ha, resiliencia, dns]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.13 HA vs. FT vs. DR.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.15 DNS Record Types.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.08 Network Address Translation (NAT) y NAT Gateway.md"]
updated: 2026-09-24
---

# Failover

> **En una línea:** el cambio automático hacia un recurso de respaldo cuando el primario deja de responder.

## Definición

Mecanismo (manual o automático) que redirige tráfico, o resolución DNS, desde un recurso que falló hacia uno sano. En Route 53 se implementa con una failover routing policy + health checks; en RDS [[multi-az|Multi-AZ]] es automático (60–120 s); en DR es la operación central de cualquier estrategia (Pilot Light, Warm Standby, Active/Active).

## Dónde aparece

- [[Route53]] — failover routing policy, MX failover con prioridades escalonadas, [[ttl|TTL]] bajo (~60 s) en registros de failover de DR
- [[ha-ft-dr]] — RDS Multi-AZ failover; "failover automático" es palabra clave de **HA**
- [[availability]], [[region-resilient]], [[globally-resilient]] — un escenario de failover DNS multi-region apunta a Route 53 por ser globally resilient
- También en: [[nat-gateway-vs-nat-instance]]

## Dato de examen

- "Minimizar downtime, recuperación rápida, failover automático" → palabra clave de **HA**, no de **FT** (que espera cero downtime, no un failover).
- Antes de un failover DNS planeado: bajar el TTL con anticipación (días antes) para que el cache viejo expire rápido.

## Ver también

[[ha-ft-dr]] · [[ttl]] · [[region-resilient]] · [[globally-resilient]]
