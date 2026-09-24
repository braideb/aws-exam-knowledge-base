---
title: SaaS (Software as a Service)
category: glossary
tags: [modelos-de-servicio, saas, shared-responsibility]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.12 Modelo de responsabilidad compartida (Shared Responsibility Model).md"]
updated: 2026-09-24
---

# SaaS — Software as a Service

> **En una línea:** consumís el producto terminado; **AWS gestiona todo**, incluso la aplicación.

## Definición

El proveedor opera la pila completa —instalaciones, hardware, SO, runtime y la propia aplicación— y vos solo la usás y configurás. Es el extremo opuesto a on-premises.

## Dónde aparece

- [[shared-responsibility-model]] — tabla de capas por modelo de servicio
- También en: [[IAM]] · [[confused-deputy]] · [[external-id]] · [[iaas]] · [[paas]] · [[trust-policy]]

## Dato de examen

- Aun en SaaS quedan cosas del cliente: **quién accede** ([[IAM]], [[least-privilege]]) y **cómo se clasifican y protegen los datos**. "AWS se encarga de todo" nunca es del todo cierto en una pregunta de seguridad.

## Ver también

[[iaas]] · [[paas]] · [[shared-responsibility-model]]
