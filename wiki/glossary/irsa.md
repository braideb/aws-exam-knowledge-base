---
title: IRSA (IAM Roles for Service Accounts)
category: glossary
tags: [eks, kubernetes, iam, roles, permisos]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.10 Elastic Kubernetes Service (EKS) 101.md"]
updated: 2026-09-28
---

# IRSA (IAM Roles for Service Accounts)

> **En una línea:** la forma de darle un **IAM role a los pods de EKS** a través de un service account de Kubernetes, en vez de usar el role del node.

## Definición

Se asocia un IAM role a un **service account**, y los pods que usan ese service account reciben [[temporary-credentials|credenciales temporales]] del role. La alternativa más nueva y simple es **EKS Pod Identity**, con la misma idea. Por debajo, IRSA usa `AssumeRoleWithWebIdentity` de STS.

## Dónde aparece

- [[EKS]] — permisos de los pods
- [[IAM]] — `AssumeRoleWithWebIdentity` y los pods de EKS
- También en: [[dva-security]]

## Dato de examen

- "Dar permisos de S3 solo a un pod" → **IRSA / EKS Pod Identity**. Usar el role del node le daría el permiso a todos los pods del node.
- Es el equivalente del [[task-role|task role]] de ECS.

## Ver también

[[task-role]] · [[least-privilege]] · [[federation]]
