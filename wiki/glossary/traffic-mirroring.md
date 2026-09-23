---
title: Traffic Mirroring
category: glossary
tags: [vpc, networking, observabilidad, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.10 VPC Flow Logs.md"]
updated: 2026-09-23
---

# Traffic Mirroring

> **En una línea:** copiar el **contenido completo de los paquetes** de una [[eni|ENI]] hacia una herramienta de análisis.

## Definición

Duplica el tráfico de red —headers y payload— y lo manda a un destino (otra ENI o un Network Load Balancer) donde corre un IDS, un sniffer o una herramienta forense. Es lo opuesto a los [[vpc-flow-logs]], que solo registran **metadatos**.

## Dónde aparece

- [[vpc-flow-logs]] — el contraste que el examen usa como distractor
- También en: [[VPC]] · [[dva-troubleshooting]] · [[vpc-cheat-sheet]]

## Dato de examen

- **Flow Logs = metadatos (quién habló con quién y si se aceptó). Traffic Mirroring = el contenido.** Si la pregunta pide "ver qué había dentro de los paquetes" o "inspección profunda", Flow Logs es la respuesta incorrecta.

## Ver también

[[vpc-flow-logs]] · [[eni]]
