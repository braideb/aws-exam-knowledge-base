---
title: Instance Profile
category: glossary
tags: [iam, ec2, roles, credenciales, imds]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Instance Profile

> **En una línea:** el envoltorio que **entrega un IAM role a una instancia EC2**.

## Definición

Un IAM role no se puede attachear directamente a una [[EC2|instancia]]: se attachea un **instance profile**, que es el contenedor del rol. La consola crea el profile automáticamente al asignar un rol, por eso los nombres suelen coincidir y se confunden.

La instancia obtiene [[temporary-credentials|credenciales temporales]] a través del **IMDS** (metadata service), y la CLI y los SDKs las toman **solas** — sin configurar nada.

## Dónde aparece

- [[EC2]] — integración con IAM: "credenciales temporales vía instance profile, sin access keys"
- [[aws-cli]] — paso **6** (último) de la cadena de credenciales
- [[IAM]] — escenario 1 de uso de roles: un servicio AWS actúa por vos

## Dato de examen

- "¿Cómo accede una app en EC2 a S3 **sin hardcodear keys**?" → **instance profile / instance role**. Es la respuesta correcta prácticamente siempre.
- La CLI usa el instance profile solo si **no** encontró credenciales antes en la cadena: `--profile` y las **variables de entorno** lo pisan.
- El equivalente en contenedores es el **ECS task role** (paso 5 de la cadena).

## Ver también

[[temporary-credentials]] · [[trust-policy]] · [[least-privilege]]
