---
title: Edge Location
category: glossary
tags: [infraestructura, cdn, edge, cloudfront]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.02 AWS Global Infrastructure.md", "raw/notas curso mejorado/04 S3/04.04 S3 Performance Optimization.md"]
updated: 2026-09-24
---

# Edge Location

> **En una línea:** punto de presencia chico y cercano al usuario — **cache + poco cómputo**, no una region.

## Definición

Instalación pequeña de AWS (**400+ PoPs**, muchos más que regiones) ubicada cerca de los usuarios finales. No tiene la infraestructura completa de una Region: sirve para acercar contenido y ejecutar lógica liviana.

Qué corre en el edge: [[CloudFront]] (CDN), **Lambda@Edge / CloudFront Functions**, **S3 Transfer Acceleration**, [[Route53]] y Global Accelerator (anycast).

## Dónde aparece

- [[global-infrastructure]] — sección Edge Locations
- [[CloudFront]] — cachea y sirve desde el edge
- [[S3]] — Transfer Acceleration sube por la edge location más cercana
- También en: [[ttl]]

## Dato de examen

- Regla mental: **Region = infraestructura completa; Edge = cache + poco cómputo cerca del usuario**.
- El tráfico de salida por CloudFront es **más barato** que el directo de S3/EC2.
- No confundir CloudFront (**cachea descargas**) con S3 Transfer Acceleration (**acelera subidas**, no cachea).

## Ver también

[[globally-resilient]] · [[region-resilient]]
