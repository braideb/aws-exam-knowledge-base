---
title: Control Plane
category: glossary
tags: [arquitectura, cloudtrail, api, gobernanza]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Control Plane

> **En una línea:** las operaciones que **crean, configuran o destruyen** un recurso.

## Definición

El conjunto de APIs de gestión: crear una instancia [[EC2]], crear un bucket, cambiar una policy, borrar una [[VPC]]. Es "administrar el recurso", en contraposición a usar lo que hay adentro ([[data-plane]]).

## Dónde aparece

- [[CloudTrail]] — los **Management Events** son las operaciones de control plane, y están **activados por defecto**
- [[S3]] — el **alias** de un access point sirve para data plane pero **no** para control plane

## Dato de examen

- La división explica la factura de [[CloudTrail]]: management events **gratis** en Event History (90 días); data events **opt-in y pagos**.
- Separar planos es la base de [[role-separation]]: se puede administrar un recurso sin poder leer sus datos.

## Ver también

[[data-plane]] · [[role-separation]]
