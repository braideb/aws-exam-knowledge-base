---
title: Execution role
category: glossary
tags: [lambda, iam, roles, permisos]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.11 Lambda en una VPC.md"]
updated: 2026-10-03
---

# Execution role

> **En una línea:** el **IAM role que un servicio asume por vos** para ejecutar tu código — el caso más conocido es el de Lambda.

## Definición

Cuando Lambda invoca una función, asume ese role y le entrega a tu código [[temporary-credentials|credenciales temporales]]. Todo lo que la función haga contra AWS se autoriza con **sus** permisos, no con los de quien la invocó. Es el equivalente en [[serverless]] del [[instance-profile]] de EC2.

## Dónde aparece

- [[lambda-in-vpc]] — necesita `AWSLambdaVPCAccessExecutionRole` para crear y borrar sus ENIs
- [[IAM]] — escenarios de uso de roles
- [[XRay]] — el execution role de una Lambda necesita `xray:PutTraceSegments` para enviar traces
- También en: [[dva-development]] · [[dva-security]] · [[Organizations]] · [[ECS]] · [[task-role]] · [[SSMParameterStore]] · [[SecretsManager]]

## Dato de examen

- Para que una Lambda se conecte a una VPC, su execution role necesita `ec2:CreateNetworkInterface`, `ec2:DescribeNetworkInterfaces` y `ec2:DeleteNetworkInterface`. La policy gestionada que los trae es **`AWSLambdaVPCAccessExecutionRole`**; sin ellos la función falla al arrancar con un error de configuración.
- No confundir con la **resource policy** de la función, que dice *quién puede invocarla*.

## Ver también

[[instance-profile]] · [[task-role]] · [[temporary-credentials]] · [[trust-policy]]
