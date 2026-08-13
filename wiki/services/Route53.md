---
title: Route 53
category: service
tags: [route53, dns, hosted-zones, records, ttl, dominios]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.14 Route 53 (R53) — Fundamentos.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.15 DNS Record Types.md", "raw/doc oficial/Supported DNS record types - Amazon Route 53.md", "raw/doc oficial/Choosing between alias and non-alias records - Amazon Route 53.md"]
updated: 2026-07-23
---

# Route 53

## ¿Qué es?

El **DNS administrado** de AWS: registra dominios y hospeda **zone files** en name servers gestionados. Es de los pocos servicios **globales con una única base de datos** → **[[globally-resilient]]**, y el único servicio AWS con **[[sla|SLA]] del 100%** (si el DNS se cae, no importa cuán resiliente sea el resto).

## Casos de uso

- Registrar dominios (R53 como *registrar* ante los *registries* de cada TLD).
- Resolver nombres hacia recursos AWS (ELB, [[CloudFront]], [[S3]] websites) con alias records.
- DNS interno privado para VPCs.

## Características clave

### Registro de un dominio (flujo)

1. R53 consulta al **registry** del TLD (ej: PIR para `.org`) por disponibilidad.
2. Crea el **zone file** y lo hospeda en **4 managed name servers**.
3. El registry agrega **NS records** en la zone del TLD apuntando a esos name servers → **delegación**: pasan a ser **autoritativos** para el dominio.

![[Pasted image 20260627165735.png]]

> **Registry vs. Registrar vs. Registrant** (el examen los usa para confundir):
> - **Registry**: administra un TLD y su base maestra (PIR para `.org`, Verisign para `.com`).
> - **Registrar**: la empresa donde **comprás** el dominio y habla con el registry (R53 actúa como registrar).
> - **Registrant**: **vos**, el dueño.
>
> R53 puede ser registrar **Y** DNS hosting a la vez, pero son funciones separadas. Caso híbrido frecuente: dominio comprado en otro registrar (GoDaddy) + hosted zone en R53 → solo hay que **cambiar los NS records en el registrar** apuntando a los 4 name servers que R53 asignó.

### Hosted Zones

- **Public**: responde desde internet.
- **Private**: solo para las **VPCs asociadas** (nombres internos).
- Almacenan **records** (recordsets).

### Tipos de record DNS

| Tipo | Mapea | Nota de examen |
|---|---|---|
| **NS** | Delegación de zona | Cómo `.com` apunta a los NS de `amazon.com` |
| **A / AAAA** | Host → IPv4 / IPv6 | Se suelen crear ambos |
| **CNAME** | Host → host (alias) | ❌ no apunta a IP, ❌ prohibido en el **apex** |
| **MX** | Correo del dominio | Priority: menor = más prioritario; valor sin punto final = host de la misma zone |
| **TXT** | Texto arbitrario | Probar propiedad del dominio; SPF/DKIM/DMARC (el record type **SPF está deprecado** — se usa TXT) |
| **CAA** | Qué CAs pueden emitir certs | Del clipping oficial — cae en SAA |
| **SOA** | Autoridad de la zona | R53 lo crea automáticamente junto con los NS |
| **PTR** | IP → nombre (reverse DNS) | El inverso del A record |
| **SRV** | Descubrir servicios (priority/weight/port + host) | Usado por protocolos como SIP/LDAP |

**Trío anti-spam (todos TXT):** **SPF** (qué servidores pueden enviar mail del dominio) · **DKIM** (firma criptográfica que prueba no-alteración) · **DMARC** (qué hacer con los que fallan + a dónde reportar). Casi obligatorio al usar **Amazon SES**. **MX failover:** varios MX con prioridades escalonadas (10/20/30); menor número = mayor prioridad; mismo número = reparto de carga. El destino de un MX debe resolver a un **A record**, nunca a una IP ni CNAME.

### TTL y caching

- [[ttl|TTL]] en segundos = cuánto puede **cachearse** un record.
- Respuesta **authoritative** (del name server de la zone) vs. **non-authoritative** (cache del resolver).
- Tip práctico: **bajar el TTL con anticipación** (días antes, para que expire el TTL viejo) antes de migrar; tras estabilizar, **volver a subirlo**. TTL bajo (60 s) también en registros de **[[failover]] de DR**.
- Trade-off: TTL alto = menos consultas/más barato pero cambios lentos; TTL bajo = cambios casi inmediatos pero más consultas/costo.

![[Pasted image 20260627180030.png]]

### ALIAS records (exclusivos de R53)

- Funcionan **en el [[zone-apex|apex]]** (donde el CNAME está prohibido).
- Apuntan directo a recursos AWS: ELB, CloudFront, S3 website, API Gateway, VPC endpoints, Elastic Beanstalk, Global Accelerator, AppSync, otra hosted zone.
- **Gratis** hacia recursos AWS (los CNAME se cobran); no permiten TTL custom.
- Ver [[alias-vs-cname]].

## Integración con otros servicios

- [[S3]] — alias hacia website endpoints (bucket = nombre del dominio).
- [[CloudFront]] — alias hacia distribuciones.
- [[VPC]] — private hosted zones.

## Gotchas y trampas del examen

- "Apuntar `midominio.com` (apex) a un ELB/CloudFront/S3" → **ALIAS**, nunca CNAME.
- CNAME a una dirección IP → inválido; eso es un A record.
- Un CNAME en un nombre **bloquea cualquier otro record** con ese mismo nombre (limitación del estándar DNS).
- MX con valor terminado en punto = FQDN absoluto; sin punto = relativo a la zone.
- R53 = globally resilient; buen candidato en escenarios de failover DNS multi-region.

## Demos del curso

- [Registering a Domain with Route53](https://learn.cantrill.io/courses/1101194/lectures/25301533)
