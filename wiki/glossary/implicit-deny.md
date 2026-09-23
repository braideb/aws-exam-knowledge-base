---
title: Implicit deny
category: glossary
tags: [security, firewall, iam, nacl, security-groups]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.06 Network Access Control Lists (NACLs).md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.07 VPC Security Groups (SGs).md"]
updated: 2026-09-23
---

# Implicit deny

> **En una línea:** todo lo que **no está permitido explícitamente, está denegado**.

## Definición

El comportamiento por defecto cuando ninguna regla matchea. Se distingue del **explicit deny**, que es una regla escrita que deniega algo concreto (y que le gana a cualquier allow).

| Mecanismo | Implicit deny | Explicit deny |
|---|---|---|
| Security Group | ✅ (lo único que hay) | ❌ no existe |
| NACL | ✅ regla `*` al final | ✅ reglas DENY numeradas |
| IAM policies | ✅ default deny | ✅ `"Effect": "Deny"` |

## Dónde aparece

- [[security-groups-vs-nacls]] — por qué un SG no puede bloquear una IP concreta
- [[VPC]] — regla `*` de las NACLs
- [[iam-policy-evaluation]] — el mismo principio en IAM (Deny > Allow > default deny)
- También en: [[S3]]

## Dato de examen

- Un SG **solo tiene implicit deny**: si permitís `0.0.0.0/0`, no hay forma de excluir una IP → para eso, **NACL con DENY**.

## Ver también

[[stateful-firewall]] · [[least-privilege]]
