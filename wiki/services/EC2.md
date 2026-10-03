---
title: EC2 (Elastic Compute Cloud)
category: service
tags: [ec2, compute, iaas, ami, instancias, ebs, eni, imds, user-data, placement-groups]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.05 Elastic Compute Cloud (EC2) — Basics.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.06 Amazon Machine Image (AMI).md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.07 Conectarse a EC2.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.02 EC2 Architecture and Resilience.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.14 Network Interfaces (ENI), IPs y DNS.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.15 Elastic IP.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.17 Amazon Machine Image (AMI).md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.22 Instance Status Checks y Auto Recovery.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.16 Demostración - Instalación manual de WordPress.md", "raw/notas curso mejorado/09 Advanced EC2/09.01 Bootstrapping EC2 con User Data.md", "raw/notas curso mejorado/09 Advanced EC2/09.02 Boot Time to Service Time y AMI Baking.md", "raw/notas curso mejorado/09 Advanced EC2/09.08 Placement Groups — Overview.md", "raw/notas curso mejorado/09 Advanced EC2/09.12 Enhanced Networking (SR-IOV, ENA, EFA).md", "raw/notas curso mejorado/09 Advanced EC2/09.13 EBS Optimized.md", "raw/notas curso mejorado/10 Databases (SQL)/10.01 Databases on EC2.md"]
updated: 2026-10-03
---

# EC2 — Elastic Compute Cloud

## ¿Qué es?

Servicio de **máquinas virtuales (instancias)** — el [[iaas|IaaS]] clásico de AWS. La unidad de consumo es la instancia: un SO con recursos asignados. Vos gestionás SO y aplicaciones; AWS gestiona del [[hypervisor]] para abajo ([[shared-responsibility-model]]), sobre su propia plataforma de [[virtualization|virtualización]], [[nitro|Nitro]].

> **Esta página es el hub del tema.** Los subtemas grandes viven aparte: [[ec2-instance-types]] · [[EBS]] · [[ebs-volume-types]] · [[instance-store-vs-ebs]] · [[storage-types]] · [[ec2-purchase-options]] · [[ec2-instance-metadata]] · [[ec2-bootstrapping]] · [[placement-groups]] · [[horizontal-vs-vertical-scaling]] · [[virtualization]].

## Casos de uso

- **SO tradicional**: la app necesita cierto sistema operativo, runtime y configuración.
- **Cómputo de larga duración**: Lambda corta a los 15 minutos; EC2 corre meses o años.
- **Apps tipo servidor**, a la espera de conexiones entrantes.
- **Cargas con ráfagas o steady-state**: hay instance types para cada perfil.
- **Monolitos** con varios componentes en la misma pila.
- **Migración de cargas y disaster recovery** de sistemas tradicionales.

## Características clave

- **Servicio privado**: corre en una subnet de una [[VPC]]; acceso público solo si se configura.
- **[[az-resilient|AZ resilient]]**: la instancia vive en una subnet → una AZ. Cae la AZ, cae la instancia. La resiliencia la diseñás vos repartiendo instancias en varias AZ.
- Facturación on-demand: **por segundo** (mínimo 60 s) para Linux/Ubuntu; **por hora** para algunas AMIs comerciales (Windows, RHEL). Ver [[ec2-purchase-options]].
- **EC2 = "default compute" del examen**: la opción por defecto salvo buena razón. Cargas largas / SO a medida / monolitos → EC2. Eventos, ejecuciones cortas, "no quiero administrar servidores" → Lambda o gestionado ([[serverless]]).

### Arquitectura: hosts e instancias

Las instancias corren sobre **EC2 hosts**, el hardware físico que administra AWS. Dos modelos:

| | **Shared host** | **Dedicated host** |
|---|---|---|
| Qué pagás | Cada **instancia** | El **host completo** |
| Quién más corre ahí | Otros clientes (aislados) | Solo vos |
| Cuándo | El 99% de los casos | Licencias por socket/core ([[host-affinity]]) |

Cada host vive en **una única AZ** y tiene sus propios discos locales (instance store), ENIs hacia las subnets y acceso a volúmenes [[EBS]]. Ver también [[dedicated-tenancy]] y [[ec2-purchase-options]].

### Ciclo de vida: cuándo se mueve de host

Una instancia se queda en su host hasta que pasa una de dos cosas:

1. El **host falla** o AWS lo retira por mantenimiento.
2. Se hace **stop + start** (distinto de un **reboot**: el reboot **no** mueve la instancia).

Si se mueve, va a otro host **de la misma AZ**. Las instancias **no cruzan de AZ** de forma nativa: ni ellas, ni sus ENIs, ni sus volúmenes EBS. Migrar de AZ significa, en el fondo, **copiar y recrear** (vía snapshot).

| Acción | Instance store | IP pública dinámica | EBS |
|---|---|---|---|
| **Reboot** | Se conserva (hay que remontar) | Se conserva | Persiste |
| **Stop + start** | **Se pierde** | **Cambia** | Persiste |
| **Falla del host** | **Se pierde** | Cambia | Persiste |

Detalle completo en [[instance-store-vs-ebs]].

### Estados y facturación

```
Running  ⇄  Stopped  →  Terminated
```

| Estado | Se cobra |
|---|---|
| **Running** | Todo (CPU, RAM, storage, red) |
| **Stopped** | Solo el **almacenamiento [[EBS]]** |
| **Terminated** | Nada — **irreversible** |

![[Pasted image 20260614013829.png]]

### Almacenamiento

Dos opciones, desarrolladas en [[instance-store-vs-ebs]] y [[storage-types]]:

- **Instance store** — discos físicos del host. El **más rápido de AWS**, pero [[ephemeral-storage|efímero]] y **solo se adjunta al lanzar**.
- **[[EBS]]** — volúmenes por red, **persistentes**, con snapshots y cifrado. El boot volume de casi cualquier instancia.

### Red: ENI, IPs y DNS

Toda instancia arranca con una **[[eni|ENI]] primaria**; se le pueden sumar secundarias, incluso en otras subnets, pero **todas en la misma AZ**. Lo que la consola muestra "en la instancia" (IPs, nombres DNS, **security groups**) en realidad vive en la ENI.

| Tipo de dirección | Cuántas por ENI | Notas clave |
|---|---|---|
| IPv4 privada principal | 1 (obligatoria) | **Estática**; no cambia nunca |
| IPv4 privada secundaria | 0 o más | — |
| IPv4 pública | 0 o 1 | **Dinámica**: cambia con stop/start, no con reboot |
| [[elastic-ip\|Elastic IP]] | 1 por cada IP privada | **Fija**; en la ENI primaria reemplaza a la pública normal |
| IPv6 | 0 o más | Siempre públicas (en IPv6 no hay distinción) |

Cada ENI tiene además su **MAC address**, sus **security groups** (que afectan a **todas** las IPs de esa interfaz) y el **source/destination check**: si está activado, descarta el tráfico que no sea de/para sus propias IPs. Hay que **desactivarlo** para que una instancia funcione como NAT instance ([[nat-gateway-vs-nat-instance]]).

#### DNS split-horizon

La instancia recibe un nombre DNS privado (`ip-10-16-0-10.ec2.internal`) que solo resuelve dentro de la VPC, y —si tiene IP pública— uno público (`ec2-3-89-7-136.compute-1.amazonaws.com`).

> **Cae en el examen:** el nombre **público** resuelve **distinto según desde dónde preguntes**: dentro de la VPC → la **IPv4 privada**; fuera → la **IPv4 pública**. Es posible porque la IP pública no está pegada a la interfaz: la traduce el **Internet Gateway**. Así un mismo nombre DNS sirve adentro y afuera, lo que simplifica las redes híbridas.

#### Elastic IP

Una IPv4 pública **fija asignada a tu cuenta**, no a una instancia. Primero se hace *allocate* y después se **asocia** a una IP privada de una ENI.

- Al asociarla a la ENI **primaria**, la IP pública dinámica que tenía **se elimina** y la Elastic IP ocupa su lugar.
- Al **desasociarla**, la instancia recibe automáticamente una nueva IP pública dinámica.

> El curso dice que AWS cobra por una Elastic IP **ociosa** (asignada pero no asociada, o asociada a una instancia detenida).

> ⚠️ Outdated (curso): desde **febrero de 2024** AWS cobra por **toda IPv4 pública**, esté en uso o no — también la Elastic IP asociada a una instancia corriendo y la IP dinámica auto-asignada. El razonamiento del curso (desalentar el acaparamiento de un recurso escaso) sigue valiendo, pero **ya no hay IPv4 pública gratis**. En la práctica conviene evitar IPs públicas cuando alcanza con salir por un NAT Gateway o llegar por [[vpc-endpoints]].

### AMI (Amazon Machine Image)

Imagen para crear instancias (o creada desde una instancia). Contiene:

1. **Permisos** — public / owner (implícito, no removible) / explicit (cuentas específicas, patrón [[golden-ami|golden AMI]]).
2. **Root volume** — el volumen de arranque (siempre ≥1).
3. **Block device mapping** — qué volumen es boot y cuál datos, y su mapeo a `/dev/xvda` etc.

> Una AMI no es un archivo: es **metadata que apunta a EBS snapshots** + permisos + block device mapping. Por eso **borrar la AMI no borra sus snapshots** (te los siguen cobrando).

> Las AMIs son **regionales**: el `ami-xxxx` solo vale en su region. Para otra region → **Copy AMI** (copia snapshots, genera costo). Por eso los templates de [[CloudFormation]] resuelven el AMI ID con `Mappings` o **SSM Parameters** en vez de hardcodearlo.

**Fuentes y costo:** una AMI puede venir de **AWS**, de la **comunidad** o del **Marketplace**. Las del Marketplace traen software comercial preinstalado, y ahí el costo total es **instancia (cómputo) + licencia del software**.

#### AMI lifecycle ([[golden-ami|AMI baking]])

```
1. Launch     → una instancia desde una AMI base
2. Configure  → instalás y dejás todo como querés
3. Create image → nueva AMI (snapshots de sus volúmenes + block device mapping)
4. Re-launch  → todas las instancias que necesites, ya configuradas
```

Una AMI **no se edita**: para cambiar algo se lanza, se ajusta y se hornea una nueva.

### Status checks y auto-recovery

Cada instancia tiene **dos** verificaciones; lo sano es **2/2 checks passed**:

| Check | Qué valida | Un fallo sugiere |
|---|---|---|
| **System Status** | El servicio EC2 y el **host físico** | Energía, red, hardware o software del host |
| **Instance Status** | La **propia instancia** | File system corrupto, red mal configurada adentro |

Ante un fallo de System Status, EC2 puede hacer **auto-recovery**: mueve la instancia a un **host nuevo** y la reinicia con **la misma configuración y las mismas IPs**.

> ⚠️ Outdated (curso): el curso lo presenta como algo que hay que configurar. Desde **marzo de 2022** el auto-recovery está **habilitado por defecto** en la mayoría de los instance types modernos. La configuración manual (vía alarma de [[CloudWatch]]) sigue existiendo para desactivarlo o elegir otra acción.

> Como mueve la instancia de host, el **instance store se pierde**. Por eso el auto-recovery no aplica a instancias cuyo almacenamiento principal sea local.

### Bootstrapping: user data vs AMI baking

El **user data** es un script que la instancia ejecuta **como root, una sola vez, en el primer launch** (lo corre [[cloud-init]]). Máximo **16 KB**, se lee de `169.254.169.254/latest/user-data` y **no es seguro** para secretos. Si falla, la instancia igual queda `running`. Frente al [[golden-ami|AMI baking]], es más flexible pero más lento: lo óptimo es hornear la parte pesada y configurar el resto con user data. Detalle completo en [[ec2-bootstrapping]].

### Placement groups

Por defecto AWS decide en qué host va cada instancia. Un placement group lo cambia: **cluster** (mismo rack, una AZ, 10 Gbps single-stream, poca resiliencia), **spread** (un rack por instancia, máximo 7 por AZ) o **partition** (7 particiones por AZ, instancias ilimitadas, para apps [[topology-aware]]). Ver [[placement-groups]].

### Rendimiento de red y de EBS

- **[[enhanced-networking|Enhanced networking]]** (SR-IOV): cada instancia tiene su tarjeta lógica, lo que da más ancho de banda, más PPS y latencia baja y constante. **ENA** llega a 100 Gbps; **EFA** es para [[hpc|HPC]]/MPI. Viene por defecto en los tipos modernos y es requisito del cluster placement group.
- **[[ebs-optimized|EBS optimized]]**: capacidad de red dedicada para EBS, separada del tráfico de datos. Hoy viene habilitado por defecto y sin costo.

### Conectarse

| SO | Protocolo | Puerto |
|---|---|---|
| Linux | SSH | 22 |
| Windows | RDP | 3389 |

**Key pairs**: la public key la tiene la instancia (`~/.ssh/authorized_keys`); la private, vos (`.pem`). Linux → SSH directo (`chmod 400` o SSH rechaza la clave). Windows → la private key **desencripta la password** del administrador local, y entrás por RDP.

- Los key pairs son **regionales**: uno de `us-east-1` no existe en `sa-east-1` (crear/importar en cada region).
- AWS guarda **solo la clave pública**; la privada se descarga **una única vez** — si la perdés, AWS no la recupera.
- **Usuario por defecto según la AMI** (error clásico: `ssh root@` rebota):

| AMI | Usuario |
|---|---|
| Amazon Linux / 2 / 2023 | `ec2-user` |
| Ubuntu | `ubuntu` |
| Debian | `admin` |
| RHEL / CentOS / SUSE | `ec2-user` (o `root`/`centos`) |

- **¿Perdiste la `.pem`?** Detener instancia → desattachar volumen raíz → attachar a otra instancia → editar `authorized_keys` → volver a attachar. (O directamente **SSM Session Manager**.)

![[Pasted image 20260614021052.png]]

## Integración con otros servicios

- [[VPC]] — la instancia vive en una subnet; su tráfico lo filtran los security groups (en su [[eni|ENI]]) y la NACL de la subnet ([[security-groups-vs-nacls]]). Para salir a internet sin IP pública: NAT Gateway ([[nat-gateway-vs-nat-instance]]); para llegar a servicios de AWS sin internet: [[vpc-endpoints]].
- [[EBS]] — el almacenamiento persistente; el ancho de banda hacia EBS depende del instance type ([[ebs-optimized]]).
- [[IAM]] — instance roles: [[temporary-credentials|credenciales temporales]] vía [[instance-profile]], entregadas por el [[ec2-instance-metadata|IMDS]], sin access keys en disco.
- [[KMS]] — cifrado de los volúmenes EBS.
- [[CloudWatch]] — métricas nativas (CPU, red) y los status checks; RAM/disco requieren el **CloudWatch Agent**, que también manda los logs del SO a [[CloudWatchLogs]].
- [[SSMParameterStore]] — configuración y secretos que la instancia lee al arrancar con su instance role, en lugar de ponerlos en el user data.
- [[S3]] — donde viven los snapshots de EBS y las AMIs.
- [[ECS]] — en EC2 mode, las instancias son los container hosts (container instances) del cluster; para correr [[containers]] sin administrar instancias está Fargate ([[ecs-ec2-vs-fargate]]).
- [[RDS]] — la alternativa gestionada a instalar la base de datos en la instancia. Poner la base en EC2 solo se justifica con un motor o versión no soportada o con acceso al SO ([[databases-on-ec2]]).

## Gotchas y trampas del examen

> Repaso de todo el tema en una pantalla: [[ec2-cheat-sheet]].

- "Los datos deben sobrevivir reinicios" → **EBS**; "cache efímero ultrarrápido" → **Instance Store**.
- **Stop/start puede cambiar de host físico** → se pierde el instance store y **cambia la IP pública dinámica** (la privada no). Un **reboot no**.
- `Terminated` no tiene vuelta atrás.
- **Una instancia, su ENI y su volumen EBS no cruzan de AZ.** Para mover datos entre AZs: snapshot.
- Los **security groups se asocian a la ENI**, no a la instancia, aunque la consola lo muestre de otro modo.
- Para que una instancia haga de NAT hay que **desactivar el source/destination check**.
- El nombre DNS público hace **split-horizon**: resuelve a la IP privada dentro de la VPC.
- Asociar una **Elastic IP** a la ENI primaria **borra** la IP pública dinámica anterior; desasociarla asigna una nueva.
- Detener la instancia **no frena el costo de EBS** ni el de las IPv4 públicas.
- Una **AMI no se edita** y **borrarla no borra sus snapshots**.
- Auto-recovery **mueve de host** → el instance store se pierde.
- Acceso administrativo sin abrir puertos ni gestionar llaves → **SSM Session Manager** (la respuesta "mejor práctica").
- Credenciales dentro de la instancia → **[[ec2-instance-metadata|IMDS]]**, y para protegerlas de un SSRF → **IMDSv2**.
- El **user data** corre **solo en el primer launch**. Si falla, la instancia sigue `running` y pasa los checks. Nunca pongas secretos ahí ([[ec2-bootstrapping]]).
- "Menor tiempo hasta estar en servicio" → **AMI baking**; con flexibilidad → baking + user data.
- Cluster placement group = **una sola AZ**, poca resiliencia; spread = **máximo 7 por AZ**; más de 7 con aislamiento → **partition** ([[placement-groups]]).

## Demos del curso

- [My first EC2 Instance — PART 1](https://learn.cantrill.io/courses/1101194/lectures/64085773)
- [My first EC2 Instance — PART 2](https://learn.cantrill.io/courses/1101194/lectures/64085774)
- [EC2 SSH vs EC2 Instance Connect](https://learn.cantrill.io/courses/1101194/lectures/27806428)
- [Instalación manual de WordPress — Parte 1](https://learn.cantrill.io/courses/1101194/lectures/27806465)
- [Instalación manual de WordPress — Parte 2](https://learn.cantrill.io/courses/1101194/lectures/27806466)
- [Creating an Animals4life AMI — Parte 1](https://learn.cantrill.io/courses/1101194/lectures/27806468)
- [Creating an Animals4life AMI — Parte 2](https://learn.cantrill.io/courses/1101194/lectures/29064547)
- [Copy & Sharing an AMI](https://learn.cantrill.io/courses/1101194/lectures/27806469)
- [Status Check y Auto Recovery](https://learn.cantrill.io/courses/1101194/lectures/27806478)
- [Shutdown, Terminate & Termination Protection](https://learn.cantrill.io/courses/1101194/lectures/27806479)
- [WordPress installation con user data — Part 1](https://learn.cantrill.io/courses/1101194/lectures/27895409) · [Part 2](https://learn.cantrill.io/courses/1101194/lectures/29447330) (ver [[ec2-bootstrapping]])

> 📖 Lectura profunda del módulo 09: [[09.00 Advanced EC2 — Índice|Advanced EC2]] (bootstrapping, instance roles, Parameter Store, CloudWatch Agent, placement groups, enhanced networking, EBS optimized)
