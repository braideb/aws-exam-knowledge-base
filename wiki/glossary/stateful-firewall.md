---
title: Stateful firewall
category: glossary
tags: [networking, firewall, security-groups]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.05 Stateful vs Stateless Firewalls.md", "raw/doc oficial/Control traffic to your AWS resources using security groups - Amazon Virtual Private Cloud.md"]
updated: 2026-09-24
---

# Stateful firewall

> **En una línea:** un firewall que **recuerda las conexiones** y deja pasar automáticamente la respuesta de un request permitido.

## Definición

Identifica que request y response pertenecen a la misma conexión. Si permite el request (entrante o saliente), la response se permite sola, sin importar las reglas en la otra dirección. Resultado: **una regla por conexión** y menos trabajo de administración.

## Dónde aparece

- [[security-groups-vs-nacls]] — los **Security Groups** son stateful
- [[VPC]] — sección Security Groups y NACLs
- También en: [[vpc-cheat-sheet]] · [[vpc-flow-logs]] · [[ephemeral-port]] · [[implicit-deny]] · [[stateless-firewall]]

## Dato de examen

- **Security Group = stateful.** "¿Hace falta una regla outbound para que vuelva la respuesta a un inbound permitido?" → **no**.

## Ver también

[[stateless-firewall]] · [[ephemeral-port]] · [[implicit-deny]]
