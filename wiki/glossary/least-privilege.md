---
title: Least Privilege
category: glossary
tags: [iam, seguridad, best-practices, policies]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Policies and permissions in AWS Identity and Access Management - AWS Identity and Access Management.md", "raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.03 IAM — Conceptos básicos.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.09 Security Token Service (STS).md"]
updated: 2026-09-28
---

# Least Privilege (mínimo privilegio)

> **En una línea:** cada identidad recibe **exactamente** los permisos que necesita, ni uno más.

## Definición

Principio rector de [[IAM]]: se parte de que **todo está denegado** (default deny) y se conceden permisos puntuales, acotando `Action`, `Resource` y `Condition`. Reduce el [[blast-radius]] de una credencial filtrada o de un error.

## Dónde aparece

- [[IAM]] — casos de uso y **IAM Access Analyzer** (policy generation a partir de la actividad real de [[CloudTrail]])
- [[iam-policy-evaluation]] — default deny y managed vs. inline
- [[Organizations]] — *service last accessed data* para saber qué se usa realmente antes de restringir
- También en: [[abac]] · [[bastion-host]] · [[blast-radius]] · [[break-glass]] · [[confused-deputy]] · [[endpoint-policy]] · [[implicit-deny]] · [[instance-profile]] · [[permissions-boundary]] · [[prefix]] · [[principal]] · [[role-separation]] · [[saas]] · [[temporary-credentials]] · [[worm]] · [[EKS]] · [[irsa]]

## Dato de examen

- Las **AWS Managed policies tienden a ser demasiado amplias** → para producción, customer managed.
- Herramientas que el examen asocia a least privilege: **Access Analyzer** (policy generation, unused access, policy validation), [[permissions-boundary|permissions boundaries]] y SCPs.
- "Dentro de AWS usá **roles**, nunca access keys hardcodeadas" es least privilege aplicado a credenciales.

## Ver también

[[permissions-boundary]] · [[role-separation]] · [[blast-radius]] · [[temporary-credentials]] · [[abac]]
