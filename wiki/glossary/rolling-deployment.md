---
title: Rolling deployment
category: glossary
tags: [deployment, ecs, cicd, estrategias-de-deploy]
exam: [DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.04 ECS — Concepts.md"]
updated: 2026-09-28
---

# Rolling deployment

> **En una línea:** un deploy que **reemplaza la versión vieja por la nueva de a tandas**, sobre la misma flota, sin levantar un entorno paralelo.

## Definición

En [[ECS]] es el deploy por defecto de un service: al actualizar la task definition, ECS va bajando tasks viejas y subiendo nuevas. Dos parámetros lo controlan:

| Parámetro | Qué limita |
|---|---|
| `minimumHealthyPercent` | Mínimo de tasks (% del desired count) que tienen que seguir corriendo |
| `maximumPercent` | Máximo de tasks que puede haber corriendo a la vez |

Durante el deploy conviven las dos versiones, y volver atrás implica otro deploy.

## Dónde aparece

- [[ECS]] — deployments de un service
- [[dva-deployment]] — estrategias de deployment del Task 4

## Dato de examen

- "Actualizar sin perder capacidad" → subir `maximumPercent` para levantar las nuevas antes de bajar las viejas. "Ahorrar recursos y tolerar menos capacidad" → bajar `minimumHealthyPercent`.
- Si piden volver atrás al instante o probar con una parte del tráfico → [[blue-green-deployment|blue/green]], no rolling.

## Ver también

[[blue-green-deployment]]
