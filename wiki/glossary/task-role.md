---
title: Task role
category: glossary
tags: [ecs, iam, roles, permisos, containers]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.04 ECS — Concepts.md"]
updated: 2026-10-03
---

# Task role

> **En una línea:** el **IAM role que asume una task de ECS** para que el código de sus containers llame a servicios de AWS.

## Definición

Se declara en la task definition. Cuando la task arranca, lo asume y entrega [[temporary-credentials|credenciales temporales]] a los containers. Es la best practice para darle permisos a una app en containers, y el equivalente del [[execution-role]] de Lambda y del [[instance-profile]] de EC2.

Hay tres roles que se confunden seguido en ECS:

| Rol | Lo usa | Para |
|---|---|---|
| **Task role** | Tu código | S3, DynamoDB, SQS… |
| **Task execution role** | ECS agent / Fargate | Pull de [[ECR]], logs a CloudWatch Logs, secrets |
| **Container instance role** | ECS agent (solo EC2 mode) | Registrar la instancia en el cluster |

## Dónde aparece

- [[ECS]] — tabla de los tres roles y gotchas
- [[IAM]] — el mismo patrón de "rol asumido por el servicio"
- [[aws-cli]] — paso de la cadena de credenciales del SDK
- [[EKS]] — su equivalente es [[irsa|IRSA / EKS Pod Identity]]
- También en: [[ECR]] · [[XRay]] · [[dva-development]] · [[dva-security]] · [[SecretsManager]]

## Dato de examen

- "La app en el container no puede escribir en S3" → **task role**. "La task no puede hacer pull de la image o no manda logs" → **task execution role**.
- Según el instructor, el task role aparece en al menos una pregunta del examen.

## Ver también

[[execution-role]] · [[instance-profile]] · [[irsa]] · [[sidecar]]
