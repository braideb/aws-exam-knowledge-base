---
title: Availability (disponibilidad)
category: glossary
tags: [resiliencia, sla, high-availability, s3]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-09-23
---

# Availability (disponibilidad)

> **En una línea:** el porcentaje del tiempo en que el sistema **responde**.

## Definición

Proporción de tiempo operativo. Se expresa en "nueves" y se traduce directo a downtime tolerado:

| Disponibilidad | Downtime/año | Por mes |
|---|---|---|
| 99% | ~3,65 días | ~7,2 h |
| 99.9% | ~8,77 h | ~43 min |
| 99.99% | ~52,6 min | ~4,3 min |
| 99.999% | ~5,26 min | ~26 s |

No confundir con [[durability|durabilidad]]: un dato puede estar perfectamente a salvo y aun así ser inalcanzable.

## Dónde aparece

- [[ha-ft-dr]] — tabla de nueves y cómo se acumula en serie/paralelo
- [[s3-storage-classes]] — disponibilidad diseñada por clase (Standard 99.99% · IA 99.9% · One Zone-IA 99.5%)
- [[Route53]] — el único servicio AWS con **[[sla|SLA]] del 100%**
- También en: [[S3]]

## Dato de examen

- En **serie** se multiplican las disponibilidades (cada capa baja el total); en **paralelo** se multiplican las *fallas* (la redundancia la sube). Ver [[single-point-of-failure]].
- "Maximizar uptime / [[failover]] automático" → **HA**; "el usuario no debe notar nada" → **FT** ([[ha-ft-dr]]).

## Ver también

[[durability]] · [[single-point-of-failure]] · [[ha-ft-dr]]
