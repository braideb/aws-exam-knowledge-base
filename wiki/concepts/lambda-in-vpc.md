---
title: Lambda en una VPC
category: concept
tags: [lambda, vpc, networking, serverless, dva-development]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.11 Lambda en una VPC.md"]
updated: 2026-09-23
---

# Lambda en una VPC

> ⚠️ **Fuente pendiente de revisión:** la sección del curso [[05.11 Lambda en una VPC]] está marcada por el humano como *"(Pendiente a revisar)"* (2026-09-23) y esta página todavía **no se contrastó con doc oficial** (no hay clippings del tema en `raw/doc oficial/`). Tratar los datos finos con cautela hasta cerrar esa revisión.

> Esta página cubre **el ángulo de red** de Lambda, que es lo que el curso desarrolló hasta acá. La página de servicio completa de Lambda (concurrencia, layers, destinations, ciclo de vida de eventos) sigue pendiente — es el hueco #1 de [[dva-development]].

## Definición

Por defecto, una función **Lambda corre en una VPC gestionada por AWS** y tiene salida a internet, pero **no** puede ver los recursos privados de tu [[VPC]] (por ejemplo, una base RDS en una subnet privada).

Cuando la configurás para conectarse a **tu** VPC, Lambda crea [[eni|ENIs]] en las subnets que le indiques y, a través de ellas, alcanza esos recursos. Le das **subnets y un security group**.

## Cómo aplica en AWS

### Las tres configuraciones posibles

| Configuración | ¿Ve recursos privados de tu VPC? | ¿Sale a internet? |
|---|---|---|
| Lambda **sin** VPC (default) | No | **Sí**, directo |
| Lambda en VPC, **subnets privadas sin NAT** | Sí | **No** (solo vía [[vpc-endpoints\|VPC Endpoints]]) |
| Lambda en VPC, **subnets privadas con ruta al NAT Gateway** | Sí | Sí, por el NAT |

El punto clave: al conectarla a la VPC, la Lambda **pierde el acceso directo a internet**. Si además necesita salir, hay que ponerla en subnets privadas y enrutar la salida por un **NAT Gateway** ([[nat-gateway-vs-nat-instance]]). Si solo necesita hablar con servicios de AWS, lo ideal son los **VPC endpoints**.

### La trampa: la subnet pública no le da internet

Una subnet es [[public-subnet|pública]] porque sus recursos tienen IPv4 pública y hay ruta al Internet Gateway. Como la ENI de la Lambda **nunca recibe IP pública**, ponerla en una subnet pública no sirve de nada: el tráfico llega al IGW y muere ahí.

> La salida a internet de una Lambda en VPC **siempre** es subnet privada + NAT Gateway.

### Permisos

El [[execution-role|execution role]] necesita permisos de EC2 para crear y borrar las ENIs: `ec2:CreateNetworkInterface`, `ec2:DescribeNetworkInterfaces` y `ec2:DeleteNetworkInterface`. La policy gestionada que los trae es **`AWSLambdaVPCAccessExecutionRole`**. Sin ellos la función queda en error de configuración al intentar arrancar en la VPC.

### Límite de escalado

Las ENIs y, sobre todo, las **IPs libres de las subnets**, son un recurso finito. Si la subnet se queda sin direcciones disponibles, las invocaciones fallan con errores del estilo **`ENILimitReached`**. Por eso conviene darle subnets con espacio de sobra y en **varias AZs**.

> Dato fino: hoy Lambda usa **Hyperplane ENIs** compartidas, así que conectar una función a una VPC ya casi no agrega latencia de arranque (antes sí era un problema conocido).

## Patrones comunes

```
Lambda en VPC
   ├── necesita RDS privada      → subnets privadas + SG; el SG de RDS permite al SG de la Lambda
   ├── necesita DynamoDB/S3      → Gateway Endpoint (gratis)
   ├── necesita otro servicio AWS→ Interface Endpoint
   └── necesita una API pública  → subnets privadas + ruta al NAT Gateway
```

## Preguntas de examen frecuentes

- *"Mi Lambda tiene que leer una RDS en subnet privada"* → configurarla **en la VPC** (subnets + SG), y el SG de RDS tiene que permitir **al SG de la Lambda** (referencia lógica entre security groups, ver [[security-groups-vs-nacls]]).
- *"Mi Lambda en VPC ya no puede llamar a una API pública"* → le falta salida: subnets privadas con ruta a un **NAT Gateway**.
- *"Mi Lambda en VPC tiene que llegar a DynamoDB sin internet"* → **Gateway Endpoint** de DynamoDB.
- *"La puse en una subnet pública y sigue sin internet"* → esperado: la ENI nunca recibe IP pública.
- *"Las invocaciones fallan con `ENILimitReached`"* → la subnet se quedó sin IPs libres; subnets más grandes y en más AZs.

## Ver también

[[VPC]] · [[vpc-endpoints]] · [[nat-gateway-vs-nat-instance]] · [[execution-role]] · [[dva-development]]
