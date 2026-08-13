---
title: Permissions Boundary
category: glossary
tags: [iam, policies, seguridad, delegacion]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Permissions Boundary

> **En una línea:** el **techo** de permisos de un user o rol — no concede nada por sí sola.

## Definición

Managed policy que limita lo máximo que las identity policies de un user o role pueden llegar a conceder:

```
efectivo = identity policy ∩ permissions boundary
```

Si además hay un [[Organizations|SCP]], las **tres capas** deben permitir la acción. Aplica a **users y roles**; no a grupos ni a service-linked roles.

## Dónde aparece

- [[iam-policy-evaluation]] — sección "Permissions boundaries en detalle"
- [[IAM]] — a un service-linked role no se le puede aplicar boundary
- [[Organizations]] — el SCP es el mismo mecanismo a nivel cuenta

## Dato de examen

- Caso de uso estrella: **delegación segura**. Se exige por policy que todo user creado lleve cierta boundary (condition **`iam:PermissionsBoundary`**), para que el delegado no pueda crear identidades más poderosas que él.
- Matiz fino: una **resource policy** que da acceso directamente al ARN de un user **no queda recortada** por el implicit deny de la boundary. Un **Deny explícito** en la boundary, en cambio, gana siempre.
- No confundir con SCP: la boundary aplica a **una identidad**; el SCP a **toda la cuenta**.

## Ver también

[[least-privilege]] · [[principal]] · [[iam-policy-evaluation]]
