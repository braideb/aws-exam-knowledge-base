---
title: Parameter Store vs Secrets Manager
category: comparison
tags: [ssm, parameter-store, secrets-manager, secretos, rotacion, kms]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/09 Advanced EC2/09.04 SSM Parameter Store.md"]
updated: 2026-09-30
---

# Parameter Store vs Secrets Manager

Los dos guardan secretos cifrados con [[KMS]] y se leen con permisos de [[IAM]]. La diferencia que el examen pregunta casi siempre es la **rotación automática**.

## Tabla comparativa

| | [[SSMParameterStore\|SSM Parameter Store]] | Secrets Manager |
|---|---|---|
| Para qué | **Configuración** y secretos simples | **Secretos** que tienen que rotar |
| Tipos | String, StringList, SecureString | Secretos (texto o JSON) |
| Cifrado | Opcional (solo **SecureString** usa KMS) | **Siempre** con KMS |
| **Rotación automática** | ❌ No | ✅ Sí, con **Lambda**, e integración nativa con **RDS** |
| Costo | **Standard gratis** (10.000 parámetros, 4 KB); Advanced de pago | De pago por secreto y por llamadas ¹ |
| Jerarquías y versionado | Sí (`get-parameters-by-path`) | Versionado por etapas ¹ |
| Integración entre ellos | Puede leer secretos de Secrets Manager con `/aws/reference/secretsmanager/` | — |

¹ Complemento: no viene de las notas del curso ni de un clipping curado.

## Cuándo usar X vs Y

| Escenario | Respuesta |
|---|---|
| Rotar automáticamente la contraseña de una base de datos RDS | **Secrets Manager** |
| Guardar connection strings, hostnames o feature flags | **Parameter Store** (String) |
| Guardar una contraseña que no rota, al menor costo | **Parameter Store** (SecureString, tier Standard) |
| Leer toda la configuración de una app de una vez | **Parameter Store** (`get-parameters-by-path`) |
| Resolver el ID de la AMI más reciente | **Parameter Store** (parámetros públicos) |

## Trampa típica del examen

- Cuando el enunciado menciona **rotación automática**, la respuesta es Secrets Manager, aunque Parameter Store "también guarde secretos cifrados".
- Cuando dice **"la opción más barata"** para configuración o secretos sin rotación, la respuesta es Parameter Store.
- Ninguno de los dos va en el **user data** ni en el código. Se leen en runtime con un role ([[ec2-bootstrapping]], [[instance-profile]]).

> 📖 Lectura profunda: [[09.04 SSM Parameter Store]]
