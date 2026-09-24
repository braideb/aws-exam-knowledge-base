---
title: Transit Gateway
category: glossary
tags: [vpc, networking, peering, hibrido]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.12 VPC Peering.md"]
updated: 2026-09-24
---

# Transit Gateway (TGW)

> **En una línea:** un **router regional central** al que se conectan VPCs, VPNs y Direct Connect, para no tener que unirlas de a pares.

## Definición

Funciona como un hub: cada VPC se *attachea* una sola vez al TGW y, a partir de ahí, puede alcanzar a todas las demás según lo que digan sus route tables. Resuelve el problema del [[vpc-peering]], que no es transitivo y crece de forma cuadrática.

## Dónde aparece

- [[vpc-peering]] — el límite que hace falta superar
- [[VPC]] — opciones de conectividad
- También en: [[vpc-cheat-sheet]] · [[edge-to-edge-routing]]

## Dato de examen

- La regla mental es de cantidad: **pocas VPCs → peering; muchas VPCs → Transit Gateway**. Conectar N VPCs todas contra todas por peering cuesta **N × (N-1) / 2** conexiones (10 VPCs = 45).
- A diferencia del peering, el TGW **sí** permite ruteo transitivo entre los attachments.

## Ver también

[[vpc-peering]] · [[edge-to-edge-routing]]
