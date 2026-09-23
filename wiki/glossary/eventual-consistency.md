---
title: Eventual Consistency
category: glossary
tags: [consistencia, sistemas-distribuidos, kms, s3, iam]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-09-23
---

# Eventual Consistency

> **En una línea:** el cambio **termina** propagándose, pero por unos instantes podés leer datos viejos.

## Definición

En un sistema distribuido, una escritura tarda en replicarse a todas las réplicas. Durante esa ventana, distintas lecturas pueden devolver resultados distintos. Lo contrario es **strong consistency**: toda lectura posterior a una escritura ve el dato nuevo.

Dónde aparece cada una en AWS:

| Comportamiento | Ejemplos |
|---|---|
| **Strong (read-after-write)** | [[S3]] — todas las operaciones, desde 2020 |
| **Eventual** | Grants de [[KMS]], propagación de policies de [[IAM]], réplicas de lectura, DynamoDB (lectura eventual por default) |

## Dónde aparece

- [[S3]] — "consistencia **strong read-after-write** para todas las operaciones" (dato de examen)
- [[KMS]] — un grant recién creado tarda en propagarse → **grant token**
- También en: [[IAM]]

## Dato de examen

- "**AccessDenied** justo después de `CreateGrant`" → eventual consistency; la solución es usar el **grant token** que devuelve `CreateGrant` (no `ListGrants`, que da el grant ID).
- S3 **ya no** es eventualmente consistente: material viejo que dice lo contrario está desactualizado.
- Un consumidor de eventos de S3 debe ser [[idempotency|idempotente]]: la entrega es **at least once**.

## Ver también

[[idempotency]] · [[durability]]
