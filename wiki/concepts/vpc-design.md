---
title: Diseño de VPC (sizing y estructura)
category: concept
tags: [vpc, cidr, subnets, sizing, ip-planning, tiers, multi-az]
exam: [SAA-C03, DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.01 VPC Sizing & Structure.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.03 VPC Subnets.md", "raw/doc oficial/VPC CIDR blocks - Amazon Virtual Private Cloud.md", "raw/doc oficial/Subnet CIDR blocks - Amazon Virtual Private Cloud.md", "raw/doc oficial/Create a VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/Infrastructure security in Amazon VPC - Amazon Virtual Private Cloud.md"]
updated: 2026-09-23
---

# Diseño de VPC — sizing y estructura

## Definición

Planificar **qué rango de IPs usa una [[VPC]] y cómo se divide en subnets** antes de crear nada. El [[cidr|CIDR]] primario no se puede cambiar después, y los rangos solapados impiden conectar redes (peering, VPN, Direct Connect). Por eso el diseño se hace pensando en el futuro: cuántas VPCs, cuántas regions y cuentas, cuántas AZs y cuántos tiers.

## Cómo aplica en AWS

**Qué considerar** (curso):
- **Tamaño**: cuántos recursos (cada uno consume IPs) van a vivir en la VPC.
- **Redes que no podés usar**: otras VPCs, otros clouds, on-premises, partners y vendors. Ante la duda, asumir lo peor.
- **Predecir el futuro**: dejar aire para crecer.
- **Estructura**: dividir por **tiers** (web / app / db…) — cada tier con su propia seguridad — y por **zonas de resiliencia** (AZs), porque cada subnet es [[az-resilient|AZ resilient]].

**Restricciones de AWS** (doc):
- VPC y subnets: de **`/16` a `/28`**. Preferir RFC 1918; evitar `172.17.0.0/16` (Docker, Cloud9, SageMaker).
- Un CIDR secundario de otra familia RFC 1918 **no** se puede sumar (primario `10.x` → no `192.168.x`).
- Cada subnet pierde **5 IPs** reservadas.
- Producción: subnets en **al menos 2 AZs** (la doc), 3 + 1 de reserva (el curso). Recomendado: subnets por tier y private subnets para lo que no debe verse desde internet.

## Patrones comunes

### Cómo se divide un rango

Cada vez que un rango se parte en 2 mitades, el prefijo sube 1. Dividir en 4 = `+2` (`/16` → `/18`); dividir en 16 = `+4` (`/16` → `/20`).

### El patrón del curso: 4 AZs × 4 tiers = 16 subnets

| Paso | Decisión | Resultado sobre una `/16` |
|---|---|---|
| AZs | 3 AZs (funciona en casi cualquier region) + 1 de reserva = **4** | 4 × `/18` (una por AZ) |
| Tiers | web, app, db + 1 de reserva = **4** | 16 subnets |
| Subnet | `/16` ÷ 16 | **`/20` por subnet** (4096 IPs, 4091 usables) |

![[Pasted image 20260910010755.png]]

### Tamaños de VPC de referencia

| VPC Size | Netmask | Subnet Size | Hosts/subnet | Subnets/VPC | Total IPs |
|---|---|---|---|---|---|
| Micro | `/24` | `/27` | 27 | 8 | 216 |
| Small | `/21` | `/24` | 251 | 8 | 2008 |
| Medium | `/19` | `/22` | 1019 | 8 | 8152 |
| Large | `/18` | `/21` | 2043 | 8 | 16344 |
| Extra Large | `/16` | `/20` | 4091 | 16 | 65456 |

*(IPs usables: ya restadas las 5 reservadas por subnet.)*

### Ejemplo completo: Animals4life

![[Pasted image 20260910004116.png]]

Rangos a evitar: on-premises `192.168.10.0/24`, pilotos en AWS `10.0.0.0/16` y en Azure `172.31.0.0/16`, oficinas `192.168.15/20/25.0/24`, y Google `10.128.0.0/9`.

Plan:
- Usar **`10.16` → `10.127`**: evita `10.0`/`10.1` (los usa todo el mundo) y choca con Google recién en `10.128`.
- **2+ redes por region y por cuenta**: 5 regions × 2 × 4 cuentas = **40 rangos**.
- Cada region arranca en un bloque de 16: US1 `10.16–10.31`, US2 `10.32–10.47`, US3 `10.48–10.63`, EU `10.64–10.79`, Australia `10.80–10.95`.
- Cada cuenta recibe **1/4** del bloque de su region: 4 redes `/16` (ej. US1 cuenta 1 = `10.16–10.19`).
- Cada VPC `/16` → 16 subnets `/20`.

Implementación en la demo: VPC `a4l-vpc1` = `10.16.0.0/16`, con `sn-reserved-A` `10.16.0.0/20`, `sn-db-A` `10.16.16.0/20`, `sn-app-A` `10.16.32.0/20`, `sn-web-A` `10.16.48.0/20`, y así en B (`10.16.64–112`) y C (`10.16.128–176`). Cada subnet recibe un `/64` del `/56` IPv6 de la VPC (`…96[00]`–`…96[0B]`).

## Por qué importa para el futuro: peering

Planificar rangos que no se pisen no es prolijidad: es lo que deja abierta la puerta a un [[vpc-peering|VPC peering]] más adelante. Dos VPCs con CIDRs solapados **no se pueden peerear**, porque la ruta **local** siempre le gana a la del peering y el router no tendría forma de decidir si una IP es propia o del otro lado. Lo mismo vale para conectar con on-premises o con otra cuenta.

## Preguntas de examen frecuentes

- "¿Cuántas IPs usables tiene una `/28`?" → 16 − 5 = **11**.
- "La empresa planea conectar la VPC con on-premises en el futuro" → elegir un CIDR que **no se solape** con ningún rango existente; nunca la Default VPC (`172.31.0.0/16` en todas las cuentas).
- "Ampliar una VPC que se quedó chica" → **agregar un CIDR secundario** (no se puede agrandar el primario). Ojo con la cuota: **5 CIDRs IPv4 por VPC incluyendo el primario** (ampliable a 50).
- Regla rápida de hosts: **usables = 2^(32 − prefijo) − 5**. El mínimo es `/28` justamente porque con menos de 16 IPs las 5 reservadas no dejarían espacio útil.
- "Diseño HA" → subnets por tier **repetidas en varias AZs**, no una subnet grande.
- Ver también: [[multi-az]] · [[single-point-of-failure]]
