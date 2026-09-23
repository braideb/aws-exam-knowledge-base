---
title: Enhanced Networking (SR-IOV)
category: glossary
tags: [ec2, networking, performance, virtualizacion]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.01 Virtualization 101.md"]
updated: 2026-09-22
---

# Enhanced Networking (SR-IOV)

> **En una línea:** la tarjeta de red física se presenta como **varias mini-tarjetas reales**, una por instancia, así el [[hypervisor]] no tiene que traducir nada.

## Definición

El nombre genérico de la técnica es **SR-IOV** (*Single Root I/O Virtualization*); en EC2 se llama Enhanced Networking. Sin ella, cada paquete pasa por una capa de software del host, lo que consume CPU y agrega latencia. Con SR-IOV el guest habla **directo** con su porción de hardware.

## Dónde aparece

- [[virtualization]] — el último escalón de la evolución
- [[ec2-instance-types]] — la `n` de un tipo como `R5dn` indica networking mejorado

## Dato de examen

Lo que aporta: **más ancho de banda**, **latencia más baja y más constante** bajo carga, y **menos CPU del host** consumida en I/O. Si el escenario pide "latencia de red baja y predecible" en EC2, apunta acá (o a un placement group).

## Ver también

[[virtualization]] · [[nitro]] · [[hypervisor]]
