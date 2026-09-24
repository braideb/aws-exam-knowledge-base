---
title: Data Plane
category: glossary
tags: [arquitectura, cloudtrail, api]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Referencing access points with ARNs, access point aliases, or virtual-hosted–style URIs - Amazon Simple Storage Service.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.13 CloudTrail.md", "raw/doc oficial/Understanding CloudTrail events.md"]
updated: 2026-09-24
---

# Data Plane

> **En una línea:** las operaciones **sobre el contenido** de un recurso ya creado.

## Definición

Lo que ocurre *dentro* del recurso: `GetObject`/`PutObject` en [[S3]], `Invoke` de Lambda, operaciones item-level de DynamoDB, `Publish` de SNS, mensajes de SQS. Suelen ser órdenes de magnitud más frecuentes que las de [[control-plane]].

## Dónde aparece

- [[CloudTrail]] — los **Data Events** son opt-in, con **costo extra** y volumen enorme
- [[S3]] — el alias de un access point se usa como si fuera un bucket name en operaciones de data plane
- También en: [[observability-costs]] · [[control-plane]] · [[role-separation]]

## Dato de examen

- "Necesito saber **quién leyó** un objeto de S3" → hay que **habilitar data events** en el trail; por defecto no se registran.
- "Quién **borró el bucket**" es control plane → sale gratis en Event History.

## Ver también

[[control-plane]] · [[role-separation]]
