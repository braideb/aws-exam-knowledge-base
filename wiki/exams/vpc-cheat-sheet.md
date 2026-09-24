---
title: VPC — Cheat sheet de examen
category: exam
tags: [vpc, networking, repaso, cheat-sheet, dva-c02]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.13 Resumen para el examen (cheat sheet).md"]
updated: 2026-09-24
---

# VPC — Cheat sheet de examen

Repaso rápido de lo que más se pregunta de redes. Cada punto linkea a la página donde está desarrollado. No reemplaza la lectura: sirve para la última pasada antes del examen.

## Los puntos, uno por línea

- **VPC:** servicio **regional**, abarca todas las AZs. [[cidr|CIDR]] IPv4 de **/28 a /16**. El CIDR primario no se cambia. IPv6 opcional (**/56**). → [[VPC]]
- **Default VPC:** `172.31.0.0/16`, una por región, subnets `/20` públicas por AZ e IGW ya listo. → [[VPC]]
- **Subnet:** vive en **una sola AZ**, no se mueve. CIDR dentro del de la VPC, sin solapar. **5 IPs reservadas** por subnet. Bloque IPv6 = **/64**. → [[vpc-design]]
- **Pública vs privada:** pública = ruta `0.0.0.0/0` → **IGW** + IP pública. Privada = sin esa ruta; sale por **NAT**. → [[public-subnet]]
- **Route table:** una por subnet (main o custom); gana la ruta **más específica** ([[longest-prefix-match]]); la local no se borra. → [[VPC]]
- **IGW:** [[region-resilient|region resilient]], **0 o 1 por VPC**, NAT estático 1:1, tráfico **bidireccional**. El SO solo ve la **IP privada**. → [[VPC]]
- **NAT Gateway:** en subnet **pública**, usa **[[elastic-ip|Elastic IP]]**, **solo saliente**, [[az-resilient|AZ-resilient]] (uno por AZ para HA regional). No sirve para IPv6 → [[egress-only-internet-gateway|Egress-Only IGW]]. → [[nat-gateway-vs-nat-instance]]
- **Security Group:** en la [[eni|ENI]], [[stateful-firewall|stateful]], **solo ALLOW**, puede referenciar otros SGs. → [[security-groups-vs-nacls]]
- **NACL:** en la **subnet**, [[stateless-firewall|stateless]] (regla para request **y** para response con [[ephemeral-port|ephemeral ports]]), **ALLOW y DENY**, se evalúa por número de regla. Ideal para **bloquear IPs**. → [[security-groups-vs-nacls]]
- **VPC Endpoints:** acceso privado a servicios AWS. **Gateway** (S3/DynamoDB, gratis, es una ruta) vs **Interface** ([[privatelink|PrivateLink]], ENI + SG, con costo, sirve desde on-premises). → [[gateway-vs-interface-endpoint]]
- **Flow Logs:** **metadatos** (ACCEPT/REJECT) a CloudWatch Logs / S3 / Firehose. **No ven el payload**. → [[vpc-flow-logs]]
- **Lambda en VPC:** ENIs en subnets privadas + SG; **pierde internet directo** (NAT para salir); endpoints para servicios AWS. → [[lambda-in-vpc]]
- **Peering:** **2** VPCs, **no transitivo**, sin CIDRs solapados. Muchas VPCs → [[transit-gateway|Transit Gateway]]. → [[vpc-peering]]
- **DNS de la VPC:** IP base **+2** (y `169.254.169.253`); `enableDnsSupport` y `enableDnsHostnames` en `true` para nombres públicos y private hosted zones. → [[VPC]]

## Los números de memoria

| Dato | Valor |
|---|---|
| CIDR IPv4 de la VPC | **/16** (máx) a **/28** (mín) |
| CIDR IPv6 de la VPC | **/56** fijo |
| CIDR IPv6 de la subnet | **/64** fijo |
| IPs reservadas por subnet | **5** |
| IGW por VPC | **0 o 1** |
| CIDR de la Default VPC | `172.31.0.0/16`, subnets **/20** |
| DNS Resolver de la VPC | IP base **+2** |
| Peerings para N VPCs full-mesh | **N × (N-1) / 2** |

## Escenario → respuesta

| El escenario dice… | La respuesta es |
|---|---|
| Salida a internet desde subnet privada | **NAT Gateway** (uno por AZ para HA) |
| Salida a internet por **IPv6** | **Egress-Only Internet Gateway** |
| Llegar a **S3/DynamoDB** sin internet y sin costo | **Gateway Endpoint** |
| Llegar a un servicio AWS **desde on-premises**, o **sin tocar la app** | **Interface Endpoint** (PrivateLink + private DNS) |
| **Bloquear una IP** concreta | **NACL** (el SG no tiene DENY) |
| Permitir tráfico "del tier web al tier app" | **SG que referencia otro SG** |
| Ver **por qué** se rechaza el tráfico | **VPC Flow Logs** (`action = REJECT`) |
| Ver el **contenido** de los paquetes | **[[traffic-mirroring\|Traffic Mirroring]]** (no Flow Logs) |
| Conectar **2** VPCs | **VPC Peering** |
| Conectar **muchas** VPCs | **Transit Gateway** |
| Lambda que lee una **RDS privada** | Lambda **en la VPC** (subnets privadas + SG) |
| Usar el NAT/IGW de la VPC vecina | **No se puede** ([[edge-to-edge-routing]]) |

## Los errores que más se repiten

1. Creer que el **NAT Gateway es resiliente a nivel región**: es [[az-resilient|AZ-resilient]], hace falta uno por AZ (salvo el modo regional nuevo).
2. Olvidar que la **NACL es stateless**: siempre hay que pensar la regla de vuelta con [[ephemeral-port|ephemeral ports]].
3. Creer que el **peering es transitivo**: no lo es. Y tampoco presta la salida a internet del otro lado.
4. Poner una **Lambda en una subnet pública** esperando que tenga internet: su ENI nunca recibe IP pública.
5. Ofrecer un **interface endpoint para DynamoDB**: DynamoDB solo soporta **gateway**.
6. Crear el peering, aceptarlo y olvidarse de las **rutas en las dos route tables**.

## Ver también

[[VPC]] · [[vpc-design]] · [[security-groups-vs-nacls]] · [[nat-gateway-vs-nat-instance]] · [[ec2-cheat-sheet]]
