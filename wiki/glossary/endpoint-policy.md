---
title: Endpoint policy
category: glossary
tags: [vpc, endpoints, iam, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.09 VPC Endpoints.md"]
updated: 2026-09-24
---

# Endpoint policy

> **En una línea:** una **resource policy pegada al VPC endpoint** que limita a qué se puede llegar *a través de él*.

## Definición

No otorga permisos: **acota** los que ya existen. Se evalúa junto con la identity policy y con la resource policy del recurso destino, dentro del esquema de [[iam-policy-evaluation]]. El uso típico es "por este endpoint solo se puede hablar con estos buckets", para que una instancia comprometida no pueda exfiltrar datos hacia un bucket ajeno.

## Dónde aparece

- [[vpc-endpoints]] — es el mecanismo de control de los gateway endpoints
- [[gateway-vs-interface-endpoint]] — endpoint policy vs security group
- También en: [[VPC]] · [[dva-security]] · [[CloudTrail]] · [[privatelink]]

## Dato de examen

- Es **el** control de un **gateway endpoint** (que no tiene security group, porque no es una ENI). Un **interface endpoint** usa **security group + endpoint policy**.
- Una endpoint policy permisiva no habilita nada por sí sola: si la identity policy no permite la acción, sigue denegada.

## Ver también

[[vpc-endpoints]] · [[iam-policy-evaluation]] · [[least-privilege]]
