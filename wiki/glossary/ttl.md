---
title: TTL (Time To Live)
category: glossary
tags: [dns, route53, cache, records]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.15 DNS Record Types.md", "raw/doc oficial/Choosing between alias and non-alias records - Amazon Route 53.md"]
updated: 2026-09-24
---

# TTL — Time To Live

> **En una línea:** cuántos segundos puede **cachearse** un record DNS antes de volver a consultarlo.

## Definición

Valor en segundos que acompaña a cada record. Mientras no expire, los resolvers responden desde su cache con una respuesta **non-authoritative**; solo al vencer vuelven a preguntarle al name server de la zona (respuesta **authoritative**).

Trade-off: TTL **alto** = menos consultas y más barato, pero los cambios tardan en verse. TTL **bajo** = cambios casi inmediatos, más consultas y más costo.

## Dónde aparece

- [[Route53]] — sección TTL y caching
- [[alias-vs-cname]] — los ALIAS **no permiten TTL configurable** (lo maneja AWS)
- También en: [[failover]] · [[zone-apex]]

## Dato de examen

- **Antes de migrar un dominio: bajar el TTL con días de anticipación** (para que expire el TTL viejo cacheado) y volver a subirlo tras estabilizar. Si el enunciado dice "cambié el record y algunos usuarios siguen yendo al servidor viejo", la causa es el TTL.
- TTL bajo (~60 s) también en registros de **[[failover]] de DR** ([[ha-ft-dr]]).

## Ver también

[[zone-apex]] · [[edge-location]] · [[eventual-consistency]]
