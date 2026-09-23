---
title: Dedicated tenancy
category: glossary
tags: [vpc, ec2, tenancy, compliance, cost]
exam: [SAA-C03, DVA-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.02 Custom VPC.md", "raw/doc oficial/Create a VPC - Amazon Virtual Private Cloud.md"]
updated: 2026-09-23
---

# Dedicated tenancy

> **En una línea:** correr las instancias en **hardware dedicado a tu cuenta**, no compartido con otros clientes.

## Definición

La tenancy se elige al crear la VPC:

| Tenancy de la VPC | Efecto |
|---|---|
| `Default` | Hardware compartido; cada instancia puede pedir su propia tenancy al lanzarse |
| `Dedicated` | **Todas** las instancias de la VPC son Dedicated Instances — sin excepción |

Dedicated es **bastante más caro**. AWS Outposts exige `Default`.

## Dónde aparece

- [[VPC]] — características de la Custom VPC
- [[ec2-purchase-options]] — Dedicated Hosts vs Dedicated Instances
- [[host-affinity]] — atar una instancia a un host físico concreto
- También en: [[EC2]]

## Dato de examen

- Requisito de compliance "hardware no compartido" → dedicated. Pero ojo: poner la **VPC** en `Dedicated` obliga a que todo lo que se lance ahí lo sea; si solo algunas instancias lo necesitan, VPC `Default` + tenancy por instancia.

## Ver también

[[hypervisor]] · [[shared-responsibility-model]]
