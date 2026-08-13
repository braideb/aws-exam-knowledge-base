---
title: CIDR
category: glossary
tags: [networking, vpc, subnets, ip]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# CIDR — Classless Inter-Domain Routing

> **En una línea:** la notación `10.0.0.0/16` que define **un rango de IPs**.

## Definición

`dirección/prefijo`: el número tras la barra indica cuántos bits quedan fijos. Cuanto **mayor** el número, **menor** el rango.

| CIDR | IPs |
|---|---|
| `/16` | 65.536 |
| `/20` | 4.096 |
| `/24` | 256 |
| `/28` | 16 |

Rangos privados (RFC 1918): `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.

## Dónde aparece

- [[VPC]] — reglas de CIDR, Default VPC (`172.31.0.0/16`), IPs reservadas por subnet

## Dato de examen

- El CIDR de una [[VPC]] va de **`/16` a `/28`** — nada más grande que `/16`.
- El CIDR primario **no se puede achicar ni cambiar** después de crear la VPC; sí se pueden **agregar hasta 5 CIDRs secundarios**.
- **AWS reserva 5 IPs por subnet** (las primeras 4 y la última) → una `/24` tiene 256 direcciones pero **251 usables**, no 254. Por eso los cálculos de "cuántas instancias entran" nunca dan redondo.
- Regla de oro: **nunca el mismo CIDR en dos VPCs que algún día podrían hablarse** (rompe peering/VPN). La Default VPC usa el mismo CIDR en **todas** las cuentas y regiones → conflicto asegurado.

## Ver también

[[az-resilient]] · [[data-sovereignty]]
