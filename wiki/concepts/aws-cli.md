---
title: AWS CLI y acceso programático
category: concept
tags: [cli, credenciales, profiles, acceso-programatico, sdk]
exam: [DVA-C02, DOP-C02, SAA-C03]
sources: ["raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.05 Demostración - AWS CLI y perfiles.md", "raw/notas curso mejorado/02 Fundamentos y cuenta AWS/02.04 IAM Access Keys.md"]
updated: 2026-09-28
---

# AWS CLI y acceso programático

## Definición

La **AWS CLI** (usar la **v2**) y los SDKs son las vías de acceso programático a AWS. No usan username+password: usan **access keys** ([[IAM]]) o [[temporary-credentials|credenciales temporales]] de roles/SSO. Es el task 2.1.3 de [[dva-development|DVA-C02]]: *configure programmatic access*.

## Configuración y profiles

```bash
aws configure                          # perfil default
aws configure --profile iamadmin-general   # named profile
```

Ambos piden: `Access Key ID`, `Secret Access Key`, `Default region`, `Default output format` (json/yaml/text/table).

**Usar un named profile** — si no se especifica, la CLI no encuentra credenciales:

```bash
aws s3 ls --profile iamadmin-general   # explícito por comando
export AWS_PROFILE=iamadmin-general    # o para toda la sesión
```

> ⚠️ `export AWS_PROFILE` es cómodo y es también **el error más peligroso de la CLI**: dejar exportado el perfil de **producción** y correr un comando destructivo creyendo que estás en desarrollo. Vale la pena mostrar el perfil activo en el prompt de la terminal.

Las credenciales viven en `~/.aws/credentials` (keys por perfil) y `~/.aws/config` (region, output, perfiles de rol/SSO).

## Cadena de credenciales (orden de precedencia)

El primero que aparece gana — explica la mayoría de los "usa las credenciales equivocadas":

1. Opciones del comando (`--profile`)
2. **Variables de entorno** (`AWS_ACCESS_KEY_ID`, `AWS_PROFILE`)
3. `~/.aws/credentials`
4. `~/.aws/config`
5. Credenciales de contenedor ([[task-role|ECS task role]])
6. **[[instance-profile|Instance profile]]** de [[EC2]] (rol vía IMDS)

## Patrones recomendados (examen)

- Dentro de AWS → **roles**, jamás access keys hardcodeadas ([[IAM]]).
- Un perfil puede **asumir un rol** automáticamente (`role_arn` + `source_profile` en `~/.aws/config`), incluso pidiendo MFA.
- Acceso humano moderno → SSO / credenciales temporales, no keys de largo plazo.
- **La CLI no exige MFA por sí sola** aunque el user lo tenga activado: hay que pedir credenciales temporales con `aws sts get-session-token --serial-number <arn-del-mfa> --token-code <código>` y exigirlo por policy con `aws:MultiFactorAuthPresent` (ver [[aws-account]]).

## Preguntas de examen frecuentes

- "La CLI dice *Unable to locate credentials*" → falta `--profile` o el perfil default no está configurado.
- "Configuré el archivo pero usa otras credenciales" → hay **variables de entorno** pisando la cadena (precedencia 2 > 3).
- "¿Cómo accede una app en EC2 sin keys?" → instance profile (paso 6 de la cadena, automático para CLI y SDKs).
- "Corrí el comando en la cuenta equivocada" → `AWS_PROFILE` exportado de una sesión anterior.
