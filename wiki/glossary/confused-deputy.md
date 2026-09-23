---
title: Confused Deputy
category: glossary
tags: [iam, seguridad, cross-account, roles, ataques]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-09-19
---

# Confused Deputy

> **En una línea:** un tercero con permisos legítimos es **engañado** para usarlos sobre la cuenta equivocada.

## Definición

Un SaaS (el "deputy") tiene permiso para asumir roles en las cuentas de **muchos** clientes. Si un atacante adivina o averigua el ARN de tu rol y consigue que el SaaS lo asuma en su nombre, el deputy ejecuta acciones en **tu** cuenta creyendo actuar para otro cliente.

**Mitigación: [[external-id|`External ID`]].** Un identificador secreto que la [[trust-policy]] del rol exige mediante `sts:ExternalId`; el SaaS debe presentarlo al asumir. Sin el valor correcto, no hay asunción.

## Dónde aparece

- [[IAM]] — gotchas: "un rol de terceros (SaaS) debe exigir External ID"
- [[CloudFront]] — variante del mismo patrón: la bucket policy de OAC exige `AWS:SourceArn` = ARN de la distribución
- [[KMS]] — `SourceArn` como grant constraint obligatorio cuando el grantee es un service principal

## Dato de examen

- Palabra clave en el enunciado: **"proveedor externo / herramienta de terceros accede a mi cuenta"** → la respuesta correcta menciona **External ID**.
- Para **servicios AWS** el equivalente son las condition keys `aws:SourceArn` y `aws:SourceAccount` en la resource policy.

## Ver también

[[trust-policy]] · [[principal]] · [[least-privilege]] · [[cross-account]]
