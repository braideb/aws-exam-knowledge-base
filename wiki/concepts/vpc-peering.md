---
title: VPC Peering
category: concept
tags: [vpc, networking, peering, conectividad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.12 VPC Peering.md"]
updated: 2026-09-23
---

# VPC Peering

> ⚠️ **Fuente pendiente de revisión:** la sección del curso [[05.12 VPC Peering]] está marcada por el humano como *"(Pendiente a revisar)"* (2026-09-23) y esta página todavía **no se contrastó con doc oficial** (no hay clippings del tema en `raw/doc oficial/`). Tratar los datos finos con cautela hasta cerrar esa revisión.

## Definición

Una **VPC peering connection** une **dos** VPCs para que se comuniquen por **IPs privadas**, como si fueran una sola red. Pueden estar en la misma cuenta o en cuentas distintas, y en la misma región o en regiones distintas.

Las tres restricciones que definen todo lo demás:

- **No es transitiva.** Si A está peered con B, y B con C, **A no llega a C**. Hace falta un peering A–C directo.
- Las VPCs **no pueden tener [[cidr|CIDRs]] solapados**.
- El peering **no atraviesa firewalls**: hay que abrir security groups y NACLs igual que siempre.

## Cómo aplica en AWS

### Los tres pasos para que funcione

Un peering recién creado **no conecta nada por sí solo**. El examen suele preguntar justamente por el paso que falta:

1. **Aceptar la solicitud.** Se crea como *request* y la cuenta dueña de la otra VPC lo tiene que aceptar (si es la misma cuenta, igual hay que aceptarlo).
2. **Rutas en las dos route tables.** En la VPC A, una ruta con destino el CIDR de B y target el peering (`pcx-xxxx`); y la simétrica en B. **Si falta una de las dos, el tráfico va pero no vuelve.**
3. **Abrir SG y NACL.** El [[security-groups-vs-nacls|security group]] de destino tiene que permitir el origen, y las NACLs de ambas subnets dejar pasar request y respuesta.

Detalle cómodo: se pueden **referenciar security groups** entre VPCs peered (si están en la misma región), igual que dentro de una misma VPC.

### Por qué el CIDR no puede solapar

Es la misma lógica del [[longest-prefix-match|longest prefix match]]: si las dos VPCs usan `10.0.0.0/16`, el router no tiene forma de decidir si `10.0.1.20` es local o del peer — y la ruta **local siempre gana**. Esto conecta directo con [[vpc-design]]: planificar rangos que no se pisen es lo que deja la puerta abierta a peerings futuros.

### Qué NO hace un peering

- **No hay [[edge-to-edge-routing|edge-to-edge routing]]:** desde la VPC A no podés usar el Internet Gateway, el NAT Gateway, los [[vpc-endpoints|VPC endpoints]] ni la VPN/Direct Connect de la VPC B. El peering sirve para llegar a los **recursos** de B, no para "prestar" su salida a internet.
- **No es transitivo** (vale la pena repetirlo: es el error clásico).
- La **resolución de DNS a IPs privadas** entre VPCs peered **no viene activada**: hay que habilitarla explícitamente en la conexión, en los dos lados.

## Patrones comunes

### El costo de escalar con peering

Para conectar *todas contra todas* N VPCs hacen falta **N × (N-1) / 2** peerings:

| VPCs | Conexiones |
|---|---|
| 3 | 3 |
| 5 | 10 |
| 10 | 45 |

Ese crecimiento cuadrático es el argumento numérico a favor del [[transit-gateway|Transit Gateway]], que actúa como hub central y **sí** rutea de forma transitiva.

> **Regla:** pocas VPCs → **peering**. Muchas VPCs → **Transit Gateway**.

## Preguntas de examen frecuentes

- *"A habla con B, B habla con C, ¿A llega a C?"* → **No.** El peering no es transitivo.
- *"El peering está creado y aceptado pero no hay conectividad"* → faltan las **rutas** (en las dos route tables) o las reglas de **SG/NACL**.
- *"Conectar 12 VPCs entre sí"* → **Transit Gateway**, no 66 peerings.
- *"¿Puedo usar el NAT Gateway de la otra VPC?"* → **No**, no hay edge-to-edge routing.
- *"Los nombres DNS privados no resuelven a través del peering"* → hay que **habilitar la resolución DNS** en la conexión, en ambos lados.

## Ver también

[[VPC]] · [[vpc-design]] · [[vpc-endpoints]] · [[security-groups-vs-nacls]]
