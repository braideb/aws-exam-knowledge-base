---
title: Prefix list
category: glossary
tags: [vpc, routing, endpoints, security-groups]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.09 VPC Endpoints.md"]
updated: 2026-09-24
---

# Prefix list

> **En una línea:** un **alias con nombre** (`pl-xxxx`) para un conjunto de rangos [[cidr|CIDR]], usable como destino de una ruta o como origen de una regla de security group.

## Definición

En vez de escribir a mano las decenas de rangos públicos que usa un servicio en una región, referenciás una prefix list. Las **managed prefix lists de AWS** representan servicios (S3, DynamoDB) y AWS las mantiene actualizada sola; también podés crear las tuyas (*customer-managed*) para agrupar, por ejemplo, los rangos de tus oficinas.

## Dónde aparece

- [[vpc-endpoints]] — el gateway endpoint agrega una ruta cuyo **destino es una prefix list**
- [[VPC]] — route tables
- También en: [[gateway-vs-interface-endpoint]] · [[privatelink]] · [[security-groups-vs-nacls]]

## Dato de examen

- La ruta de un gateway endpoint es **más específica** que `0.0.0.0/0`, así que le gana a la default route hacia el NAT Gateway por [[longest-prefix-match|longest prefix match]]. Ese es el motivo por el que el endpoint "simplemente funciona" sin tocar la app.
- Si AWS agrega rangos al servicio, la prefix list se actualiza y **no hay que editar la route table**.

## Ver también

[[longest-prefix-match]] · [[cidr]] · [[vpc-endpoints]]
