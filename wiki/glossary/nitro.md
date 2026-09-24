---
title: Nitro
category: glossary
tags: [ec2, virtualizacion, hypervisor]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.01 Virtualization 101.md"]
updated: 2026-09-24
---

# Nitro

> **En una línea:** la plataforma de virtualización propia de AWS — el [[hypervisor]] y las tarjetas dedicadas sobre las que corren las instancias EC2 modernas.

## Definición

Nitro mueve la red, el almacenamiento y la seguridad a **hardware dedicado**, dejando casi todos los recursos del host para las instancias. Es lo que hace posible que una instancia rinda prácticamente como si corriera sobre metal desnudo, y la base técnica de cosas como el cifrado de [[EBS]] sin penalidad de rendimiento o las ENIs de alto desempeño.

## Dónde aparece

- [[virtualization]] — el cierre de la evolución de la virtualización
- [[EC2]] — por qué el overhead es mínimo
- También en: [[enhanced-networking]]

## Dato de examen

No suele preguntarse por sí mismo en el DVA, pero explica varios "¿por qué?": el cifrado de EBS **no tiene impacto de rendimiento** porque lo hace el host (Nitro), no el sistema operativo; y las generaciones más nuevas de instancias soportan límites de I/O mucho más altos.

## Ver también

[[hypervisor]] · [[virtualization]] · [[enhanced-networking]]
