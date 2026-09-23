---
title: Gateway Endpoint vs Interface Endpoint
category: comparison
tags: [vpc, endpoints, privatelink, s3, dynamodb]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.09 VPC Endpoints.md"]
updated: 2026-09-23
---

# Gateway Endpoint vs Interface Endpoint

> ⚠️ **Fuente pendiente de revisión:** la sección del curso [[05.09 VPC Endpoints]] está marcada por el humano como *"(Pendiente a revisar)"* (2026-09-23) y esta página todavía **no se contrastó con doc oficial** (no hay clippings del tema en `raw/doc oficial/`). Tratar los datos finos con cautela hasta cerrar esa revisión.

Los dos dan acceso **privado** a servicios de AWS desde una [[VPC]] sin pasar por internet. Se diferencian en casi todo lo demás. El concepto completo está en [[vpc-endpoints]].

| | Gateway Endpoint | Interface Endpoint |
|---|---|---|
| Servicios | **Solo S3 y DynamoDB** | Mayoría de servicios (+ S3) |
| Cómo funciona | **Ruta** en la route table, hacia una [[prefix-list\|prefix list]] | [[eni\|ENI]] con IP privada ([[privatelink\|PrivateLink]]) |
| Dónde vive | En la route table; no ocupa IPs | Dentro de una subnet, una por AZ |
| Seguridad | [[endpoint-policy\|Endpoint policy]] | **Security group** + endpoint policy |
| Costo | **Gratis** | Por hora **+ por GB** procesado |
| Desde on-premises | **No** | **Sí** (VPN / Direct Connect) |
| DNS | Transparente por la ruta | Nombre propio, o **private DNS** para que sea transparente |
| Alta disponibilidad | Implícita (es una ruta) | La armás vos: un endpoint por AZ |

## Cuándo usar cada uno

**Gateway Endpoint** cuando:
- El servicio es **S3 o DynamoDB** — son los dos únicos que lo soportan.
- El acceso es **solo desde dentro de la VPC**.
- Importa el costo: es gratis, y es la forma de sacarse de encima el tráfico del NAT Gateway, que sí se cobra por GB ([[nat-gateway-vs-nat-instance]]).

**Interface Endpoint** cuando:
- El servicio **no es** S3 ni DynamoDB (SQS, SNS, Kinesis, Systems Manager, API Gateway…).
- Hace falta llegar **desde on-premises** por VPN o Direct Connect.
- Querés **security groups** como control de acceso, además de la policy.
- Necesitás que el nombre DNS habitual del servicio resuelva a una IP privada.

### Las dos reglas rápidas

- **DynamoDB → solo Gateway Endpoint.** No existe otra opción; si una respuesta ofrece un interface endpoint para DynamoDB, es distractor.
- **S3 → los dos.** Gateway si alcanza con acceso desde la VPC y querés que sea gratis; Interface si hace falta llegar desde on-premises.

## Trampa típica del examen

- *"Acceder a S3 desde una subnet privada sin NAT y sin costo adicional"* → **Gateway Endpoint**. La opción "interface endpoint" es correcta técnicamente pero falla en el "sin costo".
- *"Sin modificar la aplicación"* → **Interface Endpoint con private DNS habilitado**. Sin private DNS hay que cambiar el hostname en el código, que es justo lo que la pregunta prohíbe.
- Un gateway endpoint **no tiene security group**, porque no es una ENI. Si la respuesta propone "asociarle un SG al gateway endpoint", es incorrecta: su control es la [[endpoint-policy|endpoint policy]].
- El **Gateway Load Balancer Endpoint** es un tercer tipo, para appliances de terceros. No es lo mismo que el Gateway Endpoint de S3/DynamoDB.

## Ver también

[[vpc-endpoints]] · [[VPC]] · [[nat-gateway-vs-nat-instance]] · [[lambda-in-vpc]]
