---
title: VPC (Virtual Private Cloud)
category: service
tags: [vpc, networking, subnets, cidr, route-tables, igw, nat-gateway, security-groups, nacl, dns, dhcp, ipv6, default-vpc, endpoints, peering, flow-logs]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.04 Default VPC (Virtual Private Cloud) — Basics.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.01 Servicios públicos vs. privados.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.02 Custom VPC.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.03 VPC Subnets.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.04 VPC Routing e Internet Gateway.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.06 Network Access Control Lists (NACLs).md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.07 VPC Security Groups (SGs).md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.08 Network Address Translation (NAT) y NAT Gateway.md", "raw/doc oficial/What is Amazon VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/How Amazon VPC works - Amazon Virtual Private Cloud.md", "raw/doc oficial/Amazon Virtual Private Cloud Connectivity Options - Amazon Virtual Private Cloud Connectivity Options.md", "raw/doc oficial/Create a VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/VPC CIDR blocks - Amazon Virtual Private Cloud.md", "raw/doc oficial/IP addressing for your VPCs and subnets - Amazon Virtual Private Cloud.md", "raw/doc oficial/Subnet CIDR blocks - Amazon Virtual Private Cloud.md", "raw/doc oficial/Subnets for your VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/Understanding Amazon DNS - Amazon Virtual Private Cloud.md", "raw/doc oficial/DNS attributes for your VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/View DNS hostnames for your EC2 instance - Amazon Virtual Private Cloud.md", "raw/doc oficial/View and update DNS attributes for your VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/DHCP option sets in Amazon VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/DHCP option set concepts - Amazon Virtual Private Cloud.md", "raw/doc oficial/Work with DHCP option sets - Amazon Virtual Private Cloud.md", "raw/doc oficial/Configure route tables - Amazon Virtual Private Cloud.md", "raw/doc oficial/Subnet route tables - Amazon Virtual Private Cloud.md", "raw/doc oficial/How route priority works - Amazon Virtual Private Cloud.md", "raw/doc oficial/Enable internet access for a VPC using an internet gateway - Amazon Virtual Private Cloud.md", "raw/doc oficial/Add internet access to a subnet - Amazon Virtual Private Cloud.md", "raw/doc oficial/Connect to the internet or other networks using NAT devices - Amazon Virtual Private Cloud.md", "raw/doc oficial/NAT gateways - Amazon Virtual Private Cloud.md", "raw/doc oficial/NAT gateway basics - Amazon Virtual Private Cloud.md", "raw/doc oficial/Regional NAT gateways for automatic multi-AZ expansion - Amazon Virtual Private Cloud.md", "raw/doc oficial/DNS64 and NAT64 - Amazon Virtual Private Cloud.md", "raw/doc oficial/Control traffic to your AWS resources using security groups - Amazon Virtual Private Cloud.md", "raw/doc oficial/Security group rules - Amazon Virtual Private Cloud.md", "raw/doc oficial/Default security groups for your VPCs - Amazon Virtual Private Cloud.md", "raw/doc oficial/Create a security group for your VPC - Amazon Virtual Private Cloud.md", "raw/doc oficial/Configure security group rules - Amazon Virtual Private Cloud.md", "raw/doc oficial/Delete a security group - Amazon Virtual Private Cloud.md", "raw/doc oficial/Associate security groups with multiple VPCs - Amazon Virtual Private Cloud.md", "raw/doc oficial/Share security groups with AWS Organizations - Amazon Virtual Private Cloud.md", "raw/doc oficial/Control subnet traffic with network access control lists - Amazon Virtual Private Cloud.md", "raw/doc oficial/Infrastructure security in Amazon VPC - Amazon Virtual Private Cloud.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.09 VPC Endpoints.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.10 VPC Flow Logs.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.11 Lambda en una VPC.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.12 VPC Peering.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.13 Resumen para el examen (cheat sheet).md"]
updated: 2026-09-23
---

# VPC — Virtual Private Cloud

## ¿Qué es?

Una **red virtual privada dentro de AWS**, lógicamente aislada de las demás, donde corren los servicios privados (como [[EC2]]). Elegís el rango de IPs, lo dividís en subnets, y decidís con route tables, gateways, security groups y NACLs qué entra y qué sale. También conecta AWS con redes on-premises (entorno híbrido, vía VPN o Direct Connect).

La VPC en sí **no tiene costo**; se cobran algunos componentes (NAT gateways, IPAM, traffic mirroring…) y **todas las IPv4 públicas**, incluidas las [[elastic-ip|Elastic IPs]]. Las IPv4 privadas son gratis.

## Casos de uso

- Aislar cargas de trabajo en redes privadas, con **subnets por tier** (web / app / db).
- Conectar AWS con el datacenter propio (Site-to-Site VPN, Direct Connect).
- Controlar el acceso a internet de cada recurso: público, solo saliente (NAT) o nada.
- Conectar VPCs entre sí (peering, Transit Gateway).

## Características clave

- Se crea **en una cuenta y una region** específicas. Es **[[region-resilient|region resilient]]**: opera en todas las AZs de la region.
- **Privada y aislada por defecto**: lo de adentro se comunica entre sí; **nada entra ni sale sin configuración explícita**. La excepción es la **Default VPC**.

### Default VPC vs. Custom VPC

| | Default VPC | Custom VPC |
|---|---|---|
| Cantidad | **1 por region** máx | **5 por region** (soft limit, ampliable a cientos) |
| CIDR | Siempre `172.31.0.0/16` | Lo elegís |
| Subnets | Una por AZ, **públicas** | Las creás vos (arrancan privadas) |
| IGW + route `0.0.0.0/0` | Preconfigurados | Manual |
| SG + NACL | Default | Default (hay que ajustarlos) |
| IP pública auto | **Sí** | No |
| DNS hostnames públicos | Activados | Desactivados (`enableDnsHostnames=false`) |
| Uso | Demos | **Producción** |

![[Pasted image 20260614011120.png]]

Default VPC: una subnet `/20` por AZ que segmenta el CIDR **sin superponerse** (cae una AZ → solo se pierde esa subnet). Ejemplo en `us-east-2`: `172.31.0.0/20`, `172.31.16.0/20`, `172.31.32.0/20`. Si no indicás subnet al lanzar una instancia, AWS elige una default subnet.

![[Pasted image 20260614011509.png]]

**Custom VPC**, además:
- **Tenancy**: `Default` (hardware compartido; cada instancia puede pedir el suyo) o `Dedicated` ([[dedicated-tenancy]]): **todo** lo que lances en esa VPC va a hardware dedicado. Bastante más caro. Outposts exige `Default`.
- **Hybrid networking**: puede conectarse a on-premises.
- El diseño del rango y las subnets (cuántas AZs, cuántos tiers) se ve en [[vpc-design]].

![[Pasted image 20260910011836.png]]

*Diseño de referencia del curso: VPC `10.16.0.0/16`, 4 tiers (reserved/db/app/web) × 3 AZs + espacio para una cuarta, subnets `/20`.*

### Direccionamiento IP ([[cidr|CIDR]])

**IPv4**
- El CIDR primario va de **`/16` (65.536 IPs) a `/28` (16 IPs)** — nada más grande que `/16`. Recomendado: rangos RFC 1918 (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`).
- El CIDR primario **no se puede achicar, agrandar ni desasociar**. Sí se pueden **agregar CIDRs secundarios**, también de `/16` a `/28`, sin solaparse. Cuota: **5 CIDRs IPv4 por VPC contando el primario** (o sea, 4 secundarios por defecto), ampliable a **50**. Cada CIDR agregado suma su **local route** automáticamente.
- Restricciones: no se puede usar `0.0.0.0/8`, `127.0.0.0/8`, `169.254.0.0/16` ni `224.0.0.0/4`. Evitar `172.17.0.0/16` (lo usan Docker/Cloud9/SageMaker). Un secundario de otro rango RFC 1918 distinto del primario **no** se permite (ej.: primario `10.x` → no podés sumar `192.168.x`).
- Custom VPCs: **5 por region** por defecto (soft limit).
- Regla de oro: **nunca uses el mismo CIDR en dos VPCs que algún día podrían hablarse** (rompe peering/VPN/Direct Connect gateway).

**IPv6**
- Opcional. Amazon-provided = **`/56` fijo** (no elegís el rango). Se pueden asociar **hasta 5 CIDRs IPv6** a una VPC (`/44`–`/60`, en pasos de `/4`), cuota ampliable a **50**.
- Cada subnet toma un rango del de la VPC: **`/44`–`/64`**, típicamente un **`/64`**. Un `/56` tiene lugar para **256 subnets `/64`** (los 8 bits de diferencia = los 2 últimos dígitos hex del 4.º grupo).
- Las IPv6 de Amazon son **públicas y globalmente únicas**: no hay NAT; el acceso se controla con routing, SGs y NACLs.

> La diapositiva original del curso decía "un único CIDR IPv6 `/56` por VPC"; la nota del curso ya se corrigió (2026-09-23) a **5, ampliable a 50**, que coincide con [Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html). El `/56` sigue siendo el tamaño fijo del bloque Amazon-provided.

**IPs públicas IPv4**
- La subnet tiene un atributo **auto-assign public IPv4** (y otro para IPv6); se puede **sobreescribir al lanzar** la instancia.
- La IP pública dinámica sale de un pool de Amazon, **no es de tu cuenta** y se libera al detener/terminar. Para una IP fija: **[[elastic-ip|Elastic IP]]** (asignada a tu cuenta en una region hasta que la liberes; se asocia a instancias, [[eni|ENIs]], NAT gateways…).
- Las IPv4 públicas **nunca se configuran en el SO**: el IGW hace NAT 1:1 (ver más abajo).

### Subnets

- Una subnet vive **en una sola AZ, para siempre** → es **[[az-resilient|AZ resilient]]**. Una subnet **nunca** abarca varias AZs; una AZ puede tener 0, 1 o muchas subnets.
- CIDR: subconjunto del de la VPC, de `/16` a `/28`, **sin solaparse** con otras subnets.
- Tipos según IPs: **IPv4-only**, **dual-stack** o **IPv6-only**.
- Tipos según **routing** (no es un atributo, es la route table):

| Tipo | Route table |
|---|---|
| **[[public-subnet\|Public]]** | Ruta directa a un **Internet Gateway** |
| **Private** | Sin ruta al IGW; sale por **NAT** |
| **VPN-only** | Ruta a una VPN vía virtual private gateway |
| **Isolated** | Sin rutas fuera de la VPC |

- Por defecto, **todas las subnets de una VPC se ven entre sí** (local route).
- AWS recomienda poner los recursos en **private subnets** y usar [[bastion-host|bastion host]] o NAT para el acceso.

#### Las 5 IPs reservadas por subnet

En **toda** subnet AWS reserva las **primeras 4 y la última**. En `10.16.16.0/20`:

| IP | Uso |
|---|---|
| `10.16.16.0` | Network address |
| `10.16.16.1` | **VPC router** (network + 1) |
| `10.16.16.2` | **DNS** (base + 2) |
| `10.16.16.3` | Reservada para uso futuro |
| `10.16.31.255` | Broadcast (no se usa en VPC, pero se reserva) |

→ una `/24` tiene 256 direcciones pero solo **251 usables**; una `/28`, 16 → **11**. En IPv6 también se reservan las 4 primeras y la última de cada subnet. (Con BYOIP IPv4 se pueden usar la primera y la última.)

### DNS en la VPC

- DNS provisto por el **Route 53 Resolver** ("AmazonProvidedDNS"), presente en cada AZ. Direcciones: **base del CIDR primario + 2** (ej. `10.16.0.2`), **`169.254.169.253`** (link-local IPv4) y `fd00:ec2::253` (IPv6).
- Dos atributos de la VPC:

| Atributo | Qué controla | Default |
|---|---|---|
| `enableDnsSupport` | Si funciona la resolución con el DNS de Amazon | `true` |
| `enableDnsHostnames` | Si las instancias con IP pública reciben **hostname DNS público** | `false` (**`true` solo en la Default VPC**) |

- Con **los dos en `true`**: hostnames públicos + el Resolver resuelve los hostnames privados de Amazon. Si alguno está en `false`, ninguna de las dos cosas.
- **Private hosted zones de [[Route53]] e interface endpoints con private DNS exigen los dos en `true`.**
- El hostname público resuelve a la IP **pública desde afuera** y a la **privada desde adentro** de la VPC.
- **Ni SGs ni NACLs pueden filtrar el tráfico al Route 53 Resolver** (para eso: Route 53 Resolver DNS Firewall). Límite fijo de **1024 paquetes/s por ENI** hacia servicios link-local (DNS + IMDS + NTP) — superarlo = [[throttling|throttling]] de DNS.

### DHCP option sets

- Configuración de red que reciben las instancias vía DHCP: **servidores DNS, domain name, NTP, NetBIOS**, lease de IPv6.
- Cada region tiene un **default option set** (DNS = `AmazonProvidedDNS`). Una VPC tiene **un solo option set a la vez**; un option set puede asociarse a **muchas VPCs**.
- **Inmutables**: no se editan. Para cambiar algo → crear uno nuevo y asociarlo (el viejo queda huérfano y se puede borrar). Las instancias lo toman solas al renovar el lease (horas), **sin reiniciar**.
- Aplica a **toda la VPC**: no se puede dar un DNS distinto a cada subnet con este mecanismo.
- Caso de uso típico: que las instancias usen un **DNS propio** (ej. el de un Active Directory) en lugar de AmazonProvidedDNS → option set custom con esos `domain-name-servers`.
- Si la VPC queda **sin option set**, se deshabilita la resolución DNS (en instancias Xen no hay DNS → sin internet). Con un IGW, siempre especificar un DNS (propio o AmazonProvidedDNS).

### Routing: VPC router y route tables

- El **VPC router** es un router implícito, altamente disponible, en todas las AZs de la VPC, con una interfaz en cada subnet (**network + 1**). Por defecto solo enruta **entre subnets de la VPC**.
- Una **route table** define qué hace el router con el tráfico que **sale** de una subnet. Cada route = **Destination** (qué tráfico matchea) + **Target** (a dónde se envía: `local`, IGW, NAT GW, peering, VGW, ENI…).

| Regla | Detalle |
|---|---|
| Main route table | Se crea con la VPC; la usan las subnets **sin asociación explícita**. No se puede borrar; sí editar o **reemplazar** por otra. |
| Custom route table | Se crean las que quieras; se borran solo si no tienen asociaciones. |
| Asociación | **1 subnet ↔ exactamente 1 route table**; 1 route table ↔ 0..N subnets. |
| Local route | Una por cada CIDR (IPv4 e IPv6) de la VPC; en **todas** las route tables. |
| IPv4 e IPv6 | Se tratan por separado: `0.0.0.0/0` **no** incluye IPv6 → hace falta `::/0` aparte. |

- **Prioridad**: gana la ruta **más específica** ([[longest-prefix-match|longest prefix match]]): prefijo más alto = más específica. Ejemplo con `10.16.0.0/16 → local` y `0.0.0.0/0 → igw`: un paquete a `10.16.32.10` matchea ambas y gana `/16` (queda local); uno a `1.3.3.7` solo matchea `/0` → IGW.
- Empates exactos: una ruta **estática** (IGW, NAT GW, peering, TGW, ENI…) le gana a una **propagada** (VPN/BGP).
- **Local route** — la regla base es "siempre está y no se toca". La doc actual tiene dos excepciones avanzadas, ambas para meter appliances de inspección (firewall, IDS/IPS) en el camino:

| | 1. Ruta más específica que la local | 2. Reemplazar el target de la local |
|---|---|---|
| Dónde | Route table de **subnet** | Sobre todo **gateway route tables** (asociadas a un IGW o VGW: tráfico **entrante**) |
| Qué se hace | Se **agrega** una fila: destino = CIDR **completo** de una subnet de la VPC; target NAT GW, [[eni\|ENI]] o Gateway Load Balancer endpoint | La fila del CIDR de la VPC deja de decir `local` y apunta a la ENI del appliance (se puede **restaurar** después) |
| Ejemplo | `172.31.0.0/16 → local` + `172.31.0.0/20 → eni-x`: a `172.31.5.10` gana `/20` (pasa por el appliance); a `172.31.200.5` solo matchea `local` | `172.31.0.0/16 → eni-x`: **todo** el tráfico hacia la VPC pasa por el appliance |
| Alcance | Una subnet puntual; el resto sigue `local` | Toda la VPC |

- Buena práctica (curso y doc): dejar la **main route table sin ruta al IGW** y asociar explícitamente cada subnet pública a una custom. Si una subnet nueva se crea sin asociación, hereda la main — y no querés que "se vuelva pública sin querer".

![[Pasted image 20260918014507.png]]

### Internet Gateway (IGW)

- Componente **region resilient**, escalado horizontalmente y redundante: **no es cuello de botella ni punto de falla**. Soporta IPv4 e IPv6. **Gratis** (se paga la transferencia de datos).
- **1 VPC ↔ 0 o 1 IGW; 1 IGW ↔ 0 o 1 VPC.** Se puede crear sin asociar.
- Hace de gateway entre la VPC y la **internet / AWS Public Zone** (S3, SQS, SNS… vía endpoints públicos).

**Receta de una public subnet** (6 pasos del curso):
1. Crear el IGW.
2. **Attach** a la VPC.
3. Crear una **custom route table**.
4. **Asociarla** a la subnet.
5. Agregar **default routes** `0.0.0.0/0` (y opcionalmente `::/0`) → IGW.
6. Activar **auto-assign public IPv4** en la subnet (y/o IPv6), o asignar Elastic IPs.

![[Pasted image 20260918015716.png]]

Sin ruta al IGW, una instancia **no sale a internet aunque tenga IP pública**. Y con ruta, además tienen que permitirlo los SGs/NACLs.

**Checklist de "la instancia en la subnet pública no tiene internet"** (en orden; un solo eslabón que falte corta todo):
1. ¿La VPC tiene un **IGW attached**?
2. ¿La route table **de esa subnet** tiene `0.0.0.0/0 → igw`?
3. ¿La instancia tiene **IPv4 pública o Elastic IP**?
4. ¿El **Security Group** permite el tráfico?
5. ¿La **NACL** lo permite **en los dos sentidos** (incluidos los [[ephemeral-port|ephemeral ports]] de la respuesta)?

**IPv4 pública = NAT estático 1:1 en el IGW.** La instancia **solo conoce su IP privada**; el IGW guarda el mapeo privada ↔ pública. Saliente: reemplaza el *source* (`10.16.16.20` → `43.250.192.20`); entrante: reemplaza el *destination* al revés. Por eso **el SO nunca muestra la IP pública** — pregunta trampa clásica.

![[Pasted image 20260918020436.png]]

### Bastion host / jumpbox

Una instancia en una **public subnet** que recibe las conexiones de administración entrantes y desde ahí salta a los recursos privados. Se restringe por IP de origen, SSH o identidad corporativa. El curso los muestra para entenderlos, pero hoy son **mala práctica** (puerto 22 abierto 24/7, llaves SSH que rotar, [[single-point-of-failure|SPOF]] del acceso administrativo): la respuesta moderna es **SSM Session Manager** — conexión **saliente** desde la instancia, sin puerto 22, sin bastion y sin llaves (ver [[EC2]]). Detalle en [[bastion-host]].

### Security Groups y NACLs

Dos capas de firewall, que se usan juntas. La comparación completa (stateful vs stateless, orden de evaluación, trampas) está en [[security-groups-vs-nacls]].

**Network ACL** — nivel **subnet**, [[stateless-firewall|stateless]], reglas ALLOW **y DENY** numeradas (1–32766), se evalúan de menor a mayor y **la primera que matchea gana**; regla `*` = [[implicit-deny|implicit deny]]. Filtra solo el tráfico que **cruza el límite de la subnet**. La **Default NACL** permite todo; una **custom NACL** nueva deniega todo. 1 subnet ↔ 1 NACL; 1 NACL ↔ N subnets. Por ser stateless, hay que permitir la respuesta hacia los [[ephemeral-port|ephemeral ports]] (`1024–65535`).

**Security Group** — nivel **recurso** ([[eni|ENI]]), [[stateful-firewall|stateful]], **solo ALLOW** (no hay explicit deny). Todas las reglas se evalúan juntas; varios SGs en un recurso se **suman**.
- SG nuevo: **sin reglas inbound** (entra nada) y **outbound all**. Los cambios aplican al instante a todos los recursos asociados.
- **Default SG** de cada VPC: inbound **desde sí mismo** (todo), outbound all. **No se puede borrar**. Si lanzás una instancia sin SG, queda con el default.
- **Referencias lógicas**: una regla puede usar **otro SG como source/destination** ("cualquier cosa con ese SG"), en la misma VPC, por peering o (solo inbound) por Transit Gateway. Escala solo con instancias nuevas y cuenta como **1 regla**. Patrón típico: ALB-SG ← web-SG ← db-SG.
- **Self-reference**: el SG se referencia a sí mismo → comunicación libre entre todos sus miembros (clusters, domain controllers, apps HA).
- Name/description **inmutables**; nombre único por VPC y no puede empezar con `sg-`. No se pueden copiar entre regions.
- Para borrar un SG: no debe estar asociado, **ni referenciado por otro SG**, ni ser el default.
- **SG VPC Associations**: un SG puede asociarse a **otras VPCs de la misma region y cuenta** (no el default SG, ni con la default VPC).
- **Shared SGs**: se comparten con cuentas de la misma **[[Organizations|Organization]]** que usan una subnet compartida (VPC sharing).

Ni SGs ni NACLs filtran: DNS de Amazon, DHCP, metadata de EC2 (IMDS), Time Sync, activación de licencias Windows, IPs reservadas del router.

Cuotas por defecto ([Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)): **60 reglas inbound + 60 outbound por SG** (y aparte para IPv4 e IPv6), **5 SGs por ENI** (ampliable a 16; reglas × SGs por ENI ≤ 1.000), **20 reglas por NACL** en cada sentido (ampliable a 40), 200 subnets y 200 route tables por VPC.

### NAT y NAT Gateway

**NAT** reescribe direcciones de los paquetes. El IGW hace **NAT estático** (1 privada ↔ 1 pública). Lo que se suele llamar "NAT" es **[[ip-masquerading|IP masquerading]]** (en rigor NAT + PAT): **muchas IPs privadas detrás de una sola pública** → da **salida a internet, pero no permite conexiones entrantes iniciadas desde afuera**.

**NAT Gateway** (servicio gestionado; recomendado sobre la NAT instance — ver [[nat-gateway-vs-nat-instance]]):

| Aspecto | NAT Gateway (zonal, el clásico) |
|---|---|
| Dónde | En una **public subnet**, con una **Elastic IP** asociada al crearlo |
| Routing | Route table de la private subnet: `0.0.0.0/0 → nat-gw`; la subnet del NAT GW: `0.0.0.0/0 → igw` |
| Resiliencia | **AZ resilient** (redundante dentro de su AZ). Para HA: **un NAT GW por AZ** + una route table por AZ apuntando al de su AZ |
| Performance | **5 Gbps → escala solo hasta 100 Gbps**; 1M → 10M paquetes/s; 55.000 conexiones simultáneas por IP y destino (hasta 8 IPs) |
| Protocolos | TCP, UDP, ICMP |
| Seguridad | **No admite security groups**; se controla con la NACL de su subnet (usa puertos 1024–65535) |
| Costo | Por **hora** + por **GB procesado** |

Recorrido de un paquete desde una private subnet: la instancia envía con su IP privada → el NAT GW reemplaza el source por **su IP privada** → al pasar por el IGW, el source pasa a ser la **Elastic IP del NAT GW**.

![[Pasted image 20260918154733.png]]

- **Tipos de conectividad**: **Public** (default; sale a internet vía IGW) y **Private** (sin EIP; para llegar a otras VPCs u on-premises vía Transit Gateway/VGW — si lo mandás al IGW, se descarta).
- **Modo regional** (lanzado en nov-2025): un **único NAT GW regional** que se **expande y contrae solo** entre las AZs donde hay ENIs (hasta 60 min para expandirse a una AZ nueva), **no necesita public subnet**, trae su propia route table hacia el IGW, hasta 32 IPs por AZ. No soporta NAT privado. El NAT GW clásico pasa a llamarse **zonal**.
- No se puede rutear hacia un NAT GW **a través de un peering** (cliente → peering → NAT → internet no funciona), ni desde VPN/Direct Connect vía VGW (sí vía Transit Gateway).
- Uso frecuente en DVA: una **Lambda dentro de una VPC** no tiene internet → hay que ponerla en private subnets con ruta a un NAT GW.

> ⚠️ Outdated (curso): la **tabla** NAT instance vs NAT Gateway del curso todavía dice "escala hasta 45 Gbps" (el cuerpo de la nota ya se corrigió a 100 Gbps) → la doc actual dice 5 Gbps con escalado automático a 100 Gbps. La versión vieja de la nota decía que era "el único servicio que usa Elastic IPs" → falso, también se asocian a instancias y ENIs (ya corregido en el curso). "El NAT GW es AZ resilient y hay que poner uno por AZ" → sigue valiendo para el **zonal**, pero ahora existe el **regional**.

> Regla de decisión del curso: **salida a internet** desde una private subnet → NAT. Si lo que hay del otro lado son **servicios de AWS** (S3, DynamoDB…) → casi siempre **VPC endpoint**, no NAT.

**IPv6 y NAT**
- Las IPv6 de AWS son públicamente enrutables → **no hace falta NAT** para IPv6.
- `::/0 → IGW`: bidireccional. `::/0 →` **[[egress-only-internet-gateway|Egress-Only Internet Gateway]]**: **solo saliente** (el "NAT gateway de IPv6").
- **NAT64 + DNS64**: el NAT GW traduce IPv6 → IPv4 (siempre disponible, no se activa). Con **DNS64** activado en la subnet, el Route 53 Resolver sintetiza una IPv6 `64:ff9b::/96` para destinos solo-IPv4 y esa ruta se manda al NAT GW. Así, cargas **IPv6-only** hablan con servicios **IPv4-only**.

> ⚠️ Outdated (curso): "los NAT Gateways no funcionan con IPv6" → la doc actual dice que el NAT GW procesa IPv6 haciendo **NAT64**. Lo que sigue valiendo: para salida a internet por IPv6 no se usa NAT, se usa IGW o Egress-Only IGW.

### Acceso privado a servicios de AWS: VPC Endpoints

Servicios como [[S3]] o DynamoDB viven en la **AWS Public Zone**, no dentro de tu VPC. Sin endpoints, una instancia en subnet privada solo llega a ellos saliendo por el NAT Gateway. Los endpoints son el atajo privado que evita ese rodeo — y su costo.

| | Gateway Endpoint | Interface Endpoint |
|---|---|---|
| Servicios | **Solo S3 y DynamoDB** | Casi todos (+ S3) |
| Mecanismo | Ruta hacia una [[prefix-list\|prefix list]] | [[eni\|ENI]] con IP privada ([[privatelink\|PrivateLink]]) |
| Seguridad | [[endpoint-policy\|Endpoint policy]] | Security group + policy |
| Costo | Gratis | Por hora + por GB |
| Desde on-premises | No | Sí |

Desarrollado en [[vpc-endpoints]] y [[gateway-vs-interface-endpoint]].

### Conectar VPCs entre sí

- **[[vpc-peering|VPC peering]]**: une **dos** VPCs por IPs privadas. **No transitivo**, sin CIDRs solapados, y hay que poner rutas en **ambos** lados. No permite [[edge-to-edge-routing|edge-to-edge routing]]: no se "presta" el IGW ni el NAT del otro lado.
- **[[transit-gateway|Transit Gateway]]**: router regional central entre VPCs, VPNs y Direct Connect. La respuesta cuando son **muchas** VPCs — unir N todas contra todas por peering cuesta N×(N-1)/2 conexiones.
- **Site-to-Site VPN**: dos túneles IPsec entre un virtual private gateway/TGW y tu customer gateway.
- El tráfico entre recursos de AWS (incluso entre IPs públicas o entre regions) **se queda en la red global privada de AWS**.

### Ver el tráfico: VPC Flow Logs

Los [[vpc-flow-logs|VPC Flow Logs]] capturan **metadatos** (no el payload) del tráfico IP a nivel VPC, subnet o [[eni|ENI]], con la acción `ACCEPT`/`REJECT`. Destinos: CloudWatch Logs, S3 o Kinesis Data Firehose. Son **la** herramienta para diagnosticar tráfico bloqueado; para ver el contenido de los paquetes hace falta [[traffic-mirroring|Traffic Mirroring]].

### Lambda dentro de la VPC

Una función conectada a tu VPC alcanza los recursos privados pero **pierde la salida directa a internet**, y su ENI **nunca recibe IP pública**. Ver [[lambda-in-vpc]].

## Integración con otros servicios

- [[EC2]] — las instancias viven en una subnet; sus ENIs llevan los security groups.
- [[global-infrastructure]] — subnets ↔ AZs; VPC ↔ region.
- [[Route53]] — el Route 53 Resolver es el DNS de la VPC; las private hosted zones se asocian a VPCs.
- [[Organizations]] — VPC sharing y shared security groups entre cuentas de la organización.
- [[S3]] — se llega por la AWS Public Zone (IGW/NAT) o por un gateway endpoint privado ([[vpc-endpoints]]).
- [[lambda-in-vpc]] — necesita NAT GW (o endpoints) para salir de la VPC; sus ENIs consumen IPs de tus subnets.
- [[CloudWatchLogs]] — destino habitual de los [[vpc-flow-logs|flow logs]], con [[metric-filter|metric filters]] para alarmar sobre rechazos.
- [[EC2]] — el **source/destination check** que habilita una NAT instance vive en la ENI de la instancia.

## Gotchas y trampas del examen

> Repaso de todo el tema en una pantalla: [[vpc-cheat-sheet]].

- El CIDR de la Default VPC es **idéntico en todas las cuentas/regions** → conflictos de peering/VPN. En producción: Custom VPC con CIDR planificado.
- "Todo lo desplegado en la Default VPC recibe IP pública" — no asumir eso en una Custom (auto-assign apagado y `enableDnsHostnames=false`).
- Servicio privado ≠ inaccesible: una EC2 puede recibir IP pública vía IGW ([[global-infrastructure|zonas de red]]).
- Si **borrás la Default VPC**, recrearla requiere **abrir un caso de soporte**. Todas sus subnets son públicas (auto-assign public IP) → mala práctica en prod.
- Hosts por subnet: restar **5 IPs reservadas** (una `/24` = 251 usables, no 254).
- **Una subnet = una AZ**. "Una subnet que abarque dos AZs" nunca es la respuesta.
- La instancia **no ve su IP pública** en el SO: el IGW hace NAT 1:1.
- "Public subnet" = **ruta al IGW** en su route table, no un checkbox. Para que un **recurso** dentro sea alcanzable hacen falta tres cosas a la vez: ruta al IGW + IP pública/EIP + SG/NACL que lo permitan.
- Muchos servicios (una instancia, una RDS "de prueba") caen en la **Default VPC** si no especificás otra — por eso "recibió IP pública y salió a internet sin configurar nada".
- Main route table con ruta al IGW → toda subnet nueva sin asociación explícita nace **pública**. Dejarla sin ruta a internet.
- Acceso administrativo **sin puertos abiertos** → **SSM Session Manager** (SSM Agent + rol IAM), no bastion host.
- `0.0.0.0/0` no cubre IPv6: para salida IPv6 hace falta `::/0` (IGW o Egress-Only IGW).
- **NAT GW va en la public subnet**, y su ruta sale de la **private**. Un solo NAT GW para varias AZs = [[single-point-of-failure|punto único de falla]] (salvo el modo regional).
- NAT GW **no tiene security groups**. NAT instance sí, y necesita **source/destination check desactivado**.
- Bloquear **una IP maliciosa** → **NACL** (DENY). Un SG no puede denegar.
- Private hosted zone que "no resuelve" → revisar `enableDnsSupport` y `enableDnsHostnames` (los dos en `true`).
- Cambiar el DNS de la VPC → **nuevo DHCP option set** (no se editan).
- El curso puede usar valores anteriores (45 Gbps, "sin IPv6 en NAT GW", "un solo `/56`"); la doc actual es la de arriba. Si una opción del examen solo encaja con el valor viejo, pensar qué versión evalúa la pregunta.
- **DynamoDB solo soporta gateway endpoint**; S3 soporta los dos. "Interface endpoint para DynamoDB" es siempre distractor.
- Un **gateway endpoint no tiene security group** (no es una ENI): su control es la [[endpoint-policy|endpoint policy]].
- "Sin modificar la aplicación" + interface endpoint → hace falta **private DNS habilitado**, si no hay que cambiar el hostname en el código.
- El **peering no es transitivo** y **no presta la salida a internet** del otro lado. Peering creado y aceptado pero sin conectividad → faltan las rutas en **ambas** route tables, o las reglas de SG/NACL.
- Los **flow logs no son retroactivos**: si no estaban activos, no hay datos del incidente. Y capturan **metadatos, no payload**.
- Una **Lambda en subnet pública no tiene internet**: su ENI nunca recibe IP pública.

## Demos del curso

- [Custom VPC](https://learn.cantrill.io/courses/1101194/lectures/45241152)
- [Crear las subnets del VPC design](https://learn.cantrill.io/courses/1101194/lectures/26953794)
- [Configuring A4L public subnets and jumpbox — parte 1](https://learn.cantrill.io/courses/1101194/lectures/26982553)
- [Configuring A4L public subnets and jumpbox — parte 2](https://learn.cantrill.io/courses/1101194/lectures/27186078)
- [Implementar private internet access con NAT Gateway](https://learn.cantrill.io/courses/1101194/lectures/26982643)
