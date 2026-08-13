---
title: EC2 (Elastic Compute Cloud)
category: service
tags: [ec2, compute, iaas, ami, instancias, ebs]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.05 Elastic Compute Cloud (EC2) — Basics.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.06 Amazon Machine Image (AMI).md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.07 Conectarse a EC2.md"]
updated: 2026-07-23
---

# EC2 — Elastic Compute Cloud

## ¿Qué es?

Servicio de **máquinas virtuales (instancias)** — el [[iaas|IaaS]] clásico de AWS. La unidad de consumo es la instancia: un SO con recursos asignados. Vos gestionás SO y aplicaciones; AWS gestiona del [[hypervisor]] para abajo ([[shared-responsibility-model]]).

## Casos de uso

- Cargas de trabajo tradicionales que necesitan un servidor/SO completo.
- Apps monolíticas, bases de datos autogestionadas, workloads con requisitos específicos de SO.

## Características clave

- **Servicio privado**: corre en una subnet de una [[VPC]]; acceso público solo si se configura.
- **[[az-resilient|AZ resilient]]**: la instancia vive en una subnet → una AZ. Cae la AZ, cae la instancia.
- Facturación on-demand: **por segundo** (mínimo 60 s) para Linux/Ubuntu; **por hora** para algunas AMIs comerciales (Windows, RHEL). Size y capabilities se pueden cambiar (con la instancia **detenida**).
- **EC2 = "default compute" del examen**: la opción por defecto salvo buena razón. Cargas largas / SO a medida / monolitos → EC2. Eventos, ejecuciones cortas, "no quiero administrar servidores" → Lambda o gestionado.

### Estados y facturación

```
Running  ⇄  Stopped  →  Terminated
```

| Estado | Se cobra |
|---|---|
| **Running** | Todo (CPU, RAM, storage, red) |
| **Stopped** | Solo el almacenamiento **EBS** |
| **Terminated** | Nada — **irreversible** |

![[Pasted image 20260614013829.png]]

### Almacenamiento

| | **Instance Store** | **EBS** |
|---|---|---|
| Dónde vive | Discos físicos **en el host** | Volúmenes de **red**, independientes del host |
| Persistencia | **Efímero** | **Persistente** (sobrevive stop/start) |
| Performance | Rapidísimo (NVMe local) | Muy buena, pasa por red |
| Costo | Incluido en la instancia | Aparte, por GB/mes |
| Snapshots | No | Sí, se guardan en [[S3]] |
| Caso de uso | Caché, scratch, buffers | SO, bases de datos, datos que importan |

**Cuándo se pierde un Instance Store:** con **stop, hibernate, terminate o falla del host**. **No** se pierde con un **reboot** (mismo host físico). Ese es el matiz clave: *stop* cambia de host (perdés instance store + IP pública dinámica), *reboot* no.

### AMI (Amazon Machine Image)

Imagen para crear instancias (o creada desde una instancia). Contiene:
1. **Permisos** — public / owner (implícito, no removible) / explicit (cuentas específicas, patrón "golden AMI").
2. **Root volume** — el volumen de arranque (siempre ≥1).
3. **Block device mapping** — qué volumen es boot y cuál datos, y su mapeo a `/dev/xvda` etc.

> Una AMI no es un archivo: es **metadata que apunta a EBS snapshots** + permisos + block device mapping. Por eso **borrar la AMI no borra sus snapshots** (te los siguen cobrando).

> Las AMIs son **regionales**: el `ami-xxxx` solo vale en su region. Para otra region → **Copy AMI** (copia snapshots, genera costo). Por eso los templates de [[CloudFormation]] resuelven el AMI ID con `Mappings` o **SSM Parameters** en vez de hardcodearlo.

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

- [[VPC]] — la instancia vive en una subnet.
- [[IAM]] — instance roles ([[temporary-credentials|credenciales temporales]] vía [[instance-profile]], sin access keys).
- [[CloudWatch]] — métricas nativas (CPU, red); RAM/disco requieren el **CloudWatch Agent**.

## Gotchas y trampas del examen

- "Los datos deben sobrevivir reinicios" → **EBS**; "cache efímero ultrarrápido" → **Instance Store**.
- Stop/start puede cambiar de host físico → Instance Store se pierde, la IP pública dinámica cambia (la privada no).
- `Terminated` no tiene vuelta atrás.
- Acceso administrativo sin abrir puertos ni gestionar llaves → **SSM Session Manager** (la respuesta "mejor práctica").

## Demos del curso

- [My first EC2 Instance — PART 1](https://learn.cantrill.io/courses/1101194/lectures/64085773)
- [My first EC2 Instance — PART 2](https://learn.cantrill.io/courses/1101194/lectures/64085774)
