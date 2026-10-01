---
title: SSRF (Server-Side Request Forgery)
category: glossary
tags: [seguridad, ec2, imds, web]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.24 Instance Metadata (IMDS).md", "raw/notas curso mejorado/09 Advanced EC2/09.03 EC2 Instance Roles e Instance Profiles.md"]
updated: 2026-09-30
---

# SSRF (Server-Side Request Forgery)

> **En una línea:** un atacante logra que **tu servidor haga requests por él**, hacia direcciones a las que el atacante no llega directamente.

## Definición

Es una vulnerabilidad de la aplicación web. Por ejemplo, un parámetro "traer imagen desde esta URL" que no se valida. En AWS el blanco clásico es el IMDS: el atacante le pide al servidor `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>` y se queda con las **credenciales temporales del role**. Un SSRF típico solo puede hacer un `GET` simple.

## Dónde aparece

- [[ec2-instance-metadata]]: IMDSv1 vs IMDSv2
- También en: [[dva-security]] · [[ec2-cheat-sheet]] · [[IAM]]

## Dato de examen

- "Proteger las credenciales del role frente a SSRF" → **exigir IMDSv2** (`HttpTokens: required`). IMDSv2 necesita un `PUT` con header y tiene **hop limit 1**, y un SSRF no puede cumplir ninguna de las dos cosas.

## Ver también

[[instance-profile]] · [[temporary-credentials]]
