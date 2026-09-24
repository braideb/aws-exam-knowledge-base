---
title: Federation (federación de identidades)
category: glossary
tags: [iam, sts, saml, oidc, cognito, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Temporary security credentials in IAM - AWS Identity and Access Management.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.07 Cuándo usar IAM Roles - los cinco escenarios.md", "raw/doc oficial/IAM roles - AWS Identity and Access Management.md"]
updated: 2026-09-24
---

# Federation — federación de identidades

> **En una línea:** identidades de **afuera** de AWS acceden asumiendo un rol, sin ser IAM users.

## Definición

Se confía en un proveedor de identidad externo (IdP) y se le permite **asumir un rol** de la cuenta. La identidad externa nunca se convierte en un IAM user: solo obtiene [[temporary-credentials|credenciales temporales]] vía STS.

Dos sabores:

| Tipo | API de STS | Casos |
|---|---|---|
| **SAML 2.0** | `AssumeRoleWithSAML` | AD FS, IdP corporativo, SSO |
| **OIDC / Web Identity** | `AssumeRoleWithWebIdentity` | Google, Facebook, Login with Amazon |

Para apps móviles/web la recomendación oficial es **Cognito** (mismos IdPs + acceso guest no autenticado).

## Dónde aparece

- [[IAM]] — escenarios 3 y 4 de uso de roles; datos de STS
- [[dva-security]] — dominio de seguridad del DVA-C02
- También en: [[principal]] · [[temporary-credentials]]

## Dato de examen

- **Más de 5.000 identidades** (el límite de IAM users por cuenta) → **roles + federación**, jamás "pedir aumento de límite".
- Una identidad externa **no puede usarse directamente**: solo puede **asumir un rol**.
- Escenario "empleados ya tienen cuentas en Active Directory" → SAML; "millones de usuarios de una app móvil" → Web Identity / Cognito.

## Ver también

[[temporary-credentials]] · [[trust-policy]] · [[principal]]
