---
title: AWS PrivateLink
category: glossary
tags: [vpc, networking, endpoints, privatelink]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.09 VPC Endpoints.md"]
updated: 2026-09-23
---

# AWS PrivateLink

> **En una línea:** la tecnología que expone un servicio **dentro de tu VPC** como una [[eni|ENI]] con IP privada, sin que el tráfico salga a internet.

## Definición

Es lo que hay por debajo de los **interface endpoints**: en vez de rutear hacia el servicio, AWS te coloca una interfaz de red con una IP de tu subnet, y el servicio queda accesible como si viviera en tu red. Como es una ENI, se protege con **security groups**. Sirve tanto para servicios de AWS (SQS, SNS, API Gateway…) como para servicios de terceros o propios publicados por otra cuenta.

## Dónde aparece

- [[vpc-endpoints]] — los interface endpoints están basados en PrivateLink
- [[gateway-vs-interface-endpoint]] — la columna que diferencia los dos tipos
- También en: [[VPC]] · [[vpc-cheat-sheet]]

## Dato de examen

- **PrivateLink ⇒ interface endpoint ⇒ ENI + security group + costo por hora y por GB.** Si la pregunta dice "gratis" o "solo S3/DynamoDB", es un gateway endpoint, que **no** usa PrivateLink.
- Es la única de las dos formas accesible **desde on-premises** (por VPN o Direct Connect).

## Ver también

[[endpoint-policy]] · [[prefix-list]] · [[eni]]
