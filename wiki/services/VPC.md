---
title: VPC (Virtual Private Cloud)
category: service
tags: [vpc, networking, subnets, igw, default-vpc]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.04 Default VPC (Virtual Private Cloud) — Basics.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.01 Servicios públicos vs. privados.md"]
updated: 2026-07-23
---

# VPC — Virtual Private Cloud

## ¿Qué es?

Una **red virtual privada dentro de AWS**, donde corren los servicios privados (como [[EC2]]). También conecta AWS con redes on-premises (entorno híbrido, vía VPN o Direct Connect).

## Casos de uso

- Aislar cargas de trabajo en redes privadas.
- Conectar AWS con el datacenter propio.
- Controlar el acceso a internet de los recursos.

## Características clave

- Se crea **en una cuenta y una region** específicas. Es **[[region-resilient|region resilient]]** (opera desde múltiples AZs).
- **Privada y aislada por defecto**: lo de adentro se comunica entre sí; nada entra o sale sin configuración. La excepción es la **Default VPC**.

### Default VPC vs. Custom VPC

| | Default VPC | Custom VPC |
|---|---|---|
| Cantidad | **1 por region** máx | Muchas |
| CIDR | Siempre `172.31.0.0/16` | Lo elegís |
| Subnets | Una por AZ, **públicas** | Las creás vos |
| IGW + SG + NACL | Preconfigurados | Manual |
| IP pública auto | **Sí** | No |
| Uso | Demos | **Producción** |

![[Pasted image 20260614011120.png]]

### Estructura de la Default VPC

- Subnets segmentan el CIDR **sin superponerse**, una por AZ (cae una AZ → solo se pierde esa subnet).
- **Internet Gateway (IGW)** ya conectado; traduce **1:1** la IP pública ↔ privada (la instancia nunca ve su IP pública en el SO).
- Ejemplo en `us-east-2`: `172.31.0.0/20`, `172.31.16.0/20`, `172.31.32.0/20`.

![[Pasted image 20260614011509.png]]

### Reglas de [[cidr|CIDR]] (memorizar)

- El CIDR de una VPC va de **`/16` (65.536 IPs) a `/28` (16 IPs)** — nada más grande que `/16`.
- El CIDR primario **no se puede achicar ni cambiar** tras crear la VPC; sí se pueden **agregar hasta 5 CIDRs secundarios**.
- Custom VPC: **5 por region** por defecto (soft limit). Rangos privados RFC 1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.
- Regla de oro: **nunca uses el mismo CIDR en dos VPCs que algún día podrían hablarse** (rompe peering/VPN).

### Las 5 IPs reservadas por subnet

En **toda** subnet AWS reserva las **primeras 4 y la última**. En una `/24` (`10.0.0.0/24`):

| IP | Uso |
|---|---|
| `.0` | Network address |
| `.1` | VPC router |
| `.2` | DNS (Amazon Provided DNS, base VPC+2) |
| `.3` | Reservada para uso futuro |
| `.255` | Broadcast (no se usa, pero se reserva) |

→ una `/24` tiene 256 direcciones pero solo **251 usables**. Por eso un cálculo de "cuántas instancias entran" nunca da redondo.

## Integración con otros servicios

- [[EC2]] — las instancias se lanzan dentro de una subnet.
- [[global-infrastructure]] — subnets ↔ AZs.
- [[Route53]] — private hosted zones se asocian a VPCs.

## Gotchas y trampas del examen

- El CIDR de la Default VPC es **idéntico en todas las cuentas/regiones** → conflictos de peering/VPN. En producción: Custom VPC con CIDR planificado.
- "Todo lo desplegado en la Default VPC recibe IP pública" — no asumir eso en una Custom.
- Servicio privado ≠ inaccesible: una EC2 puede recibir IP pública vía IGW ([[global-infrastructure|zonas de red]]).
- Si **borrás la Default VPC**, recrearla requiere **abrir un caso de soporte**. Todas sus subnets son públicas (auto-assign public IP) → mala práctica en prod.
- Cálculo de hosts por subnet: restar **5 IPs reservadas** (una `/24` = 251 usables, no 254).
