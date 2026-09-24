---
title: VPC Flow Logs
category: concept
tags: [vpc, networking, troubleshooting, observabilidad, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.10 VPC Flow Logs.md", "raw/notas curso mejorado/07 Monitoring and logging/07.08 VPC Flow Logs.md"]
updated: 2026-09-24
---

# VPC Flow Logs

> ⚠️ **Fuente pendiente de revisión:** la sección del curso [[05.10 VPC Flow Logs]] está marcada por el humano como *"(Pendiente a revisar)"* (2026-09-23) y esta página todavía **no se contrastó con doc oficial** (no hay clippings del tema en `raw/doc oficial/`). La sección [[07.08 VPC Flow Logs]] del módulo de monitoring (2026-09-24) confirma los datos centrales —metadata, tres niveles, no real time, protocolos, tráfico no registrado—, pero sigue siendo material del curso.

## Definición

Los **VPC Flow Logs** capturan **metadatos** del tráfico IP que pasa por una [[VPC]] — **no el contenido de los paquetes**: IPs de origen y destino, puertos, protocolo, bytes/paquetes y la acción (**ACCEPT** o **REJECT**).

Es la herramienta para responder *"¿por qué este tráfico no llega?"* y *"¿quién habló con quién?"*.

## Cómo aplica en AWS

### Los tres niveles

Se habilitan sobre **toda la VPC**, una **subnet** o una [[eni|ENI]] específica, y capturan **desde el punto de captura hacia abajo**. A nivel VPC cubren todas las ENIs, incluidas las que se creen **después**. Al revés no funciona: activarlo en una ENI no cubre a su subnet.

Cada flow log se configura para capturar el tráfico **ACCEPTED**, **REJECTED** o **ALL**.

![[Pasted image 20260924000057.png]]

### Destinos

- **CloudWatch Logs** → [[CloudWatchLogs]] — para alarmas y métricas (con [[metric-filter|metric filters]]) o búsquedas con Logs Insights.
- **S3** → [[S3]] — archivado barato y consultas puntuales con **Athena** (SQL, se paga por dato leído). Athena **no es un destino**: consulta lo que ya está en S3.
- **Firehose** (Amazon Data Firehose, ex Kinesis Data Firehose) — análisis [[near-real-time|casi en tiempo real]] o envío a un tercero. *(El curso en 07.08 menciona solo S3 y CloudWatch Logs; Firehose se agregó después.)*

### Alcance y ciclo de vida

- Se crean **sobre un recurso concreto** y **no son retroactivos**: solo capturan desde que se activan. No sirven para investigar un incidente que ya pasó si no estaban encendidos.
- Son **inmutables**: no se editan. Para cambiar formato o destino, se borra y se crea otro.
- El **intervalo de agregación** es de **10 minutos** por defecto, bajable a **1 minuto**. Esa es la razón de que "no sean tiempo real".

### Qué NO capturan

Tráfico del **DNS de Amazon**, la **metadata de la instancia** (`169.254.169.254`, ver [[ec2-instance-metadata]]), **Amazon Time Sync** (`169.254.169.123`), activación de licencias de Windows, tráfico DHCP, y el que va a la IP reservada del router de la VPC.

### Formato de un record

Cada fila es un **flow log record** con campos en orden fijo (formato default, versión 2):

```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
2       ACC-ID     eni-ID       119.18.34.78 10.16.48.20 1337 443 6 4 336 1432917027 1432917142 ACCEPT OK
```

![[Pasted image 20260924000314.png]]

### Campos que importan

| Campo | Qué dice |
|---|---|
| `srcaddr` / `dstaddr` | IPs de origen y destino |
| `srcport` / `dstport` | Puertos; el puerto **alto** de un extremo es el [[ephemeral-port\|ephemeral port]] |
| `protocol` | Número IANA (**6** = TCP, **17** = UDP, **1** = ICMP) |
| `packets` / `bytes` | Volumen capturado en el intervalo |
| `action` | **ACCEPT** o **REJECT** |
| `log-status` | `OK`, `NODATA` (no hubo tráfico), `SKIPDATA` (se perdieron registros) |

## Patrones comunes

### Deducir si cortó el Security Group o la NACL

Los flow logs dicen **que** se bloqueó, no **quién**. Pero se deduce leyendo la dirección del tráfico, aprovechando que la NACL es [[stateless-firewall|stateless]] y el SG es [[stateful-firewall|stateful]] ([[security-groups-vs-nacls]]):

| Lo que se ve | Quién cortó |
|---|---|
| Request **ACCEPT**, respuesta **REJECT** | La **NACL**: el SG dejó pasar y permite la vuelta por ser stateful, así que lo que falta es la regla outbound de la NACL sobre los ephemeral ports |
| **Solo** el request, como **REJECT** | Puede ser el **SG** (sin regla inbound) **o** la NACL de entrada — hay que mirar las dos |

Este razonamiento es el que más cae del tema.

**Ejemplo del curso — un ping (ICMP):** dos filas, con `protocol = 1` y puertos en `0` (ICMP no usa puertos):

```
2 ACC-ID eni-ID 119.18.34.78 10.16.48.20 0 0 1 4 336 1432917027 1432917142 ACCEPT OK   ← entra el echo request
2 ACC-ID eni-ID 10.16.48.20 119.18.34.78 0 0 1 4 336 1432917094 1432917142 REJECT OK   ← se bloquea la respuesta
```

Entrada aceptada + respuesta rechazada = patrón stateless → la NACL.

![[Pasted image 20260924000637.png]]

## Preguntas de examen frecuentes

- *"Diagnosticar por qué se rechaza tráfico"* o *"auditar conexiones"* en una VPC/subnet/ENI → **VPC Flow Logs** (`action = REJECT`).
- *"Ver el contenido de los paquetes"* → **NO** son Flow Logs, es [[traffic-mirroring|Traffic Mirroring]]. Es el distractor más frecuente.
- *"Necesito alarmas cuando aparezcan rechazos"* → destino **CloudWatch Logs** + metric filter.
- *"Guardar meses de tráfico lo más barato posible"* → destino **S3** + Athena.
- *"Investigar un incidente de la semana pasada"* → si los flow logs no estaban activos, **no hay datos**. No son retroactivos.

- *"Ver en tiempo real el flujo de paquetes"* → **no** se puede con flow logs: hay demora de captura.

## Ver también

[[VPC]] · [[security-groups-vs-nacls]] · [[CloudWatchLogs]] · [[dva-troubleshooting]]

> 📖 Lectura profunda: [[05.10 VPC Flow Logs]] · [[07.08 VPC Flow Logs]]
