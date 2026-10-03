---
title: Parameter Store vs Secrets Manager
category: comparison
tags: [ssm, parameter-store, secrets-manager, secretos, rotacion, kms]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/09 Advanced EC2/09.04 SSM Parameter Store.md", "raw/notas curso mejorado/10 Databases (SQL)/10.16 AWS Secrets Manager.md", "raw/doc oficial/AWS Systems Manager Parameter Store - AWS Systems Manager.md", "raw/doc oficial/WKLD.03 Use ephemeral secrets or a secrets-management service - AWS Prescriptive Guidance.md", "raw/doc oficial/Managed rotation for AWS Secrets Manager secrets - AWS Secrets Manager.md"]
updated: 2026-10-03
---

# Parameter Store vs Secrets Manager

Los dos guardan secretos cifrados con [[KMS]] y se leen con permisos de [[IAM]]. La diferencia que el examen pregunta casi siempre es la **rotación automática**.

## Tabla comparativa

| | [[SSMParameterStore\|SSM Parameter Store]] | [[SecretsManager\|Secrets Manager]] |
|---|---|---|
| Para qué | **Configuración** y secretos simples | **Secretos** (credenciales, API keys, certificados) |
| Tipos | String, StringList, SecureString | Secretos (texto o JSON con varios pares clave/valor) |
| Cifrado | Opcional (solo **SecureString** usa KMS) | **Siempre** con KMS |
| **Rotación automática** | ❌ No | ✅ Sí: con **Lambda** o *managed rotation* (RDS, Aurora, Redshift, DocumentDB) |
| Tamaño | Standard **4 KB**, Advanced **8 KB** | Secretos más grandes (certificados) ¹ |
| Costo | **Standard gratis** (10.000 parámetros); Advanced y high-throughput de pago | Por **secreto/mes** y por **cada 10.000 llamadas** a la API |
| Versionado | Últimas **100 versiones** ¹ | Versiones con **staging labels** ¹ |
| Integración entre ellos | Lee secretos de Secrets Manager con `/aws/reference/secretsmanager/` | — |

¹ De la doc oficial (clippings en `sources`).

La propia doc de Parameter Store recomienda Secrets Manager para **credenciales de bases de datos, API keys y tokens**, y deja SecureString para configuración sensible. Para configuración que cambia seguido y necesita despliegue gradual con rollback, la doc menciona un tercero: **AWS AppConfig**.

## Cuándo usar X vs Y

| Escenario | Respuesta |
|---|---|
| Rotar automáticamente la contraseña de una base de datos RDS | **Secrets Manager** |
| Guardar connection strings, hostnames o feature flags | **Parameter Store** (String) |
| Guardar una contraseña que no rota, al menor costo | **Parameter Store** (SecureString, tier Standard) |
| Secreto de más de 4 KB o con varios pares clave/valor | **Secrets Manager** ¹ |
| Leer toda la configuración de una app de una vez | **Parameter Store** (`get-parameters-by-path`) |
| Resolver el ID de la AMI más reciente | **Parameter Store** (parámetros públicos) |

## Trampa típica del examen

- Cuando el enunciado menciona **rotación automática**, la respuesta es Secrets Manager, aunque Parameter Store "también guarde secretos cifrados".
- Cuando dice **"la opción más barata"** para configuración o secretos sin rotación, la respuesta es Parameter Store.
- Ninguno de los dos va en el **user data** ni en el código. Se leen en runtime con un role ([[ec2-bootstrapping]], [[instance-profile]]).

> 📖 Lectura profunda: [[09.04 SSM Parameter Store]] · [[10.16 AWS Secrets Manager]]
