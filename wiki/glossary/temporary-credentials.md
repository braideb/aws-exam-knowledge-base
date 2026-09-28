---
title: Temporary Credentials
category: glossary
tags: [iam, sts, roles, credenciales, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Temporary security credentials in IAM - AWS Identity and Access Management.md", "raw/doc oficial/IAM roles - AWS Identity and Access Management.md", "raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.04 IAM Access Keys.md"]
updated: 2026-09-28
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
- [[ec2-instance-metadata]] — se obtienen del IMDS en `iam/security-credentials/`
- [[execution-role]] — el equivalente para Lambda
- También en: [[Organizations]] · [[observability-costs]] · [[break-glass]] · [[federation]] · [[least-privilege]] · [[presigned-url]] · [[trust-policy]] · [[ECS]] · [[EKS]] · [[irsa]] · [[task-role]]

## Dato de examen

- Duración: **15 min – 12 h**; con **role chaining**, máximo **1 h**.
- STS es **global** (`sts.amazonaws.com`) con endpoints regionales opcionales; las credenciales **funcionan globalmente** sin importar dónde se emitieron.
- Las credenciales ya emitidas **siguen vivas** aunque le quites permisos al rol → se cortan con una policy sobre **`aws:TokenIssueTime`**.
- **No generar [[presigned-url|presigned URLs]] de [[S3]] con un rol**: la URL muere cuando expira la sesión.

## Ver también

[[trust-policy]] · [[federation]] · [[instance-profile]] · [[least-privilege]]
