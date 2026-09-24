---
title: Single Point of Failure (SPOF)
category: glossary
tags: [resiliencia, high-availability, arquitectura]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.13 HA vs. FT vs. DR.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.04 VPC Routing e Internet Gateway.md"]
updated: 2026-09-24
---

# Single Point of Failure (SPOF)

> **En una línea:** el componente cuya falla derriba todo el sistema.

## Definición

Cualquier elemento sin redundancia del que dependa el resto. La redundancia **solo sirve si elimina el SPOF**: dos web servers detrás de un load balancer pero en **una sola AZ** siguen teniendo un SPOF — la AZ.

Cómo se acumula la disponibilidad ([[ha-ft-dr]]):
- **En serie** (todos deben funcionar) → las disponibilidades se **multiplican**: 99.9% × 99.9% × 99.9% = **99.7%**.
- **En paralelo** (basta uno) → se multiplican las **probabilidades de falla**: dos componentes al 99% dan **99.99%**.

## Dónde aparece

- [[ha-ft-dr]] — cálculo de disponibilidad en serie vs. paralelo
- [[global-infrastructure]] — dimensionamiento N+1 entre AZs
- [[VPC]] — un solo NAT Gateway zonal para varias AZs
- [[nat-gateway-vs-nat-instance]] — la NAT instance sin [[failover]] es un SPOF
- [[horizontal-vs-vertical-scaling]] — escalar vertical deja una sola instancia como SPOF
- También en: [[vpc-design]] · [[availability]] · [[az-resilient]] · [[blast-radius]]

## Dato de examen

- Cada capa en serie **baja** el total: agregar componentes encadenados empeora la disponibilidad aunque cada uno sea bueno.
- Buscar el SPOF es el método para responder "¿qué le falta a esta arquitectura?".

## Ver también

[[az-resilient]] · [[blast-radius]] · [[ha-ft-dr]]
