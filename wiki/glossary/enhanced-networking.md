---
title: Enhanced Networking (SR-IOV)
category: glossary
tags: [ec2, networking, performance, virtualizacion]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.01 Virtualization 101.md", "raw/notas curso mejorado/09 Advanced EC2/09.12 Enhanced Networking (SR-IOV, ENA, EFA).md"]
updated: 2026-09-30
---

# Enhanced Networking (SR-IOV)

> **En una línea:** la tarjeta de red física se presenta como **varias mini-tarjetas reales**, una por instancia, así el [[hypervisor]] no tiene que traducir nada.

## Definición

El nombre genérico de la técnica es **SR-IOV** (*Single Root I/O Virtualization*); en EC2 se llama Enhanced Networking. Sin ella, cada paquete pasa por una capa de software del host, lo que consume CPU y agrega latencia. Con SR-IOV el guest habla **directo** con su porción de hardware. Viene **habilitado por defecto, o sin costo**, en la mayoría de los instance types modernos, y es **requisito** de los cluster [[placement-groups|placement groups]].

| Adaptador | Qué es | Hasta |
|---|---|---|
| **ENA** (Elastic Network Adapter) | El actual | **100 Gbps** |
| **Intel 82599 VF** | El viejo | 10 Gbps |
| **EFA** (Elastic Fabric Adapter) | Interfaz para [[hpc\|HPC]] con comunicación entre nodos de muy baja latencia (**MPI**) | — |

## Dónde aparece

- [[virtualization]] — el último escalón de la evolución
- [[ec2-instance-types]] — la `n` de un tipo como `R5dn` indica networking mejorado
- [[placement-groups]] — requisito del cluster placement group
- También en: [[nitro]] · [[EC2]] · [[ec2-cheat-sheet]] · [[hpc]] · [[dva-troubleshooting]]

## Dato de examen

Lo que aporta: **más ancho de banda**, **más PPS**, **latencia más baja y más constante** bajo carga, y **menos CPU del host** consumida en I/O. Si el escenario pide "latencia de red baja y predecible" en EC2, apunta acá (o a un placement group). Si menciona **MPI/HPC**, la respuesta es **EFA**.

## Ver también

[[virtualization]] · [[nitro]] · [[hypervisor]] · [[ebs-optimized]] · [[hpc]]
