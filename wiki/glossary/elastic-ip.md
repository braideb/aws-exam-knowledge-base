---
title: Elastic IP
category: glossary
tags: [vpc, ec2, networking, ipv4, nat-gateway]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.08 Network Address Translation (NAT) y NAT Gateway.md", "raw/doc oficial/What is Amazon VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/IP addressing for your VPCs and subnets - Amazon Virtual Private Cloud.md"]
updated: 2026-09-23
---

# Elastic IP (EIP)

> **En una línea:** una **IPv4 pública estática** asignada a tu cuenta en una region, que no cambia hasta que la liberes.

## Definición

A diferencia de la IP pública "normal" de una instancia (que sale de un pool de Amazon y se pierde al detenerla), la Elastic IP es **tuya**: se asocia y desasocia a instancias, [[eni|ENIs]] o recursos como un NAT Gateway, y se puede mover de un recurso a otro.

| | IP pública auto-asignada | Elastic IP |
|---|---|---|
| Dueño | Pool de Amazon | Tu cuenta (en una region) |
| Stop/start de la instancia | Cambia | Se mantiene |
| Costo | Se cobra (toda IPv4 pública) | Se cobra (asociada o no) |

## Dónde aparece

- [[VPC]] — IPs públicas IPv4; el NAT Gateway público necesita una EIP al crearse
- [[nat-gateway-vs-nat-instance]] — IP pública de cada opción
- [[EC2]] — asociarla a la ENI primaria borra la IP pública dinámica; desasociarla asigna una nueva
- También en: [[observability-costs]] · [[vpc-cheat-sheet]]

## Dato de examen

- "Una IP pública que no cambie aunque se reinicie/reemplace la instancia" → **Elastic IP**.
- El NAT GW **no** es el único que usa EIPs (dato erróneo de las notas originales).
- Todas las IPv4 públicas se cobran, incluidas las EIPs.

## Ver también

[[eni]] · [[ip-masquerading]]
