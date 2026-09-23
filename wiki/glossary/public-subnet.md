---
title: Public subnet
category: glossary
tags: [vpc, subnets, routing, igw]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.04 VPC Routing e Internet Gateway.md", "raw/doc oficial/Subnets for your VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/Enable internet access for a VPC using an internet gateway - Amazon Virtual Private Cloud.md"]
updated: 2026-09-23
---

# Public subnet

> **En una línea:** una subnet cuya **route table tiene una ruta directa a un Internet Gateway**.

## Definición

"Pública" no es un atributo que se tilda: lo define el **routing**. Una subnet es pública si su route table manda `0.0.0.0/0` (o `::/0`) al IGW. Para que un recurso dentro llegue a internet además necesita **IP pública/[[elastic-ip|Elastic IP]]** (o IPv6) y SGs/NACLs que lo permitan.

| Tipo | Route table |
|---|---|
| Public | Ruta al **IGW** |
| Private | Sin ruta al IGW (sale por NAT) |
| VPN-only | Ruta a una VPN (virtual private gateway) |
| Isolated | Sin rutas fuera de la VPC |

## Dónde aparece

- [[VPC]] — tipos de subnet y receta de public subnet
- [[nat-gateway-vs-nat-instance]] — el NAT GW zonal va en una public subnet
- [[lambda-in-vpc]] — poner la función en una subnet pública **no** le da internet
- También en: [[vpc-cheat-sheet]]

## Dato de examen

- Instancia con IP pública que **no** llega a internet → su subnet no tiene ruta al IGW (es privada).
- Las subnets de una Custom VPC **arrancan privadas**; las de la Default VPC son públicas.

## Ver también

[[bastion-host]] · [[longest-prefix-match]]
