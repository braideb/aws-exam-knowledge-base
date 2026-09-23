---
title: AZ Resilient
category: glossary
tags: [resiliencia, infraestructura, availability-zones]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-09-23
---

# AZ Resilient

> **En una línea:** vive en **una sola AZ**; si esa AZ cae, el recurso cae.

## Definición

El nivel más bajo de resiliencia: el servicio o recurso se ejecuta dentro de una única Availability Zone y no tiene redundancia propia. El nombre confunde — no significa "resistente a la caída de una AZ", sino "resiliente **solo dentro de** su AZ".

Ejemplos: instancia [[EC2]], volumen EBS, subnet de [[VPC]], RDS single-AZ, NAT Gateway.

## Dónde aparece

- [[global-infrastructure]] — tabla de los tres niveles de resiliencia
- [[EC2]] — "la instancia vive en una subnet → una AZ"
- [[VPC]] — las subnets se mapean 1:1 a una AZ; el NAT Gateway zonal es AZ resilient
- [[vpc-design]] — por qué se repiten las subnets de cada tier en varias AZs
- [[nat-gateway-vs-nat-instance]] — un NAT GW por AZ (o el modo regional)
- [[EBS]] — el volumen vive en una AZ; el snapshot (en S3) es el que sube a región
- También en: [[ec2-cheat-sheet]] · [[vpc-cheat-sheet]]

## Dato de examen

- Los recursos AZ resilient son **siempre el punto débil** de un diagrama: son los que hay que redundar entre AZs.
- Repartir no es sobrevivir: 6 instancias en 3 AZs dejan **4** vivas al caer una AZ. Ver el dimensionamiento N+1 en [[global-infrastructure]].

## Ver también

[[region-resilient]] · [[globally-resilient]] · [[single-point-of-failure]]
