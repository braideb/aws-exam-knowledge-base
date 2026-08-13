---
title: Temporary Credentials
category: glossary
tags: [iam, sts, roles, credenciales, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-24
---

# Temporary Credentials (credenciales temporales)

> **En una línea:** credenciales que **caducan solas** — las que entrega STS al asumir un rol.

## Definición

Al llamar `sts:AssumeRole` (o sus variantes federadas), STS devuelve cuatro cosas:

`AccessKeyId` + `SecretAccessKey` + **`SessionToken`** + `Expiration`

El `SessionToken` es lo que las distingue de unas access keys de largo plazo. **No se almacenan** junto al user: se generan on-demand y expiran sin que haya que revocarlas una por una.

## Dónde aparece

- [[IAM]] — IAM Roles, STS y los 5 escenarios
- [[aws-cli]] — cadena de credenciales; perfiles que asumen rol
- [[EC2]] — [[instance-profile]]: credenciales temporales entregadas por IMDS

## Dato de examen

- Duración: **15 min – 12 h**; con **role chaining**, máximo **1 h**.
- STS es **global** (`sts.amazonaws.com`) con endpoints regionales opcionales; las credenciales **funcionan globalmente** sin importar dónde se emitieron.
- Las credenciales ya emitidas **siguen vivas** aunque le quites permisos al rol → se cortan con una policy sobre **`aws:TokenIssueTime`**.
- **No generar [[presigned-url|presigned URLs]] de [[S3]] con un rol**: la URL muere cuando expira la sesión.

## Ver también

[[trust-policy]] · [[federation]] · [[instance-profile]] · [[least-privilege]]
