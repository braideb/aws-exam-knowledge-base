---
title: Host Affinity
category: glossary
tags: [ec2, dedicated-hosts, licencias]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.20 Dedicated Hosts y Dedicated Instances.md"]
updated: 2026-09-22
---

# Host Affinity

> **En una línea:** atar una instancia a **un host físico concreto**, para que un stop + start la devuelva al mismo hardware.

## Definición

Es una opción de los **Dedicated Hosts**. Rompe a propósito la regla general de EC2, donde un stop + start puede mudar la instancia a otro host. Existe casi exclusivamente por **licenciamiento**: las licencias que se cuentan por socket o por núcleo físico necesitan que la instancia no ande saltando de máquina.

## Dónde aparece

- [[ec2-purchase-options]] — Dedicated Hosts
- [[EC2]] — ciclo de vida de la instancia

## Dato de examen

Si la pregunta menciona **licencias por socket o por core** o "necesito ver los núcleos físicos" → **Dedicated Host** (con Host Affinity). Si solo dice "regulación: no podemos compartir hardware con otros clientes" → **Dedicated Instances**, que es más barato y no ofrece affinity.

## Ver también

[[dedicated-tenancy]] · [[ec2-purchase-options]]
