---
title: Security Groups vs NACLs
category: comparison
tags: [vpc, security-groups, nacl, firewall, stateful, stateless, ephemeral-ports]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.05 Stateful vs Stateless Firewalls.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.06 Network Access Control Lists (NACLs).md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.07 VPC Security Groups (SGs).md", "raw/doc oficial/Infrastructure security in Amazon VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/Control subnet traffic with network access control lists - Amazon Virtual Private Cloud.md", "raw/doc oficial/Control traffic to your AWS resources using security groups - Amazon Virtual Private Cloud.md", "raw/doc oficial/Security group rules - Amazon Virtual Private Cloud.md", "raw/doc oficial/Default security groups for your VPCs - Amazon Virtual Private Cloud.md"]
updated: 2026-09-22
---

# Comparativa: Security Groups vs NACLs

## Antes de la tabla: stateful vs stateless

Toda conexión TCP tiene dos mitades: el **request** (del cliente, desde un [[ephemeral-port|ephemeral port]] `1024–65535`, hacia un well-known port como `443`) y la **response** (del `443` de vuelta al ephemeral port del cliente). Que cada mitad sea *inbound* u *outbound* depende de si mirás desde el cliente o desde el servidor.

| Caso | Request | Response |
|---|---|---|
| Bob → servidor web | `119.18.36.73:1337` → `1.3.3.7:443` (inbound para el server) | `1.3.3.7:443` → `119.18.36.73:1337` (outbound) |
| Servidor → updates | `1.3.3.7:12345` → `133.33.33.7:443` (outbound) | `133.33.33.7:443` → `1.3.3.7:12345` (inbound) |

- **[[stateless-firewall|Stateless]]** (NACL): ve request y response como cosas independientes → **2 reglas por conexión**, y la de la response tiene que abrir el **rango de ephemeral ports**, no el puerto de la app.
- **[[stateful-firewall|Stateful]]** (SG): reconoce que la response pertenece a un request permitido y la deja pasar sola → **1 regla**.

![[Pasted image 20260918032310.png]]

## Tabla comparativa

| | **Security Group** | **Network ACL** |
|---|---|---|
| Nivel | **Recurso** (se asocia a la [[eni\|ENI]]) | **Subnet** |
| Estado | [[stateful-firewall\|Stateful]]: la respuesta se permite sola | [[stateless-firewall\|Stateless]]: request y response necesitan su regla |
| Tipos de regla | **Solo ALLOW** | **ALLOW y DENY** |
| Qué no permitís | [[implicit-deny\|Implicit deny]] (sin forma de denegar explícitamente) | Explicit DENY + regla `*` (implicit deny) |
| Evaluación | **Todas las reglas juntas** | **En orden** por número (1–32766), la **primera que matchea** gana |
| Source/destination | IP, CIDR, prefix list **o otro SG** (referencias lógicas, self-reference) | Solo IP/CIDR (no entiende recursos lógicos) |
| Alcance | Instancias con el SG, estén en la subnet que estén | Todo lo que **cruza el borde** de la subnet; no el tráfico dentro de la misma subnet |
| Default | Default SG: inbound desde sí mismo, outbound all. SG nuevo: nada inbound, all outbound | **Default NACL: permite todo**. Custom NACL nueva: **deniega todo** |
| Asociación | Varios SGs por recurso (se suman) | 1 NACL por subnet; 1 NACL → N subnets |
| Costo | Gratis | Gratis |

Ninguno de los dos filtra el DNS de Amazon (Route 53 Resolver), DHCP, IMDS ni Time Sync.

## Cómo se ve cada uno en un caso real

**NACL de la subnet web** para que Bob llegue por HTTPS:

| Regla | Tipo | Puerto | Origen/destino | Acción | Para qué |
|---|---|---|---|---|---|
| Inbound 110 | HTTPS | 443 | `0.0.0.0/0` | Allow | Request de Bob |
| Outbound 120 | Custom TCP | 1024–65535 | `0.0.0.0/0` | Allow | Response a sus ephemeral ports |
| `*` | todo | todo | `0.0.0.0/0` | Deny | Implicit deny |

Y si web habla con app por un puerto de la aplicación, hacen falta **4 reglas más**: outbound (request) + inbound (ephemeral) en la NACL de web, e inbound (request) + outbound (ephemeral) en la NACL de app.

**SG** para el mismo escenario: web-SG con inbound `443` desde `0.0.0.0/0`; app-SG con inbound `1337` desde **`sg-…/web-SG`**. Cualquier instancia nueva con web-SG ya puede hablar con app, sin tocar reglas.

![[Pasted image 20260918044517.png]]

## Cuándo usar cada uno

- **SG = el control principal** (lo dice la doc): más versátil, stateful, referencias entre SGs. Alcanza para casi todo.
- **NACL = control secundario / guard rail**:
  - **Bloquear una IP o red concreta** (actor malicioso) → DENY en la NACL. Un SG no puede.
  - Barrera gruesa a nivel subnet y **defensa en profundidad** por si alguien lanza una instancia con el SG equivocado.
- Patrón típico: SG por tier encadenados (ALB-SG → web-SG → db-SG) + NACLs default o con algunos DENY.

## Cómo saber cuál de los dos cortó

Los [[vpc-flow-logs|VPC Flow Logs]] dicen **que** se bloqueó, no **quién**. Pero se deduce leyendo la dirección del tráfico, justamente aprovechando que uno es stateful y el otro no:

| Lo que se ve en los flow logs | Quién cortó |
|---|---|
| Request **ACCEPT**, respuesta **REJECT** | La **NACL**: el SG dejó pasar y permite la vuelta por ser stateful, así que falta la regla outbound de la NACL sobre los [[ephemeral-port\|ephemeral ports]] |
| **Solo** el request, como **REJECT** | Puede ser el **SG** (sin regla inbound) **o** la NACL de entrada — hay que mirar los dos |

## Trampa típica del examen

- "Bloquear una IP maliciosa que ataca el web server" → **NACL con DENY** (numerada **antes** que el ALLOW). Agregar algo al SG no sirve: no hay deny.
- "Se permitió el puerto 443 inbound en la NACL y el cliente no recibe respuesta" → falta la **outbound a ephemeral ports** (stateless).
- "Se permitió inbound en el SG, ¿hace falta outbound para la respuesta?" → **No**, es stateful.
- Una NACL con DENY en la regla 200 y ALLOW en la 100 para el mismo tráfico → **se permite** (gana el número más bajo).
- "Dos instancias de la **misma subnet** no se comunican" → la NACL **no** es la causa (no filtra dentro de la subnet): mirar los SGs.
- "Escalar reglas cuando el número de instancias cambia" → **referenciar SGs** (o self-reference), no IPs.
- Custom NACL recién creada y asociada → corta **todo** el tráfico hasta que agregues reglas.
- Los SGs se asocian a **ENIs**, no a subnets ni (técnicamente) a instancias.
