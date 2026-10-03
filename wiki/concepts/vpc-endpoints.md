---
title: VPC Endpoints
category: concept
tags: [vpc, networking, endpoints, privatelink, s3, dynamodb]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.09 VPC Endpoints.md", "raw/doc oficial/Set up alternating users rotation for AWS Secrets Manager - AWS Secrets Manager.md"]
updated: 2026-10-03
---

# VPC Endpoints

> ⚠️ **Fuente pendiente de revisión:** la sección del curso [[05.09 VPC Endpoints]] está marcada por el humano como *"(Pendiente a revisar)"* (2026-09-23) y esta página todavía **no se contrastó con doc oficial** (no hay clippings del tema en `raw/doc oficial/`). Tratar los datos finos con cautela hasta cerrar esa revisión.

## Definición

Un **VPC endpoint** conecta tu [[VPC]] con servicios de AWS de forma **privada**: sin Internet Gateway, sin NAT, sin VPN ni Direct Connect. El tráfico nunca sale a la internet pública, se queda dentro de la red de AWS.

El problema que resuelve: servicios como [[S3]], DynamoDB o SQS viven en la **AWS Public Zone**, no dentro de tu VPC. Sin endpoints, una instancia en una subnet privada solo llega a ellos saliendo por un NAT Gateway hacia internet — un rodeo que cuesta plata y expone tráfico que no hacía falta exponer.

## Cómo aplica en AWS

Hay **dos tipos**, y la diferencia está desarrollada en [[gateway-vs-interface-endpoint]].

### Gateway Endpoints

- Solo para **S3 y DynamoDB**.
- Funcionan agregando una **ruta** en la route table de la subnet: el destino es una [[prefix-list|prefix list]] (`pl-xxxx`) que representa los rangos del servicio, y el target es el propio endpoint (`vpce-xxxx`). **No son un dispositivo dentro de la subnet.**
- **Sin costo** adicional.
- Se controlan con una [[endpoint-policy|endpoint policy]].
- **No** accesibles desde fuera de la VPC (no sirven desde on-premises).

### Interface Endpoints

- Para **casi todos los demás servicios** (SQS, SNS, Kinesis, API Gateway, CloudWatch, Systems Manager…) y también para S3.
- Basados en [[privatelink|AWS PrivateLink]]: colocan una [[eni|ENI]] con IP privada dentro de una subnet que elegís (una por AZ para alta disponibilidad).
- Como son una ENI, se protegen con **security groups**.
- Tienen **costo por hora y por GB** procesado.
- Usan **DNS privado**: al activarlo, el nombre normal del servicio (`sqs.us-east-1.amazonaws.com`) resuelve a la IP privada del endpoint.
- **Sí** accesibles desde on-premises vía VPN o Direct Connect.

## Patrones comunes

### Por qué la ruta del gateway endpoint gana

```
Route table de la subnet privada
  10.16.0.0/16   → local
  pl-xxxx (S3)   → vpce-xxxx     ← más específica, gana
  0.0.0.0/0      → nat-xxxx
```

Al ser una ruta más específica que la default, le gana por [[longest-prefix-match|longest prefix match]]. Por eso el tráfico hacia S3 se desvía al endpoint sin tocar una línea de la aplicación.

### Private DNS: el detalle que hace transparente el cambio

Si **no** activás el private DNS, el interface endpoint igual te da un nombre propio (`vpce-….sqs.us-east-1.vpce.amazonaws.com`) que hay que poner a mano en la app o en el SDK. Activarlo es lo que hace que el cambio sea invisible para el código.

> **Regla:** si una pregunta dice *"sin modificar la aplicación"*, la respuesta incluye **private DNS enabled**.

> ⚠️ Outdated (curso): el curso presenta los interface endpoints como **solo TCP sobre IPv4**. Hoy AWS ofrece endpoints **dualstack** (IPv4/IPv6) para varios servicios, y el tipo de dirección se elige al crearlo. La idea de fondo —una ENI con IP privada en tu subnet— no cambia.

### Un tercer tipo que casi no cae

Existe el **Gateway Load Balancer Endpoint**, que redirige tráfico hacia appliances de seguridad de terceros. Aparece en exámenes de arquitectura/networking, no en el DVA-C02; alcanza con saber que existe para no confundirlo con el Gateway Endpoint de S3/DynamoDB.

## Preguntas de examen frecuentes

- *"Acceder a S3/DynamoDB desde una subnet privada, sin NAT y sin costo"* → **Gateway Endpoint**.
- *"Llegar a un servicio de AWS desde on-premises"* o *"sin tocar la app"* → **Interface Endpoint** + private DNS.
- *"¿Qué tipo soporta DynamoDB?"* → **solo Gateway**. S3 soporta los dos.
- *"Limitar a qué buckets se puede llegar por el endpoint"* → [[endpoint-policy|endpoint policy]].
- Una Lambda en VPC que debe hablar con DynamoDB sin salir a internet → Gateway Endpoint ([[lambda-in-vpc]]).
- *"La rotación de Secrets Manager falla con timeout y la base está en subnets privadas"* → la Lambda de rotación corre en la VPC y no llega a la API: falta un **Interface Endpoint** de Secrets Manager (o un NAT) ([[SecretsManager]]).

## Ver también

[[gateway-vs-interface-endpoint]] · [[VPC]] · [[nat-gateway-vs-nat-instance]] · [[vpc-peering]]
