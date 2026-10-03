---
title: IaaS (Infrastructure as a Service)
category: glossary
tags: [modelos-de-servicio, iaas, shared-responsibility]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.05 Elastic Compute Cloud (EC2) — Basics.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.12 Modelo de responsabilidad compartida (Shared Responsibility Model).md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.01 Virtualization 101.md"]
updated: 2026-10-03
---

# IaaS — Infrastructure as a Service

> **En una línea:** AWS te da la máquina virtual; **el SO para arriba es tuyo**.

## Definición

Modelo en el que el proveedor gestiona instalaciones, hardware, red y [[hypervisor]], y vos gestionás sistema operativo, runtime, aplicación y datos. El corte está justo en el hypervisor.

El IaaS clásico de AWS es [[EC2]].

## Dónde aparece

- [[shared-responsibility-model]] — tabla On-Prem / IaaS / [[paas|PaaS]] / [[saas|SaaS]]
- [[EC2]] — "vos gestionás SO y aplicaciones; AWS del hypervisor para abajo"
- [[virtualization]] — la línea que separa lo que controlás de lo que controla AWS
- También en: [[ec2-cheat-sheet]] · [[hypervisor]] · [[serverless]] · [[paas]]

## Dato de examen

- Parchear el SO de una EC2 → **tuyo**. Seguridad física del datacenter → **AWS**. Security Group → **siempre tuyo**.
- El método para responder preguntas de responsabilidad: identificar el modelo (IaaS/PaaS/SaaS) y ubicar la capa en la tabla.

## Ver también

[[paas]] · [[saas]] · [[serverless]] · [[shared-responsibility-model]]
