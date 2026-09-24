---
title: Egress-Only Internet Gateway
category: glossary
tags: [vpc, ipv6, igw, routing]
exam: [SAA-C03, DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.08 Network Address Translation (NAT) y NAT Gateway.md", "raw/doc oficial/How Amazon VPC works - Amazon Virtual Private Cloud.md", "raw/doc oficial/Create a VPC - Amazon Virtual Private Cloud.md"]
updated: 2026-09-24
---

# Egress-Only Internet Gateway (EIGW)

> **En una línea:** un Internet Gateway **solo para IPv6 y solo saliente**: las instancias salen a internet, pero nadie puede entrar.

## Definición

Como las IPv6 de AWS son públicas, no hay NAT para IPv6. Si querés que una private subnet salga por IPv6 sin exponerse, ponés la ruta `::/0 → eigw-…`. Es el equivalente funcional de un NAT Gateway, pero para IPv6.

| Ruta `::/0` hacia | Resultado |
|---|---|
| Internet Gateway | Bidireccional |
| Egress-Only IGW | **Solo saliente** |

## Dónde aparece

- [[VPC]] — sección IPv6 y NAT
- [[nat-gateway-vs-nat-instance]] — para IPv6 no se usa NAT
- También en: [[vpc-cheat-sheet]] · [[ip-masquerading]]

## Dato de examen

- "Instancias IPv6 en una private subnet que necesitan updates de internet pero no deben ser alcanzables" → **Egress-Only IGW** (no NAT Gateway).

## Ver también

[[ip-masquerading]] · [[public-subnet]]
