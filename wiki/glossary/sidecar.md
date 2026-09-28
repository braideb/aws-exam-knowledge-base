---
title: Sidecar
category: glossary
tags: [containers, ecs, patrones, observabilidad]
exam: [DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.04 ECS — Concepts.md"]
updated: 2026-09-28
---

# Sidecar

> **En una línea:** un **container auxiliar que corre al lado del container de la app**, dentro de la misma task (o pod), para darle una función extra.

## Definición

La app y el sidecar comparten el ciclo de vida y la red de la task, pero cada uno tiene su propia image. El sidecar se ocupa de algo transversal (tracing, logs, proxy) sin tocar el código de la app. Es el caso típico de task de [[ECS]] con más de un container.

## Dónde aparece

- [[ECS]] — task con varios containers
- [[XRay]] — el daemon de X-Ray corre como sidecar en ECS
- [[EKS]] — en Kubernetes el mismo patrón vive dentro de un pod (containers tightly coupled)
- También en: [[dva-development]] · [[dva-troubleshooting]]

## Dato de examen

- "Habilitar X-Ray en ECS" → correr el **X-Ray daemon como sidecar container** en la task, y darle `xray:PutTraceSegments` al [[task-role|task role]].

## Ver también

[[task-role]] · [[distributed-tracing]]
