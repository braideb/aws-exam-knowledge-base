---
title: Principal
category: glossary
tags: [iam, seguridad, policies, autenticacion]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Principal

> **En una línea:** **quién** hace la solicitud a AWS — todavía sin verificar.

## Definición

Una entidad (persona, aplicación, servicio AWS) que emite un request. El flujo es: **Principal → Authentication → Authorization**. Una vez autenticado pasa a ser una *authenticated identity*; recién ahí IAM evalúa sus [[iam-policy-evaluation|policies]].

En una policy, el campo **`Principal`** indica a quién aplica el statement — y **solo existe en resource policies** (bucket policies, key policies, [[trust-policy|trust policies]]).

## Dónde aparece

- [[IAM]] — Principal → Authentication → Authorization
- [[iam-policy-evaluation]] — anatomía del statement
- [[S3]] — bucket policies con `Principal` para acceso anónimo o [[cross-account]]

## Dato de examen

- **¿Cómo distingo una resource policy de una identity policy?** → la resource policy tiene campo **`Principal`**.
- Un **IAM group no es una identidad** → **no puede** ir como `Principal`.
- `arn:aws:iam::123456789012:root` como `Principal` significa **la cuenta entera**, no el root user literal ([[arn]]).
- **Service principal** = un servicio AWS actuando (`cloudfront.amazonaws.com`, `s3.amazonaws.com`).

## Ver también

[[trust-policy]] · [[federation]] · [[least-privilege]]
