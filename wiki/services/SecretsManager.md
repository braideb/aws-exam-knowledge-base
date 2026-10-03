---
title: AWS Secrets Manager
category: service
tags: [secrets-manager, secretos, rotacion, lambda, kms, rds, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.16 AWS Secrets Manager.md", "raw/doc oficial/Managed rotation for AWS Secrets Manager secrets - AWS Secrets Manager.md", "raw/doc oficial/Set up single user rotation for AWS Secrets Manager - AWS Secrets Manager.md", "raw/doc oficial/Set up alternating users rotation for AWS Secrets Manager - AWS Secrets Manager.md", "raw/doc oficial/Multi-user secrets rotation for Amazon RDS.md", "raw/doc oficial/Password management with Amazon RDS and AWS Secrets Manager - Amazon Relational Database Service.md", "raw/doc oficial/WKLD.03 Use ephemeral secrets or a secrets-management service - AWS Prescriptive Guidance.md", "raw/doc oficial/AWS Systems Manager Parameter Store - AWS Systems Manager.md"]
updated: 2026-10-03
---

# AWS Secrets Manager

## ¿Qué es?

Servicio para guardar **secretos** (contraseñas de bases de datos, API keys, tokens, certificados) **cifrados con [[KMS]]**, con acceso controlado por [[IAM]] y pensado para que las aplicaciones los **lean en runtime** por SDK/API. Su diferencia con [[SSMParameterStore|Parameter Store]] es la **rotación automática** (con Lambda o gestionada por el servicio dueño de la credencial) y la **integración nativa** con [[RDS]], [[Aurora]], Redshift y DocumentDB.

## Casos de uso

- **Credenciales de bases de datos** que tienen que **rotar** sin tocar la aplicación.
- Sacar contraseñas del código, del [[ec2-bootstrapping|user data]] y de las variables de entorno.
- Secretos en formato documento (varios pares clave/valor) o de **más de 4 KB** (certificados), que no entran en Parameter Store Standard ¹.
- Credenciales que usan **RDS Proxy** y la **RDS Data API** para conectarse a la base.

## Características clave

### Cómo lo usa una aplicación (ejemplo del curso: Catagram)

```
App (SDK de Secrets Manager) ──IAM role──► Secrets Manager ──► devuelve credenciales
App ──credenciales──► Base de datos

Rotación:
Secrets Manager ──invoca cada N días──► Lambda (execution role)
                                           ├── genera password nueva y la guarda en el secreto
                                           └── la cambia en la base (RDS)
```

Mientras la app **lea el secreto cada vez** (o lo refresque), siempre tiene la versión vigente.

### Rotación

- **Con Lambda**: Secrets Manager invoca una función (AWS provee plantillas para los motores de RDS) que cambia la contraseña **en el secreto y en la base**. La Lambda usa un **[[execution-role]]**.
- **Managed rotation** ¹: el servicio dueño rota solo, **sin Lambda propia**. Por ejemplo, RDS y Aurora con la contraseña del **master** (`--manage-master-user-password`: rota cada **7 días** por defecto), además de DocumentDB y Redshift. Rota en ~1 minuto.
- **Estrategias de rotación** ¹:

| Estrategia | Cómo funciona | Contra |
|---|---|---|
| **Single user** | Cambia la contraseña del mismo usuario | Puede haber fallos de login durante la rotación |
| **Alternating users** | Clona el usuario (`_clone`) y alterna cuál se actualiza; necesita un **secreto admin** | Más alta disponibilidad, más configuración |

- La Lambda de rotación corre **en la VPC** de la base. Para llamar a la API de Secrets Manager necesita salida: un **interface endpoint** de Secrets Manager o un NAT ([[vpc-endpoints]]) ¹.
- Se puede rotar tan seguido como **cada 4 horas** ¹.

### Seguridad

- Cifrado en reposo **siempre** con KMS (AWS managed o customer managed key).
- **Separación de funciones**: leer un secreto exige permisos en **Secrets Manager y en KMS** (`kms:Decrypt` si es una customer managed key).
- Acceso por [[IAM]] (roles, no access keys) y auditoría con CloudTrail.

### Costo

De pago: **por secreto por mes** y **por cada 10.000 llamadas** a la API. Parameter Store Standard es gratis: esa es la otra mitad de la decisión de examen.

### Relación con Parameter Store

- Parameter Store puede **leer secretos de Secrets Manager** con el prefijo `/aws/reference/secretsmanager/<secreto>`, así el código usa una sola API.
- La propia doc de Parameter Store recomienda Secrets Manager para credenciales y deja SecureString para configuración sensible ¹.

→ Comparación completa: [[parameter-store-vs-secrets-manager]].

## Integración con otros servicios

- [[RDS]] / [[Aurora]]: rotación nativa, contraseña del master gestionada, credenciales de **RDS Proxy** y de la **Data API**.
- [[KMS]]: cifrado de cada secreto.
- [[IAM]]: el rol de la app (instance role, [[task-role]], execution role de Lambda) lee el secreto.
- [[ECS]]: el *task execution role* inyecta secretos en la task definition.
- [[SSMParameterStore]]: referencia `/aws/reference/secretsmanager/`.
- [[vpc-endpoints]]: acceso privado desde la VPC (rotación, apps sin internet).

## Gotchas y trampas del examen

- **Palabras clave → Secrets Manager**: *secreto* + **rotación** (+ RDS). Sin rotación y con el costo como prioridad → Parameter Store.
- *"Rotar la contraseña de RDS cada 30 días sin cambiar la app"* → Secrets Manager con rotación (o la contraseña del master gestionada por RDS).
- *"Durante la rotación algunas conexiones fallan"* → estrategia **alternating users**.
- *"La rotación falla con timeout y la base está en subnets privadas"* → la Lambda de rotación no llega a la API de Secrets Manager: falta el **VPC endpoint** o el NAT.
- *"AccessDenied al leer el secreto aunque la policy de Secrets Manager lo permite"* → falta permiso sobre la **KMS key** (customer managed).
- Ningún secreto va en el **user data** ni en el código.

¹ Complemento de la doc oficial (clippings en `sources`), no de las notas del curso.

> 📖 Lectura profunda: [[10.16 AWS Secrets Manager]] · [[09.04 SSM Parameter Store]]
