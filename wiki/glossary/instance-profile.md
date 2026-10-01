---
title: Instance Profile
category: glossary
tags: [iam, ec2, roles, credenciales, imds]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.07 Cuándo usar IAM Roles - los cinco escenarios.md", "raw/notas curso mejorado/09 Advanced EC2/09.03 EC2 Instance Roles e Instance Profiles.md"]
updated: 2026-09-30
---

# Instance Profile

> **En una línea:** el envoltorio que **entrega un IAM role a una instancia EC2**.

## Definición

Un IAM role no se puede attachear directamente a una [[EC2|instancia]]: se attachea un **instance profile**, que es el contenedor del rol. La consola crea el profile automáticamente, con el mismo nombre, al crear un rol de EC2. Por eso los nombres suelen coincidir y se confunden. Con la **CLI o CloudFormation** hay que crear **las dos cosas por separado** (rol + `AWS::IAM::InstanceProfile`).

La instancia obtiene [[temporary-credentials|credenciales temporales]] a través del **IMDS** (metadata service), y la CLI y los SDKs las toman **solas** — sin configurar nada.

## Dónde aparece

- [[EC2]] — integración con IAM: "credenciales temporales vía instance profile, sin access keys"
- [[aws-cli]] — paso **6** (último) de la cadena de credenciales
- [[IAM]] — escenario 1 de uso de roles: un servicio AWS actúa por vos
- [[ec2-instance-metadata]] — el mecanismo concreto por el que la instancia recibe las credenciales
- [[XRay]] — el daemon en EC2 publica traces con los permisos del instance role
- [[ec2-bootstrapping]] — el script de user data lee los secretos de Parameter Store con el rol
- También en: [[SSMParameterStore]] · [[parameter-store-vs-secrets-manager]] · [[CloudWatchLogs]] · [[dva-security]] · [[ec2-cheat-sheet]] · [[execution-role]] · [[service-linked-role]] · [[temporary-credentials]] · [[trust-policy]] · [[ECS]] · [[task-role]]

## Dato de examen

- "¿Cómo accede una app en EC2 a S3 **sin hardcodear keys**?" → **instance profile / instance role**. Es la respuesta correcta prácticamente siempre.
- La CLI usa el instance profile solo si **no** encontró credenciales antes en la cadena: `--profile` y las **variables de entorno** lo pisan.
- El equivalente en contenedores es el **[[task-role|ECS task role]]** (paso 5 de la cadena).

## Ver también

[[temporary-credentials]] · [[trust-policy]] · [[least-privilege]]
