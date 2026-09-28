---
title: ENI (Elastic Network Interface)
category: glossary
tags: [vpc, ec2, networking, security-groups]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.07 VPC Security Groups (SGs).md", "raw/doc oficial/IP addressing for your VPCs and subnets - Amazon Virtual Private Cloud.md"]
updated: 2026-09-28
---

# ENI — Elastic Network Interface

> **En una línea:** la **tarjeta de red virtual** de un recurso dentro de una VPC.

## Definición

Vive en una subnet (y por lo tanto en una AZ) y lleva las IPs privadas (primaria y secundarias), la IP pública o [[elastic-ip|Elastic IP]], la IPv6, y los **security groups**. Toda instancia EC2 tiene una **ENI primaria** (`eth0`); se pueden agregar más. Muchos servicios gestionados (NAT Gateway, Lambda en VPC, endpoints) crean ENIs "requester-managed" en tus subnets.

## Dónde aparece

- [[VPC]] — los SGs se asocian a ENIs; el NAT GW regional se expande a las AZs donde hay ENIs
- [[security-groups-vs-nacls]] — nivel de operación del SG
- [[EC2]] — la ENI primaria lleva las IPs, el DNS y los security groups; el **source/destination check** vive acá
- [[lambda-in-vpc]] — Lambda crea ENIs en tus subnets y consume sus IPs libres
- [[vpc-endpoints]] — un interface endpoint **es** una ENI con IP privada
- También en: [[gateway-vs-interface-endpoint]] · [[nat-gateway-vs-nat-instance]] · [[vpc-cheat-sheet]] · [[vpc-flow-logs]] · [[elastic-ip]] · [[privatelink]] · [[traffic-mirroring]] · [[ECS]] · [[dynamic-port-mapping]] · [[ecs-ec2-vs-fargate]]

## Dato de examen

- Los **security groups se asocian a ENIs**, no a instancias ni a subnets (aunque la consola lo muestre "en la instancia").
- La IP privada primaria **no cambia** al detener/arrancar; se libera al terminar.

## Ver también

[[elastic-ip]] · [[az-resilient]]
