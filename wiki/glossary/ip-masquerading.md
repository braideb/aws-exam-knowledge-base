---
title: IP masquerading
category: glossary
tags: [networking, nat, vpc]
exam: [SAA-C03, DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.08 Network Address Translation (NAT) y NAT Gateway.md", "raw/doc oficial/Connect to the internet or other networks using NAT devices - Amazon Virtual Private Cloud.md"]
updated: 2026-09-24
---

# IP masquerading

> **En una línea:** esconder **muchas IPs privadas detrás de una sola IP pública** — lo que normalmente se llama "NAT".

## Definición

Es un tipo de NAT (en rigor NAT + PAT, traducción de puertos). Permite que un rango privado inicie conexiones salientes y reciba sus respuestas, pero **nadie de afuera puede iniciar una conexión hacia adentro**, porque la IP pública es compartida. Se contrapone al **NAT estático** (1 privada ↔ 1 pública), que es lo que hace el Internet Gateway.

| | NAT estático | IP masquerading |
|---|---|---|
| Relación | 1:1 | N:1 |
| Entrante desde internet | ✅ | ❌ |
| En AWS | Internet Gateway | NAT Gateway / NAT instance |

## Dónde aparece

- [[VPC]] — sección NAT y NAT Gateway
- [[nat-gateway-vs-nat-instance]]
- También en: [[egress-only-internet-gateway]] · [[elastic-ip]]

## Dato de examen

- "Salida a internet para actualizar software, **sin** permitir conexiones entrantes" → NAT (masquerading) en IPv4, **Egress-Only IGW** en IPv6.

## Ver también

[[elastic-ip]] · [[egress-only-internet-gateway]]
