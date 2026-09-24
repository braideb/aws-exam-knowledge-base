---
title: Longest prefix match
category: glossary
tags: [vpc, routing, route-tables, cidr]
exam: [SAA-C03, DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.04 VPC Routing e Internet Gateway.md", "raw/doc oficial/How route priority works - Amazon Virtual Private Cloud.md"]
updated: 2026-09-24
---

# Longest prefix match

> **En una línea:** cuando varias rutas matchean un destino, **gana la más específica** (el prefijo `/` más alto).

## Definición

Una route table puede tener rutas que se solapan (`10.16.0.0/16` y `0.0.0.0/0` cubren las dos a `10.16.32.10`). El router elige la de **prefijo más largo**: más bits fijos = rango más chico = más específica = más prioridad.

| Destino del paquete | Rutas que matchean | Gana |
|---|---|---|
| `10.16.32.10` | `10.16.0.0/16 → local`, `0.0.0.0/0 → igw` | `/16` → local |
| `172.31.5.5` | `172.31.0.0/16 → pcx`, `0.0.0.0/0 → igw` | `/16` → peering |
| `1.3.3.7` | solo `0.0.0.0/0 → igw` | IGW |

Si dos rutas tienen **el mismo** destino, la **estática** le gana a la **propagada** (VPN).

## Dónde aparece

- [[VPC]] — prioridad de rutas y local route
- [[vpc-endpoints]] — la ruta del gateway endpoint le gana a la default hacia el NAT
- [[vpc-peering]] — por qué los CIDRs no pueden solaparse: la ruta local siempre gana
- También en: [[nat-gateway-vs-nat-instance]] · [[vpc-cheat-sheet]] · [[prefix-list]] · [[public-subnet]]

## Dato de examen

- `0.0.0.0/0` es la ruta **menos** específica: solo se usa si nada más matchea.
- IPv4 e IPv6 se evalúan por separado.

## Ver también

[[cidr]] · [[public-subnet]]
