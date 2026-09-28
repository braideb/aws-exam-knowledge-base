---
title: Hypervisor
category: glossary
tags: [iaas, ec2, shared-responsibility, virtualizacion]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.01 Virtualization 101.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.12 Modelo de responsabilidad compartida (Shared Responsibility Model).md"]
updated: 2026-09-28
---

# Hypervisor

> **En una línea:** la capa de software que virtualiza el hardware físico — el punto exacto donde AWS deja de gestionar y empezás a gestionar vos en EC2.

## Definición

El hypervisor es la capa que particiona un servidor físico en instancias virtuales aisladas. En el modelo de responsabilidad compartida de [[iaas|IaaS]], es la línea de corte: AWS gestiona todo lo que está en el hypervisor y por debajo (hardware, red física, instalaciones); el cliente gestiona todo lo que corre arriba (sistema operativo, runtime, aplicación, datos).

## Dónde aparece

- [[EC2]] — "el IaaS clásico de AWS... AWS gestiona del hypervisor para abajo"
- [[iaas]] — "el corte está justo en el hypervisor"
- [[shared-responsibility-model]] — tabla de capas On-Prem / IaaS / [[paas|PaaS]] / [[saas|SaaS]]
- [[virtualization]] — cómo se llegó de la binary translation a SR-IOV
- [[nitro]] — el hypervisor propio de AWS
- También en: [[dedicated-tenancy]] · [[enhanced-networking]] · [[containers]]

## Dato de examen

- "¿Quién parchea el sistema operativo de una instancia EC2?" → el cliente, no AWS — el corte de IaaS está justo encima del hypervisor.

## Ver también

[[iaas]] · [[shared-responsibility-model]]
