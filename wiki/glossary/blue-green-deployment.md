---
title: Blue/green deployment
category: glossary
tags: [deployment, ecs, codedeploy, cicd, estrategias-de-deploy]
exam: [DVA-C02, DOP-C02, SAA-C03]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.04 ECS — Concepts.md"]
updated: 2026-09-28
---

# Blue/green deployment

> **En una línea:** levantar la versión nueva (**green**) **al lado** de la vieja (**blue**) y pasar el tráfico de una a otra, con rollback inmediato volviendo el tráfico.

## Definición

En [[ECS]] lo hace **CodeDeploy**: crea un segundo grupo de tasks con la versión nueva y mueve el tráfico del ALB. El traspaso puede ser:

| Modo | Cómo mueve el tráfico |
|---|---|
| **All at once** | Todo de una vez |
| **Canary** | Un porcentaje primero y el resto después |
| **Linear** | En incrementos iguales cada cierto tiempo |

## Dónde aparece

- [[ECS]] — deployments de un service
- [[dva-deployment]] — estrategias de deployment del Task 4

## Dato de examen

- "Probar la versión nueva con el 10% del tráfico y después pasar todo" → blue/green **canary**.
- "Rollback inmediato sin redeployar" → blue/green (el entorno viejo sigue arriba).
- Cuesta más que [[rolling-deployment|rolling]] mientras dura, porque corren las dos versiones completas.

## Ver también

[[rolling-deployment]]
