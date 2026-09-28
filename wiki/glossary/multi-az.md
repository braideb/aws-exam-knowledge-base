---
title: Multi-AZ
category: glossary
tags: [resiliencia, rds, ha, availability-zones]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.13 HA vs. FT vs. DR.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.03 Availability Zones (AZ).md", "raw/doc oficial/Regional NAT gateways for automatic multi-AZ expansion - Amazon Virtual Private Cloud.md"]
updated: 2026-09-28
---

# Multi-AZ

> **En una línea:** un servicio replicado en más de una Availability Zone dentro de la misma region — sobrevive a la caída de una AZ, no a la caída de la region.

## Definición

Configuración donde un servicio (RDS, ELB, Auto Scaling) mantiene réplicas activas en al menos dos AZs de la misma region. Es el mecanismo concreto detrás de ser [[region-resilient|region resilient]]: mientras quede al menos una AZ sana, el servicio sigue funcionando — pero si cae la region entera, cae con ella.

## Dónde aparece

- [[ha-ft-dr]] — HA se logra con Multi-AZ + Auto Scaling + ELB; RDS Multi-AZ falla en 60–120 s
- [[global-infrastructure]] — tabla de niveles de resiliencia, RDS Multi-AZ como ejemplo de [[region-resilient]]
- [[region-resilient]]
- [[vpc-design]] — subnets por tier repetidas en 3 AZs (+1 de reserva)
- También en: [[failover]] · [[nat-gateway-vs-nat-instance]] · [[EKS]]

## Dato de examen

- "¿Sobrevive a la caída de una AZ?" → distribuir **Multi-AZ**.
- "¿Sobrevive a la caída de la region?" → Multi-AZ **no alcanza**, hace falta replicación cross-region explícita (S3 CRR, backups en otra region).

## Ver también

[[region-resilient]] · [[ha-ft-dr]] · [[az-resilient]]
