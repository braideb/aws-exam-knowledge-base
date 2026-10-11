---
title: CloudFormation
category: service
tags: [cloudformation, iac, templates, stacks, yaml, deployment]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.10 CloudFormation — Basics.md", "raw/notas curso mejorado/12 CloudFormation/12.01 Physical and Logical Resources.md", "raw/notas curso mejorado/12 CloudFormation/12.14 StackSets.md", "raw/notas curso mejorado/12 CloudFormation/12.15 DeletionPolicy.md", "raw/notas curso mejorado/12 CloudFormation/12.16 Stack Roles.md", "raw/notas curso mejorado/12 CloudFormation/12.17 Change Sets.md", "raw/notas curso mejorado/12 CloudFormation/12.18 Custom Resources.md"]
updated: 2026-10-10
---

# CloudFormation

> **Hub.** Esta página es el mapa del tema. Los tres clusters densos viven en páginas propias: [[cloudformation-template|anatomía de una template portable]], [[cloudformation-intrinsic-functions|funciones intrínsecas]] y [[cfn-init-and-signals|cfn-init y señales]]. La decisión multi-stack, en [[nested-stacks-vs-cross-stack-references]].

## ¿Qué es?

**Infrastructure as Code**: crear, actualizar y eliminar infraestructura AWS mediante **templates** en YAML o JSON, en vez de configurar a mano. En la template declarás el **qué** (recursos lógicos), no el **cómo**; CloudFormation hace las llamadas a la API, en orden y con reintentos. Una template aplicada crea una **stack**, que materializa cada recurso lógico en un **recurso físico** real y mantiene ambos sincronizados.

> **La template es el plano; la stack es la construcción.** Como CFN sabe qué recursos físicos creó, borrar la stack los limpia a todos. *(Detalle: [[12.01 Physical and Logical Resources]].)*

## Casos de uso

- Infraestructura reproducible y versionable entre entornos, regiones y cuentas.
- Automatizar despliegues — pilar del dominio [[dva-deployment|Deployment]] (24% del DVA-C02) y de IaC en DOP-C02.
- Repartir infraestructura masiva a muchas cuentas/regiones con **StackSets**.

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
| Parameters / Mappings / Conditions / Outputs | No | La maquinaria de portabilidad → [[cloudformation-template]] |

### CloudFormation vs. otras IaC

| Herramienta | Alcance | Lenguaje | State |
|---|---|---|---|
| **CloudFormation** | Solo AWS, declarativo, gratis (pagás recursos) | YAML/JSON | Lo maneja AWS |
| **Terraform** | Multi-cloud | HCL | **Vos** manejás el state file |
| **CDK** | Genera CloudFormation por debajo | TypeScript/Python/… | Vía CFN |
| **SAM** | Extensión de CFN para **[[serverless\|serverless]]** (Lambda/API GW/DynamoDB) | YAML | Vía CFN |

- "Solo AWS, declarativo, sin manejar state" → **CloudFormation**.
- "Misma herramienta para AWS y Azure/GCP" → **Terraform**.

### Escribir la template (portabilidad)

El objetivo casi siempre es una **template portable**: sin valores atados a mano, reutilizable en cualquier cuenta/región. Lo cubre [[cloudformation-template]] (parameters, pseudo parameters, Mappings, Conditions, Outputs, DependsOn) apoyándose en las [[cloudformation-intrinsic-functions|funciones intrínsecas]].

### Configurar EC2 y señalizar el fin del bootstrapping

CFN cree que un recurso terminó (`CREATE_COMPLETE`) apenas AWS lo crea, aunque el **bootstrapping** siga corriendo o haya fallado. Para cerrar esa brecha están **CreationPolicy**, **WaitConditions**, **cfn-signal**, **cfn-init** (desired state) y **cfn-hup** → [[cfn-init-and-signals]].

### Arquitecturas multi-stack

Una stack única topa con el límite de **500 recursos** y con el aislamiento (`!Ref` no cruza el borde de la stack). Las dos salidas —**nested stacks** (reutilizar la *template*) y **cross-stack references** (reutilizar la *stack*)— se comparan en [[nested-stacks-vs-cross-stack-references]].

### StackSets

Despliegan y administran infraestructura **a través de múltiples cuentas y regiones desde un solo lugar** (la *cuenta de administrador*). El StackSet es un contenedor de **stack instances** (referencias a stacks en cada cuenta/región de destino). Los permisos vienen por **self-managed roles** o **service-managed roles** (con [[Organizations]], se crean solos). Términos: *concurrent accounts*, *failure tolerance*, *retain stacks*. Uso típico: activar AWS Config o repartir [[IAM]] roles cross-account en toda la organización. *(Detalle: [[12.14 StackSets]].)*

### DeletionPolicy

Atributo **por recurso** que decide qué pasa al eliminarlo:

| Valor | Comportamiento |
|---|---|
| **Delete** (default) | Lo elimina con el recurso lógico |
| **Retain** | No lo elimina; sobrevive a la stack |
| **Snapshot** | Toma un snapshot antes de borrar — solo **EBS, ElastiCache, Neptune, RDS, Redshift** |

> **Trampa:** la DeletionPolicy **solo aplica a eliminaciones, no a *replacements***. Un cambio que fuerza replacement descarta el recurso viejo y **pierde los datos** igual. *(Detalle: [[12.15 DeletionPolicy]].)*

### Stack Roles (PassRole)

Normalmente CFN usa los permisos de **quien crea la stack**. Con **stack roles**, le [[pass-role|pasás]] un [[IAM]] role y CFN **lo asume** para actuar. Así el operador solo necesita permisos sobre la stack + `iam:PassRole`, sin poder crear esos recursos por fuera de CFN ni asumir el role directamente. Separación de responsabilidades limpia. *(Detalle: [[12.16 Stack Roles]].)*

### Change Sets

Previsualizan qué le haría a los recursos una template nueva **antes** de ejecutar. Clasifican cada cambio en **no interruption**, **some interruption** (ej. reinicio) o **replacement** (crea copia nueva y descarta la vieja → riesgo de pérdida de datos). Podés crear varios, descartar y ejecutar uno. *(Detalle: [[12.17 Change Sets]].)*

### Custom Resources

Lógica propia para lo que CFN no soporta de fábrica o vive fuera de AWS. Arquitectura de **webhook**: CFN manda el `event` (Create/Update/Delete) a una **Lambda** o **SNS topic**, y el código responde por la **ResponseURL** ([[presigned-url|URL prefirmada]]) con `SUCCESS`/`FAILED`. Caso clásico: vaciar un bucket [[S3]] antes de borrarlo (si no, falla con *"bucket not empty"*). *(Detalle: [[12.18 Custom Resources]].)*

## Integración con otros servicios

- **Todos** — cualquier recurso AWS se declara en una template.
- [[IAM]] — stack roles vía [[pass-role|PassRole]]; StackSets reparte roles cross-account.
- [[SSMParameterStore]] — un parámetro **público** de SSM da la última AMI por región como `Default`.
- [[EC2]] / [[ec2-bootstrapping]] — user data vs [[cfn-init-and-signals|cfn-init]].
- Pilar de [[dva-deployment|Deployment]] (24% del DVA-C02).

## Gotchas y trampas del examen

- "¿Qué sección es obligatoria?" → **Resources**, solo esa.
- `Description` va justo después de `AWSTemplateFormatVersion` si ambas existen.
- **Una template no es una stack**: reutilizar template → nested stacks; reutilizar la stack real → cross-stack references.
- DeletionPolicy no protege contra **replacement**.
- `CREATE_COMPLETE` ≠ "la app anda" → hace falta **cfn-signal**.
- StackSets y cross-stack references **no** cruzan cuenta/región solos: StackSets lo hace con roles; las cross-stack references **no cruzan** región ni cuenta.

## Demos del curso

- [Simple Automation With CloudFormation](https://learn.cantrill.io/courses/1101194/lectures/25216284)
- Non-portable template: [parte 1](https://learn.cantrill.io/courses/3028179/lectures/66797628) · [parte 2](https://learn.cantrill.io/courses/3028179/lectures/66797629)
- [Template v2 Portable](https://learn.cantrill.io/courses/1101194/lectures/28326940)
- WaitCondition / cfn-signal / cfn-init / cfn-hup: [parte 1](https://learn.cantrill.io/courses/1101194/lectures/28333069) · [parte 2](https://learn.cantrill.io/courses/1101194/lectures/30471883) — [[12.11 Demostración - cfn-signal, cfn-init y cfn-hup|comandos y templates]]
- [Change Sets](https://learn.cantrill.io/courses/1101194/lectures/28338687)
- Custom Resources: [parte 1](https://learn.cantrill.io/courses/1101194/lectures/28338903) · [parte 2](https://learn.cantrill.io/courses/1101194/lectures/29652962)

> 📖 Lectura profunda: [[12.00 CloudFormation — Índice|Módulo 12 completo (12.01–12.18)]] · [[01.10 CloudFormation — Basics]]

## Ver también

[[cloudformation-template]] · [[cloudformation-intrinsic-functions]] · [[cfn-init-and-signals]] · [[nested-stacks-vs-cross-stack-references]] · [[dva-deployment]]
