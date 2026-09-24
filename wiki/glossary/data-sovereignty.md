---
title: Data Sovereignty
category: glossary
tags: [compliance, regions, gobernanza, legal]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/04 S3/04.10 S3 Replication.md"]
updated: 2026-09-24
---

# Data Sovereignty

> **En una línea:** los datos quedan sujetos a las leyes del **país donde están almacenados**.

## Definición

Principio legal por el cual la información se rige por la jurisdicción del territorio que la aloja. En AWS se materializa en la elección de **region**: los datos de una region **no salen de ella** salvo configuración explícita tuya (replicación, backups, exportaciones).

## Dónde aparece

- [[global-infrastructure]] — "soberanía de datos" como motivo de la separación por regions
- [[S3]] — SRR (misma region) como caso de uso de soberanía; CRR como decisión **deliberada** de sacar datos
- También en: [[cidr]] · [[region-resilient]]

## Dato de examen

- Escenario "los datos de clientes europeos no pueden salir de la UE" → elegir la region correcta **y** restringir por [[Organizations|SCP]] las regiones habilitadas (exceptuando siempre servicios globales como IAM, STS y CloudFront, o se rompe todo).
- Nombre global ≠ dato global: el nombre de un bucket de S3 es único a nivel mundial, pero su contenido vive en una sola region ([[region-resilient]]).

## Ver también

[[region-resilient]] · [[blast-radius]]
