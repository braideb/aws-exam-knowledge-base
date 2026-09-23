---
title: Bastion host (jumpbox)
category: glossary
tags: [vpc, ec2, security, ssh]
exam: [SAA-C03, DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.04 VPC Routing e Internet Gateway.md", "raw/doc oficial/Infrastructure security in Amazon VPC - Amazon Virtual Private Cloud.md"]
updated: 2026-09-23
---

# Bastion host (jumpbox)

> **En una línea:** una instancia en una **public subnet** que sirve de puerta de entrada para administrar recursos privados.

## Definición

Todas las conexiones de administración (SSH/RDP) llegan al bastion; desde ahí se "salta" a las instancias de las private subnets. Se endurece aceptando solo ciertas IPs de origen, autenticación SSH o integración con el directorio corporativo. **Bastion host** y **jumpbox** son lo mismo.

## Dónde aparece

- [[VPC]] — sección bastion host / jumpbox
- [[nat-gateway-vs-nat-instance]] — una NAT instance puede hacer de bastion; un NAT GW no
- [[public-subnet]]

**Por qué es mala práctica:** puerto 22 abierto a internet **24/7** (superficie de ataque permanente), llaves SSH que hay que rotar y proteger, y es un [[single-point-of-failure|punto único de falla]] del acceso administrativo — si se cae o se compromete, se pierde (o se filtra) la entrada.

**La alternativa: SSM Session Manager.** Invierte el sentido de la conexión: la instancia **sale** hacia el servicio SSM, nadie entra. Requiere el **SSM Agent** (preinstalado en la mayoría de las AMIs de Amazon) y un **rol IAM** con permisos de SSM ([[instance-profile]]). Shell desde consola o CLI, sin puerto 22, sin llaves, y con la sesión auditada en [[CloudTrail]]/[[CloudWatchLogs]].

## Dato de examen

- El curso: "los jumpboxes son mala práctica, pero hay que saber reconocerlos".
- "Acceso administrativo seguro **sin puertos abiertos** ni gestionar llaves" → **Session Manager**, nunca bastion.

## Ver también

[[public-subnet]] · [[least-privilege]]
