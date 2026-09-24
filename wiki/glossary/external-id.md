---
title: External ID
category: glossary
tags: [iam, trust-policy, seguridad, cross-account]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/IAM roles - AWS Identity and Access Management.md", "raw/doc oficial/Temporary security credentials in IAM - AWS Identity and Access Management.md"]
updated: 2026-09-24
---

# External ID

> **En una línea:** el identificador secreto que una [[trust-policy|trust policy]] exige a un tercero antes de dejarlo asumir un rol — la defensa estándar contra el [[confused-deputy|confused deputy]].

## Definición

Valor que la trust policy de un rol puede exigir vía la condition `sts:ExternalId`. Cuando un servicio de terceros ([[saas|SaaS]]) necesita asumir un rol en tu cuenta, exigir un External ID evita que ese mismo rol sea usado —por error o por un atacante— en nombre de otro cliente del mismo SaaS: el escenario clásico de confused deputy.

## Dónde aparece

- [[IAM]] — gotcha: "un rol de terceros (SaaS) debe exigir External ID en la trust policy"
- [[confused-deputy]] — la mitigación central del patrón
- [[trust-policy]] — junto con la regla de no-wildcard en el `Principal`
- También en: [[cross-account]]

## Dato de examen

- Enunciado con **"proveedor externo / herramienta de terceros accede a mi cuenta"** → la respuesta correcta menciona **External ID** en la trust policy.
- Para **servicios AWS** (no terceros) el equivalente son las condition keys `aws:SourceArn` / `aws:SourceAccount`.

## Ver también

[[confused-deputy]] · [[trust-policy]] · [[cross-account]]
