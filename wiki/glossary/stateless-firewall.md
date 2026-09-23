---
title: Stateless firewall
category: glossary
tags: [networking, firewall, nacl]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.05 Stateful vs Stateless Firewalls.md", "raw/doc oficial/Control subnet traffic with network access control lists - Amazon Virtual Private Cloud.md"]
updated: 2026-09-23
---

# Stateless firewall

> **En una línea:** un firewall que **no recuerda conexiones**: evalúa cada paquete por separado, así que request y response necesitan cada uno su regla.

## Definición

Ve el request y la response como tráfico independiente. Cada conexión necesita **dos reglas** (una en cada dirección), y la de la response tiene que abrir el rango de [[ephemeral-port|ephemeral ports]] del otro lado. Más flexible en teoría, más difícil de administrar.

## Dónde aparece

- [[security-groups-vs-nacls]] — las **NACLs** son stateless
- [[VPC]] — sección Security Groups y NACLs
- También en: [[vpc-cheat-sheet]] · [[vpc-flow-logs]]

## Dato de examen

- **NACL = stateless.** Si el tráfico entra pero la respuesta no vuelve, falta la regla en la **dirección contraria** hacia los ephemeral ports.

## Ver también

[[stateful-firewall]] · [[ephemeral-port]]
