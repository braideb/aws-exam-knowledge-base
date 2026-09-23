---
title: Edge-to-edge routing
category: glossary
tags: [vpc, peering, routing]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.12 VPC Peering.md"]
updated: 2026-09-23
---

# Edge-to-edge routing

> **En una línea:** usar la **salida de otra VPC** (su IGW, NAT, endpoint o VPN) a través de un peering — algo que AWS **no permite**.

## Definición

Un [[vpc-peering|peering]] conecta los *recursos* de dos VPCs, no sus puertas de salida. Desde la VPC A no podés rutear hacia el Internet Gateway, el NAT Gateway, los [[vpc-endpoints|VPC endpoints]] ni la VPN/Direct Connect de la VPC B. Si A necesita salir a internet, necesita su propia salida.

## Dónde aparece

- [[vpc-peering]] — dentro de "qué NO hace un peering"
- También en: [[VPC]] · [[vpc-cheat-sheet]]

## Dato de examen

- Distractor clásico: *"la VPC B ya tiene NAT Gateway, ¿puedo hacer peering y ahorrarme el de A?"* → **No.** No hay edge-to-edge routing. Para compartir una salida hace falta un [[transit-gateway|Transit Gateway]] con la topología adecuada.

## Ver también

[[vpc-peering]] · [[transit-gateway]]
