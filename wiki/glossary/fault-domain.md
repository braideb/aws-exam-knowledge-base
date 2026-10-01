---
title: Fault Domain
category: glossary
tags: [resiliencia, ec2, placement-groups, arquitectura]
exam: [SAA-C03, DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/09 Advanced EC2/09.11 Partition Placement Groups.md"]
updated: 2026-09-30
---

# Fault Domain

> **En una línea:** un conjunto de recursos que **comparten una causa de falla** (energía, red, rack): si esa causa falla, caen todos juntos.

## Definición

Repartir componentes en fault domains distintos garantiza que una sola falla no los tire a todos. En EC2, cada **partición** de un partition placement group es un fault domain: tiene sus propios racks, con energía y red aisladas. A mayor escala, una **AZ** también funciona como fault domain.

## Dónde aparece

- [[placement-groups]]: partition placement groups (7 particiones por AZ)
- [[global-infrastructure]]: cada AZ es un dominio de falla independiente

## Dato de examen

- "Más de 7 instancias por AZ separadas en fault domains" → **partition** placement group, no spread.

## Ver también

[[blast-radius]] · [[topology-aware]] · [[single-point-of-failure]]
