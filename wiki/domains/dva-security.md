---
title: "DVA-C02 · Dominio 2: Security"
category: domain
tags: [dva-c02, security, iam, cifrado, autenticacion]
exam: [DVA-C02]
sources: ["https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html"]
updated: 2026-10-03
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
| **Secret management** (Task 3): Parameter Store SecureString + KMS, `--with-decryption`, rotación con Secrets Manager (Lambda o *managed*, single vs alternating users) | [[SSMParameterStore]], [[SecretsManager]], [[parameter-store-vs-secrets-manager]] | ✅ curso + doc oficial |
| **Autenticación a la base sin contraseña**: IAM DB auth (token de 15 min, `rds-db:connect`, autentica pero no autoriza) | [[RDS]] | ✅ |
| **Cifrado de bases**: KMS al crear, réplica = mismo estado, snapshot → copia cifrada; [[transparent-data-encryption\|TDE]]; compartir snapshots cifrados | [[RDS]], [[KMS]] | ✅ |
| Secretos fuera del **user data** (se lee en texto plano desde el IMDS) | [[ec2-bootstrapping]] | ✅ |
| Cifrado de volúmenes y snapshots (DEK por volumen) | [[EBS]] | ✅ |
| Acceso privado a servicios y [[endpoint-policy\|endpoint policies]] | [[vpc-endpoints]] | ✅ |
| Filtrado de red: SG vs NACL, diagnóstico con flow logs | [[security-groups-vs-nacls]], [[vpc-flow-logs]] | ✅ |
| **Permisos de containers**: [[task-role\|task role]] vs task execution role vs container instance role; [[irsa\|IRSA / Pod Identity]] en EKS | [[ECS]], [[EKS]] | ✅ solo curso |
| Seguridad de images: repository policies, image scanning (Inspector), tag immutability | [[ECR]] | ✅ solo curso |

## Servicios más importantes para este dominio

[[IAM]], [[KMS]], [[S3]], [[Organizations]] — todos con página.

> 🔐 **El gotcha de seguridad del módulo 08:** en [[ECS]] hay **tres roles** y el examen los mezcla. El código de la app usa el **task role**. El pull de la image, el envío de logs y la lectura de secrets usa el **task execution role**. En EC2 mode, registrar la instancia en el cluster usa el **container instance role**. Nunca se le dan permisos a la app vía el rol de la instancia. En EKS la misma idea es IRSA / Pod Identity, en vez del role del node.

> 🔐 **El gotcha de seguridad más rentable del ingest de EC2:** si una app web en una instancia tiene un **[[ssrf|SSRF]]**, con **IMDSv1** alcanza para robar las credenciales temporales del IAM role. La mitigación esperada en el examen es **exigir IMDSv2** (`HttpTokens: required`), que obliga a un `PUT` con header — algo que un SSRF no puede armar. Ver [[ec2-instance-metadata]].

> 🔐 **El gotcha del módulo 09:** un **SecureString** necesita **dos** permisos para leerse descifrado: el de SSM sobre el parámetro y **`kms:Decrypt`** sobre la key. Si el enunciado pide **rotación automática**, la respuesta es Secrets Manager, no Parameter Store. Y ningún secreto va en el **user data**. Ver [[SSMParameterStore]] y [[parameter-store-vs-secrets-manager]].

> 🔐 **El gotcha del módulo 10:** IAM DB auth **autentica pero no autoriza**: los permisos dentro de la base siguen siendo del usuario local. El cifrado de RDS se elige **al crear** y no se activa después: snapshot → copia cifrada → restore. Y un snapshot cifrado con la **AWS managed key** no se puede compartir con otra cuenta. Ver [[RDS]] y [[SecretsManager]].

> ⚠️ **Huecos pendientes de ingest**: **Cognito** (User Pools vs Identity Pools — muy preguntado en DVA), **ACM** (certificados). *Secrets Manager salió de esta lista con el ingest del módulo 10.* El dominio con mejor cobertura actual de la wiki.
