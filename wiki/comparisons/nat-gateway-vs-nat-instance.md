---
title: NAT Gateway vs NAT instance
category: comparison
tags: [vpc, nat, nat-gateway, nat-instance, ha, ipv6]
exam: [SAA-C03, DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.08 Network Address Translation (NAT) y NAT Gateway.md", "raw/doc oficial/Compare NAT gateways and NAT instances - Amazon Virtual Private Cloud.md", "raw/doc oficial/Enable private resources to communicate outside the VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/NAT gateway basics - Amazon Virtual Private Cloud.md", "raw/doc oficial/Regional NAT gateways for automatic multi-AZ expansion - Amazon Virtual Private Cloud.md", "raw/doc oficial/Connect to the internet or other networks using NAT devices - Amazon Virtual Private Cloud.md"]
updated: 2026-09-23
---

# Comparativa: NAT Gateway vs NAT instance

Las dos dan a una private subnet **salida a internet sin entrada iniciada desde afuera** ([[ip-masquerading|IP masquerading]]). AWS recomienda el **NAT Gateway**. Contexto completo en [[VPC]].

## Tabla comparativa

| Atributo | **NAT Gateway** | **NAT instance** |
|---|---|---|
| Qué es | Servicio gestionado de VPC | Una **instancia EC2** con una AMI configurada para hacer NAT |
| Disponibilidad | Redundante **dentro de su AZ**. Para HA: uno por AZ (o el modo **regional**) | Vos armás el [[failover\|failover]] (scripts) — es un [[single-point-of-failure\|SPOF]] |
| Ancho de banda | 5 Gbps, escala solo hasta **100 Gbps** | El del tipo de instancia |
| Mantenimiento | AWS | Vos (parches, updates del SO) |
| Tipo y tamaño | Oferta única, no se elige | Elegís tipo y tamaño |
| IP pública | [[elastic-ip\|Elastic IP]] elegida **al crearlo** | EIP o IP pública; se puede cambiar cuando quieras |
| Security groups | **No se pueden asociar** (se ponen en los recursos detrás) | Sí, en la instancia y en los recursos detrás |
| NACL | La de su subnet | La de su subnet |
| Port forwarding | ❌ | ✅ (configurándolo a mano) |
| Usarlo de [[bastion-host\|bastion]] | ❌ | ✅ |
| Timeout de conexión | Devuelve **RST** | Envía **FIN** |
| Fragmentación IP | Solo UDP | UDP, TCP e ICMP |
| Costo | Por NAT GW, por hora y **por GB procesado** | Por instancia, por hora, según tipo |
| Setup clave | [[public-subnet\|Public subnet]] + EIP + ruta `0.0.0.0/0 → nat-gw` en la private | Public subnet + **desactivar source/destination check** + ruta `0.0.0.0/0 → i-…` |

**Source/destination check**: por defecto una EC2 solo acepta tráfico cuyo origen o destino es ella misma. Una NAT instance reenvía tráfico ajeno → hay que **desactivarlo**, o no funciona.

![[Pasted image 20260918154702.png]]

## Zonal vs regional (NAT Gateway)

| | **Zonal** (el clásico) | **Regional** (nov-2025) |
|---|---|---|
| Dónde vive | Una AZ, en una public subnet | La VPC; se expande a las AZs con [[eni\|ENIs]] |
| Public subnet | Necesaria | **No hace falta** |
| HA multi-AZ | Uno por AZ + una route table por AZ | Automática (hasta 60 min para sumar una AZ nueva) |
| IPs | Hasta 8 | Hasta 32 por AZ |
| NAT privado | ✅ | ❌ |

## Cuándo usar cada uno

- **Casi siempre: NAT Gateway.** Menos administración, más ancho de banda, HA dentro de la AZ.
- **NAT instance** solo si necesitás algo que el NAT GW no hace: port forwarding, usarla como bastion, o el costo en entornos muy chicos de prueba.
- **HA real**: un NAT GW **zonal por AZ**, cada private subnet apuntando al de **su** AZ — o un NAT GW **regional**.
- **IPv6**: no se usa NAT → **[[egress-only-internet-gateway|Egress-Only IGW]]** para salida. El NAT GW solo interviene en IPv6 para **NAT64** (IPv6-only → IPv4-only).

## La tercera opción: no usar NAT

Si lo único que hay del otro lado son **servicios de AWS**, el NAT es un rodeo caro: el tráfico sale a la internet pública y se paga por GB procesado. Un [[vpc-endpoints|VPC endpoint]] lo evita por completo — y el **gateway endpoint** de S3/DynamoDB además **es gratis**. La ruta del endpoint le gana a la default `0.0.0.0/0` hacia el NAT por [[longest-prefix-match|longest prefix match]].

> Dato relacionado: el **source/destination check** que hay que desactivar en una NAT instance vive en su [[eni|ENI]], no en la instancia ([[EC2]]).

## Trampa típica del examen

- "Las instancias privadas de las AZ B y C pierden internet cuando cae la AZ A" → había **un solo NAT GW** (zonal) en la AZ A. Solución: uno por AZ (o regional).
- "NAT instance configurada, ruta OK, pero no pasa tráfico" → **source/destination check** sin desactivar.
- "Restringir el tráfico que pasa por el NAT GW con un security group" → no se puede: SG en las instancias, o NACL en la subnet del NAT.
- "NAT Gateway en una private subnet" → no sale a internet: el NAT GW zonal va en la **public**.
- "Salida IPv6 sin permitir entrada" → **Egress-Only IGW**, no NAT GW.
- El IGW es **[[region-resilient|region resilient]]**; el NAT GW zonal es **[[az-resilient|AZ resilient]]**. No confundirlos.
- La tabla del curso dice "hasta 45 Gbps": dato viejo (el cuerpo de la nota ya dice 100), hoy son **5 Gbps con escalado automático a 100 Gbps**.
