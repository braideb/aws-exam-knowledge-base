---
title: CloudFormation — Anatomía de una template portable
category: concept
tags: [cloudformation, iac, parameters, mappings, conditions, outputs]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/12 CloudFormation/12.02 Template y Pseudo Parameters.md", "raw/notas curso mejorado/12 CloudFormation/12.04 Mappings.md", "raw/notas curso mejorado/12 CloudFormation/12.05 Outputs.md", "raw/notas curso mejorado/12 CloudFormation/12.06 Conditions.md", "raw/notas curso mejorado/12 CloudFormation/12.07 DependsOn.md"]
updated: 2026-10-10
---

# CloudFormation — Anatomía de una template portable

## Definición

Una **template portable** es la que no tiene valores atados a mano y por eso se puede aplicar una y otra vez en cualquier cuenta y región. Se logra combinando las secciones de abajo con las [[cloudformation-intrinsic-functions|funciones intrínsecas]]. Esta página recorre las piezas que **alimentan** los recursos con valores calculados en despliegue; la mecánica de cálculo está en la página de funciones.

## Cómo aplica en AWS

### Parameters y pseudo parameters

Ambos meten valores desde afuera; la diferencia es **quién** los provee.

| | **Template parameters** | **Pseudo parameters** |
|---|---|---|
| Quién puebla | Un humano o proceso al crear/actualizar la stack (consola/CLI/API) | **AWS**, automáticamente |
| Se declaran | Sí, en `Parameters` | **No** — existen siempre |
| Ejemplos | `InstanceType`, `EnvType` | `AWS::Region`, `AWS::AccountId`, `AWS::StackName`, `AWS::StackId` |

Config de un template parameter: **Default**, **AllowedValues** (lista cerrada), **constraints** (longitud/regex), **NoEcho** (enmascara en consola/CLI/API) y **Type** (simple, o *AWS-aware* como "elegí una VPC/subnet de esta lista", que la consola rellena según región y cuenta).

> **Para el examen:** `NoEcho` **no** es para secretos. Un secreto de verdad va en [[SecretsManager|Secrets Manager]] o [[SSMParameterStore|Parameter Store (SecureString)]] y se referencia desde la template.

### Mappings

Un **diccionario** dentro de la template: una clave (o top-level + second-level key) devuelve valores con nombre. Se lee con `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`. El uso clásico es un `RegionMap` de AMI IDs indexado por región y arquitectura. Mejoran la **portabilidad**: guardan los datos que hacen que la template se comporte distinto según la entrada, sin tocar los recursos.

### Conditions

Se evalúan a **true/false** **antes** que los recursos (un `if` al principio del despliegue). Se arman con las funciones lógicas (`Fn::Equals`, `Fn::And`, `Fn::Or`, `Fn::Not`, `Fn::If`) y se asocian a un recurso con `Condition:`: si da true, el recurso se crea; si no, no. Patrón típico: `IsProd: !Equals [ !Ref EnvType, 'prod' ]` para crear instancias/Elastic IPs extra solo en prod.

### Outputs

Opcionales. Exponen valores (ej. la URL final) en consola/CLI, y —clave— quedan **accesibles desde una stack padre** cuando hay nesting. Un output se puede **exportar** (`Export`), que es lo que habilita las [[nested-stacks-vs-cross-stack-references|cross-stack references]].

### DependsOn — dependencias implícitas vs explícitas

CFN paraleliza lo que puede, infiriendo el orden desde las **referencias** (`!Ref` → **dependencia implícita**). Cuando hay una dependencia real que no sale de ningún `!Ref`, la declarás con **`DependsOn`** (explícita): "B no arranca hasta que A esté `CREATE_COMPLETE`".

> Ejemplo canónico: una Elastic IP necesita un Internet Gateway ya **adjunto** a la VPC. Esa relación no surge de un `!Ref`, así que la EIP lleva `DependsOn: InternetGatewayAttachment`. Sin eso, la creación puede fallar.

## Patrones comunes

| Querés… | Pieza |
|---|---|
| Que el usuario elija de una lista válida | parameter con **AllowedValues** |
| La región/cuenta/stack actual sin hardcodear | **pseudo parameter** |
| AMI correcta por región | **Mappings** (`RegionMap`) o parámetro público de [[SSMParameterStore]] |
| Crear recursos solo en prod | **Condition** + `Condition:` en el recurso |
| Devolver la URL o un ID | **Output** (y `Export` si otra stack lo consume) |
| Forzar un orden que CFN no ve | **DependsOn** |

## Preguntas de examen frecuentes

- *"¿Cómo paso una contraseña a la template de forma segura?"* → **no** `NoEcho`; Secrets Manager / Parameter Store SecureString.
- *"La AMI no existe en la otra región"* → Mappings por región, o un parámetro público de SSM como `Default`.
- *"El recurso B se crea antes que A y falla"* → si no hay `!Ref` que lo ordene, **DependsOn**.
- *"Quiero la misma template para dev y prod con menos recursos en dev"* → **Conditions** sobre un parameter de entorno.

> 📖 Lectura profunda: [[12.02 Template y Pseudo Parameters]] · [[12.04 Mappings]] · [[12.05 Outputs]] · [[12.06 Conditions]] · [[12.07 DependsOn]]

## Ver también

[[wiki/services/CloudFormation]] · [[cloudformation-intrinsic-functions]] · [[nested-stacks-vs-cross-stack-references]] · [[SSMParameterStore]]
