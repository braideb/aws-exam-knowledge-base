---
title: CloudFormation — Funciones intrínsecas
category: concept
tags: [cloudformation, iac, intrinsic-functions, ref, getatt, sub]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/12 CloudFormation/12.03 Intrinsic Functions.md"]
updated: 2026-10-10
---

# CloudFormation — Funciones intrínsecas

## Definición

Las **funciones intrínsecas** (`Fn::*`, o su forma corta `!`) son lo que convierte una template de documento estático en una que **calcula valores en el despliegue**. Son la herramienta central para pasar de una template con valores a mano a una [[cloudformation-template|portable]].

## Cómo aplica en AWS

| Función | Qué hace | Nota de examen |
|---|---|---|
| **`!Ref`** | El **valor principal**: de un parameter, su valor; de un recurso lógico, su **ID físico** | — |
| **`!GetAtt`** | Un **atributo puntual**: `!GetAtt Logical.Attribute` (ej. `PublicIp`, `DNSName`) | Ref = identificador; GetAtt = dato específico |
| **`!Join`** / **`!Split`** | Pegar una lista en string / partir un string en lista | Armar una URL: `!Join ['', ['http://', !GetAtt Instance.DNSName]]` |
| **`!GetAZs`** / **`!Select`** | Lista de AZs de una región / elegir un elemento por índice | Índice **base 0**; GetAZs devuelve las AZs donde la **default VPC** tiene subnets |
| **`!Sub`** | Sustituye `${...}` con valor de runtime | **No permite autorreferencias** (huevo/gallina en user data) |
| **`!Base64`** | Texto plano → base64 | El user data de EC2 lo exige |
| **`!Cidr`** | Calcula rangos [[cidr\|CIDR]] de subnets dentro del de la VPC | Se combina con `!Select` por subnet |
| **`!FindInMap`** | Lee un Mappings (`[Map, TopKey, SecondKey]`) | Ver [[cloudformation-template]] |
| **`!ImportValue`** | Trae un output **exportado** por otra stack | La base de las [[nested-stacks-vs-cross-stack-references\|cross-stack references]] |

### Ref vs GetAtt

Todo parameter y recurso lógico tiene un **valor principal**. Para una instancia EC2 ese valor es su **ID físico** (`i-123…`), así que `!Ref Instance` lo devuelve; `!GetAtt Instance.PublicIp` en cambio devuelve un atributo secundario. En una frase: **Ref da el identificador principal; GetAtt da un dato puntual.**

### Dentro de `!Sub`

Tres formatos: `${Parameter}`, `${LogicalResource}` (equivale a `!Ref`) y `${LogicalResource.Attribute}` (equivale a `!GetAtt`). Lo que **no** podés es referenciar el propio recurso que se está creando: el user data se inyecta *antes* de que exista su `InstanceId`.

## Patrones comunes

- **Portabilidad de AZ:** `AvailabilityZone: !Select [ 0, !GetAZs '' ]` = "la primera AZ de la región actual", sin nombrarla.
- **Subnets automáticas:** `!Select [ "0", !Cidr [ !GetAtt VPC.CidrBlock, 16, 12 ]]` reparte el CIDR de la VPC en subnets sin calcularlas a mano.
- **URL de salida:** `!Join` + `!GetAtt ....DNSName` en un Output.

## Preguntas de examen frecuentes

- *"Necesito el ID físico del recurso"* → **`!Ref`**. *"Necesito su IP/DNS/[[arn|ARN]]"* → **`!GetAtt`**.
- *"¿Por qué `!GetAZs` no me devuelve todas las AZs?"* → devuelve las de la **default VPC**; si le borraste subnets, la lista queda corta.
- *"El `!Sub` del user data falla referenciando su propio InstanceId"* → **autorreferencia inválida**.
- *"Pasar el ID de una VPC creada en otra stack"* → `!ImportValue` sobre su export (misma región/cuenta).

> 📖 Lectura profunda: [[12.03 Intrinsic Functions]]

## Ver también

[[wiki/services/CloudFormation]] · [[cloudformation-template]] · [[nested-stacks-vs-cross-stack-references]] · [[VPC]] · [[cidr]]
