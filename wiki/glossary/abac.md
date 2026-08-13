---
title: ABAC (Attribute-Based Access Control)
category: glossary
tags: [iam, policies, tags, seguridad, escalabilidad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-25
---

# ABAC — Attribute-Based Access Control

> **En una línea:** permisos por **tag**, en vez de enumerar recursos uno por uno.

## Definición

En vez de listar cada ARN en el `Resource` de una policy (eso es **RBAC**, role-based), se concede acceso a todo lo que lleve cierto atributo:

```json
"Condition": { "StringEquals": { "aws:ResourceTag/Environment": "dev" } }
```

Escala muchísimo mejor: **una sola policy cubre recursos que todavía no existen**, siempre que se creen con el tag correcto. El contracara es que la seguridad pasa a depender de que el **tagging sea disciplinado**.

## Dónde aparece

- [[iam-policy-evaluation]] — sección del campo `Condition`
- [[least-privilege]] — cómo acotar sin escribir una policy por recurso

## Dato de examen

- Enunciado *"dar acceso solo a los recursos del entorno `dev` sin listarlos uno por uno"* o *"la lista de recursos crece constantemente"* → **ABAC con `aws:ResourceTag`**.
- Condition keys de la familia: `aws:ResourceTag/<clave>` (tag del recurso), `aws:RequestTag/<clave>` (tag que viene en la request), `aws:PrincipalTag/<clave>` (tag de quien llama) y `aws:TagKeys`.
- Pariente cercano: las **variables de policy** (`${aws:username}`), que resuelven el mismo problema de "una policy para muchos" pero por identidad en vez de por tag.

## Ver también

[[least-privilege]] · [[principal]] · [[permissions-boundary]]
