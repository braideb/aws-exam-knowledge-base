---
title: Region Resilient
category: glossary
tags: [resiliencia, infraestructura, regions, availability-zones]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Region Resilient

> **En una línea:** sobrevive la caída de una **AZ**, no la de la **region**.

## Definición

Servicio que replica automáticamente entre las Availability Zones de su region. Opera mientras quede al menos una AZ sana, pero si cae la region entera, el servicio cae con ella. Su [[blast-radius]] es **la region**.

Ejemplos: [[S3]], DynamoDB, [[VPC]], ELB, RDS [[multi-az|Multi-AZ]].

## Dónde aparece

- [[global-infrastructure]] — tabla de los tres niveles de resiliencia
- [[S3]] — "los objetos se replican entre las AZs de la region y no salen de ella"
- [[VPC]] — "se crea en una cuenta y una region; opera desde múltiples AZs"

## Dato de examen

- La pregunta "¿sobrevive a la caída de la **region**?" siempre se responde **no** para estos servicios: hace falta replicación cross-region **explícita** (S3 CRR, backups en otra region, [[Route53]] [[failover]]).
- Que S3 sea region resilient no contradice que el **nombre del bucket sea único globalmente**: el nombre es global, los datos son regionales.

## Ver también

[[globally-resilient]] · [[az-resilient]] · [[data-sovereignty]]
