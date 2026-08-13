---
title: Zone Apex
category: glossary
tags: [dns, route53, records, alias]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Zone Apex

> **En una línea:** el dominio **a secas** (`midominio.com`), sin subdominio delante.

## Definición

También llamado *naked domain* o *root domain*: el nivel superior de una hosted zone. Es donde viven obligatoriamente los records **SOA** y **NS** de la zona — y por eso el estándar DNS **prohíbe un CNAME** ahí (un CNAME bloquea cualquier otro record con el mismo nombre).

Ese conflicto es exactamente lo que resuelven los **ALIAS records** de [[Route53]].

## Dónde aparece

- [[Route53]] — tipos de record y ALIAS records
- [[alias-vs-cname]] — la comparación completa

## Dato de examen

- **"Apuntar `midominio.com` a un ELB / [[CloudFront]] / website de [[S3]]"** → **ALIAS**, nunca CNAME. Es una de las preguntas más repetidas.
- Un CNAME tampoco puede apuntar a una **IP** (eso es un A record) ni existir en el apex.
- Los ALIAS son **gratis** hacia recursos AWS y no permiten [[ttl]] custom.

## Ver también

[[ttl]] · [[alias-vs-cname]] · [[Route53]]
