---
title: Trust Policy
category: glossary
tags: [iam, roles, sts, seguridad, policies]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Trust Policy

> **En una línea:** define **quién puede asumir** un rol (la otra policy define qué puede hacer).

## Definición

Toda IAM Role tiene **dos** policies:

| Policy | Responde |
|---|---|
| **Trust policy** | *¿Quién puede asumirme?* — identidades de la cuenta, otras cuentas, servicios AWS, identidades federadas |
| **Permissions policy** | *¿Qué puede hacer quien me asumió?* |

Es una resource policy: lleva campo **`Principal`**. Si el principal está autorizado, **STS** entrega [[temporary-credentials|credenciales temporales]].

## Dónde aparece

- [[IAM]] — sección IAM Roles y los 5 escenarios de uso
- [[KMS]] — la key policy sigue la misma lógica de confianza explícita
- [[Organizations]] — `OrganizationAccountAccessRole` y el "switch role"

## Dato de examen

- **No se permite wildcard (`*`) en el ARN del `Principal`** de una trust policy.
- Un rol para un tercero (SaaS) debe exigir **[[external-id|External ID]]** en la trust policy → previene el [[confused-deputy]].
- [[cross-account|Cross-account]]: hacen falta **las dos puntas** — trust policy en el rol destino **y** permiso `sts:AssumeRole` en la identidad de origen.

## Ver también

[[principal]] · [[temporary-credentials]] · [[confused-deputy]] · [[instance-profile]]
