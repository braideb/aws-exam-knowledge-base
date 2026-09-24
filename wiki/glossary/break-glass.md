---
title: Break Glass
category: glossary
tags: [iam, roles, seguridad, operaciones, patrones]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.07 Cuándo usar IAM Roles - los cinco escenarios.md"]
updated: 2026-09-24
---

# Break Glass

> **En una línea:** acceso de emergencia que existe, pero que **deja rastro cuando se usa**.

## Definición

El equivalente digital de la cajita roja que dice *"romper el vidrio en caso de emergencia"*: un rol con permisos elevados que alguien con acceso limitado puede **asumir puntualmente** durante un incidente.

**No es una feature de AWS** — es un **patrón** que se arma con piezas normales de [[IAM]]:

1. Un rol con permisos elevados.
2. Una [[trust-policy]] que define quién puede asumirlo.
3. Fricción deliberada: **MFA** obligatorio.
4. Una **alerta automática** (Slack, PagerDuty) cada vez que alguien lo asume — para que romper el vidrio no pase desapercibido.

## Dónde aparece

- [[IAM]] — escenario 2 de los cinco usos de roles

## Dato de examen

Por qué es mejor que darle permisos altos permanentes a la persona:

- **El acceso se vence solo** ([[temporary-credentials]], típicamente 1 h) — nadie tiene que acordarse de revocarlo.
- **Queda todo registrado**: [[CloudTrail]] loguea el `AssumeRole` **y** cada acción hecha con el rol.
- Es una **decisión consciente**: en el día a día la persona ni siquiera tiene esos permisos disponibles, así que no rompe nada por descuido.

## Ver también

[[trust-policy]] · [[temporary-credentials]] · [[least-privilege]]
