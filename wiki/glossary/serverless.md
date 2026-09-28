---
title: Serverless
category: glossary
tags: [modelos-de-servicio, serverless, lambda, arquitectura]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.10 CloudFormation — Basics.md", "raw/notas curso mejorado/04 S3/04.11 S3 Presigned URLs.md", "raw/notas curso mejorado/04 S3/04.13 CORS (Cross-Origin Resource Sharing).md"]
updated: 2026-09-28
---

# Serverless

> **En una línea:** no administrás servidores y **pagás por uso**, no por tiempo encendido.

## Definición

Modelo donde AWS provisiona, escala y parchea la infraestructura de forma invisible: no hay instancia que elegir ni SO que mantener. Escala a cero (sin uso, sin costo) y hacia arriba automáticamente.

Ejemplos: Lambda, DynamoDB on-demand, [[S3]], SQS, API Gateway, EventBridge, Fargate ([[ecs-ec2-vs-fargate]]).

## Dónde aparece

- [[EC2]] — "eventos, ejecuciones cortas, no quiero administrar servidores → Lambda o gestionado"
- [[shared-responsibility-model]] — parchear Lambda/DynamoDB es **de AWS**
- [[dva-development]] — pilar del dominio de Development
- También en: [[CloudFormation]] · [[EventBridge]] · [[S3]] · [[IAM]] · [[XRay]] · [[execution-role]] · [[iaas]] · [[idempotency]] · [[paas]] · [[presigned-url]] · [[ECS]] · [[containers]] · [[ecs-ec2-vs-fargate]]

## Dato de examen

- Palabras que delatan serverless: *"sin administrar servidores"*, *"pagar solo por lo que se usa"*, *"picos impredecibles"*, *"event-driven"*.
- Serverless **no significa** "sin límites": Lambda tiene timeout, memoria y concurrencia; el examen usa eso como distractor frente a [[EC2]].

## Ver también

[[iaas]] · [[paas]] · [[idempotency]]
