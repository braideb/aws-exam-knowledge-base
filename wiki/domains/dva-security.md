---
title: "DVA-C02 · Dominio 2: Security"
category: domain
tags: [dva-c02, security, iam, cifrado, autenticacion]
exam: [DVA-C02]
sources: ["https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html"]
updated: 2026-09-22
---

# DVA-C02 · Dominio 2 — Security

## Peso en el examen

**26%**

## Resumen de lo que el examen evalúa

Implementar autenticación/autorización para las apps (IAM, roles, [[federation|federación]], Cognito), [[encryption-at-rest|cifrado en tránsito y en reposo]], y manejo de datos sensibles (secretos, claves).

## Task statements oficiales (exam guide)

**Task 1 — Autenticación/autorización**: federación con identity providers (Cognito, IAM), bearer tokens, acceso programático, llamadas autenticadas a servicios AWS, **asumir IAM roles**, permisos para [[principal|principals]], autorización app-level de grano fino, autenticación cross-service en microservicios.

**Task 2 — Cifrado con servicios AWS**: at rest vs in transit, gestión de certificados (AWS Private CA), client-side vs server-side, cifrar/descifrar con claves, certificados y llaves SSH para dev, **cifrado [[cross-account]]**, habilitar/deshabilitar rotación de claves.

**Task 3 — Datos sensibles en el código**: clasificación (PII/PHI), cifrar variables de entorno, **secret management** (Secrets Manager), sanitización y masking, patrones de acceso multi-tenant.

## Temas clave — cobertura actual

| Tema | Página | Estado |
|---|---|---|
| IAM: users, groups, roles, STS, federación | [[IAM]] | ✅ fuerte |
| Evaluación de policies, tipos, boundaries | [[iam-policy-evaluation]] | ✅ fuerte |
| ARNs y referencias a recursos | [[arn]] | ✅ |
| Root user, MFA, buenas prácticas de cuenta | [[aws-account]] | ✅ |
| KMS, [[envelope-encryption\|envelope encryption]], key policies | [[KMS]] | ✅ fuerte |
| Cifrado de datos en S3 (SSE-*, client-side) | [[s3-encryption]] | ✅ fuerte |
| Seguridad de S3: bucket policies, BPA, presigned, Object Lock | [[S3]] | ✅ fuerte |
| SCPs como techo organizacional | [[Organizations]] | ✅ |
| Responsabilidad del cliente vs AWS | [[shared-responsibility-model]] | ✅ |
| **Credenciales dentro de una instancia: IMDSv1 vs IMDSv2** | [[ec2-instance-metadata]] | ✅ fuerte |
| Roles asumidos por servicios (instance profile, execution role) | [[instance-profile]], [[execution-role]] | ✅ |
| Cifrado de volúmenes y snapshots (DEK por volumen) | [[EBS]] | ✅ |
| Acceso privado a servicios y [[endpoint-policy\|endpoint policies]] | [[vpc-endpoints]] | ✅ |
| Filtrado de red: SG vs NACL, diagnóstico con flow logs | [[security-groups-vs-nacls]], [[vpc-flow-logs]] | ✅ |

## Servicios más importantes para este dominio

[[IAM]], [[KMS]], [[S3]], [[Organizations]] — todos con página.

> 🔐 **El gotcha de seguridad más rentable del ingest de EC2:** si una app web en una instancia tiene un **SSRF**, con **IMDSv1** alcanza para robar las credenciales temporales del IAM role. La mitigación esperada en el examen es **exigir IMDSv2** (`HttpTokens: required`), que obliga a un `PUT` con header — algo que un SSRF no puede armar. Ver [[ec2-instance-metadata]].

> ⚠️ **Huecos pendientes de ingest**: **Cognito** (User Pools vs Identity Pools — muy preguntado en DVA), **Secrets Manager**, **SSM Parameter Store**, **ACM** (certificados). El dominio con mejor cobertura actual de la wiki.
