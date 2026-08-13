---
title: Serverless
category: glossary
tags: [modelos-de-servicio, serverless, lambda, arquitectura]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Serverless

> **En una línea:** no administrás servidores y **pagás por uso**, no por tiempo encendido.

## Definición

Modelo donde AWS provisiona, escala y parchea la infraestructura de forma invisible: no hay instancia que elegir ni SO que mantener. Escala a cero (sin uso, sin costo) y hacia arriba automáticamente.

Ejemplos: Lambda, DynamoDB on-demand, [[S3]], SQS, API Gateway, EventBridge, Fargate.

## Dónde aparece

- [[EC2]] — "eventos, ejecuciones cortas, no quiero administrar servidores → Lambda o gestionado"
- [[shared-responsibility-model]] — parchear Lambda/DynamoDB es **de AWS**
- [[dva-development]] — pilar del dominio de Development

## Dato de examen

- Palabras que delatan serverless: *"sin administrar servidores"*, *"pagar solo por lo que se usa"*, *"picos impredecibles"*, *"event-driven"*.
- Serverless **no significa** "sin límites": Lambda tiene timeout, memoria y concurrencia; el examen usa eso como distractor frente a [[EC2]].

## Ver también

[[iaas]] · [[paas]] · [[idempotency]]
