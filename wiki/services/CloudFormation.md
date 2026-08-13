---
title: CloudFormation
category: service
tags: [cloudformation, iac, templates, stacks, yaml]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.10 CloudFormation — Basics.md"]
updated: 2026-07-23
---

# CloudFormation

## ¿Qué es?

**Infrastructure as Code**: crear, actualizar y eliminar infraestructura AWS mediante **templates** en YAML o JSON, en vez de configurar a mano. La template aplicada se convierte en un **stack** de recursos.

## Casos de uso

- Infraestructura reproducible entre entornos y regiones.
- Automatizar despliegues (pilar del dominio IaC de DOP-C02 y de Deployment en DVA-C02).

## Características clave

### Por qué IaC y no clickear la consola

- **Repetible**: el mismo template levanta dev/staging/prod idénticos.
- **Versionable**: la infra vive en Git (historial, PRs, code review).
- **Documentado por definición**: el template *es* la documentación.
- **Destruible**: borrar el stack borra todos sus recursos, sin huérfanos cobrando.
- **Reversible**: si un update falla, CFN hace **rollback automático**.

### Secciones de una template

| Sección | ¿Obligatoria? | Qué hace |
|---|---|---|
| **Resources** | ✅ **la única** | Recursos a crear/actualizar/eliminar |
| Description | No | Texto libre (si existe, va inmediatamente después de `AWSTemplateFormatVersion`) |
| AWSTemplateFormatVersion | No | Versión del formato |
| Parameters | No | Inputs del usuario al aplicar (ej: instance size) |
| Mappings / Conditions / Outputs… | No | Se ven más adelante en el curso |

### CloudFormation vs. otras IaC (pregunta de comparación)

| Herramienta | Alcance | Lenguaje | State |
|---|---|---|---|
| **CloudFormation** | Solo AWS, declarativo, gratis (pagás recursos) | YAML/JSON | Lo maneja AWS |
| **Terraform** | Multi-cloud | HCL | **Vos** manejás el state file |
| **CDK** | Genera CloudFormation por debajo | TypeScript/Python/… | Vía CFN |
| **SAM** | Extensión de CFN para **serverless** (Lambda/API GW/DynamoDB) | YAML | Vía CFN |

- "Solo AWS, declarativo, sin manejar state" → **CloudFormation**.
- "Misma herramienta para AWS y Azure/GCP" → **Terraform**.

## Integración con otros servicios

- Todos — cualquier recurso AWS se puede declarar en una template.
- [[IAM]] — el stack opera con permisos (roles de servicio).
- Pilar del dominio [[dva-deployment|Deployment]] del examen DVA-C02 (24% del examen).

## Gotchas y trampas del examen

- "¿Qué sección es obligatoria?" → **Resources**, solo esa.
- `Description` debe ir justo después de `AWSTemplateFormatVersion` si ambas existen.
- Borrar el stack borra sus recursos (salvo políticas de retención).

> ⚠️ Página inicial — se ampliará cuando el curso llegue a CloudFormation en profundidad (change sets, drift, StackSets, nested stacks).

## Demos del curso

- [Simple Automation With CloudFormation](https://learn.cantrill.io/courses/1101194/lectures/25216284)
