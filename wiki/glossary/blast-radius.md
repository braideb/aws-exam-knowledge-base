---
title: Blast Radius
category: glossary
tags: [resiliencia, seguridad, arquitectura, multi-account]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.01 Cuenta de AWS (AWS Account).md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.08 S3 Buckets — Basics.md"]
updated: 2026-09-24
---

# Blast Radius

> **En una línea:** hasta dónde llega el daño cuando algo falla o es comprometido.

## Definición

El alcance de la explosión. Es el argumento que justifica casi toda decisión de aislamiento en AWS: separar cuentas por entorno, repartir entre AZs, usar buckets distintos, separar claves de [[KMS]]. Reducir el blast radius no evita el incidente — evita que se propague.

Radios típicos: un bucket de [[S3]] → la **region**; una [[aws-account|AWS Account]] → la **cuenta**; una instancia [[EC2]] → la **AZ**.

## Dónde aparece

- [[aws-account]] — "las cuentas contienen el blast radius de errores y exploits" (DEV/TEST/PROD separadas)
- [[S3]] — el blast radius de un bucket es la region
- [[global-infrastructure]] — repartir componentes entre AZs
- También en: [[data-sovereignty]] · [[globally-resilient]] · [[least-privilege]] · [[region-resilient]] · [[single-point-of-failure]]

## Dato de examen

- Escenario "aislar entornos / limitar el daño de un error humano" → **cuentas separadas** gestionadas con [[Organizations]], no simplemente policies distintas.

## Ver también

[[single-point-of-failure]] · [[region-resilient]] · [[least-privilege]]
