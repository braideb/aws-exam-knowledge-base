---
title: FSx (FSx for Windows File Server y FSx for Lustre)
category: service
tags: [fsx, storage, file-storage, windows, smb, active-directory, lustre, hpc]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/11 Advanced Storage/11.04 FSx for Windows File Server.md", "raw/notas curso mejorado/11 Advanced Storage/11.05 FSx — Key Features and Benefits.md", "raw/notas curso mejorado/11 Advanced Storage/11.06 FSx for Lustre — Concepts.md", "raw/notas curso mejorado/11 Advanced Storage/11.07 FSx for Lustre — Storage, Performance y Arquitectura.md", "raw/notas curso mejorado/11 Advanced Storage/11.08 Comparativa - EFS vs FSx for Windows vs FSx for Lustre.md"]
updated: 2026-10-05
---

# FSx — File systems gestionados de terceros

## ¿Qué es?

Familia de file systems **gestionados** que implementan un file system ya existente, con los servidores ocultos, como hace RDS con las bases de datos. El curso cubre dos variantes:

- **FSx for Windows File Server**: file shares **nativos de Windows**, accedidos por **[[smb|SMB]]** e integrados con **[[active-directory|Active Directory]]**.
- **FSx for Lustre**: el file system **Lustre**, para **[[hpc|HPC]]** y machine learning, con clientes **Linux** y permisos **[[posix|POSIX]]**, y opcionalmente respaldado por un bucket de [[S3]].

*(Complemento: el curso no las cubre)* La familia incluye además **FSx for NetApp ONTAP** y **FSx for OpenZFS**. [[EKS]] las lista entre sus storage providers.

## Casos de uso

| Variante | Cuándo |
|---|---|
| FSx for Windows | Apps y usuarios Windows que necesitan file shares con AD, permisos de Windows, DFS; escritorios de **WorkSpaces** |
| FSx for Lustre | ML, modelado financiero, simulaciones, procesamiento masivo de datos que hoy viven en S3 |

---

## FSx for Windows File Server

### Características clave

- **Nativo**, no emulado: **SMB**, el **modelo de permisos de Windows**, **DFS** (Distributed File System, para agrupar y escalar shares a nivel empresa) y **deduplicación** a nivel sub-archivo.
- **Active Directory obligatorio** como user store. Puede ser **managed** (AWS Directory Service) o **self-managed**, incluido el **AD on-premises** que la empresa ya tiene.
- **Single-AZ** o **[[multi-az|Multi-AZ]]**, con acceso por **[[eni|ENIs]]** en la VPC. Aun en Single-AZ replica dentro de la AZ para tolerar fallas de hardware; Multi-AZ da HA entre AZs.
- **Backups** automáticos y on-demand.
- Cifrado en reposo con [[KMS]] y la opción de **forzar el cifrado en tránsito**.
- Accesible desde la VPC y desde redes conectadas: peering, VPN, Direct Connect.
- Se accede con la ruta UNC de siempre: `\\fs-xxx123.animals4life.org\catpics`.
- **VSS (Volume Shadow Copies)**: el usuario hace clic derecho → "versiones anteriores" y **restaura archivos él mismo**, sin un admin y sin tocar AWS.

### Rendimiento

El curso da **8 MB/s a 2 GB/s**, cientos de miles de [[iops|IOPS]] y latencia **< 1 ms**.

> ⚠️ Outdated: desde 2023 el máximo es **12 GB/s (12.288 MB/s)** en varias regiones (N. Virginia, Ohio, Oregon, Irlanda, Tokio, Singapur); 2 GB/s sigue siendo el tope en el resto. Las IOPS sobre disco llegan a ~350.000–400.000. ([anuncio de AWS](https://aws.amazon.com/about-aws/whats-new/2023/08/amazon-fsx-windows-file-server-increases-throughput-12-gbs/), [doc de performance](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/performance.html))

![[Pasted image 20261005013117.png]]

---

## FSx for Lustre

### Deployment types

| | Scratch | Persistent |
|---|---|---|
| Para qué | Rendimiento puro, cargas **cortas o temporales** | Almacenamiento de **largo plazo** |
| Replicación / HA | **Ninguna**: si falla hardware, sus datos se pierden | Replicación **dentro de una sola AZ** + **self-healing** |
| Caída de la AZ | Se pierde | **También** se pierde: no es multi-AZ |
| Baseline | **200 MB/s por TiB** | **50, 100 o 200 MB/s por TiB** |
| Burst | Hasta **1.300 MB/s por TiB** con [[burst-credit\|créditos]] | Igual |

Cuanto más grande es un file system Scratch, más servidores y discos tiene, y por eso **más probabilidad de falla**.

> ⚠️ Outdated (parcial): los valores 50/100/200 MB/s/TiB son de **Persistent_1**. La generación actual, **Persistent_2** (SSD), ofrece **125, 250, 500 o 1.000 MB/s/TiB**. ([doc de FSx for Lustre](https://docs.aws.amazon.com/fsx/latest/LustreGuide/managing-throughput-capacity.html))

### El repositorio S3: lazy load y sin sincronización

- Al crear el file system podés asociarlo a un **bucket de S3** como repositorio.
- Los objetos **aparecen** en el file system pero **no están ahí**: se traen con **[[lazy-loading|lazy load]]** la primera vez que un cliente los lee.
- **No hay sincronización automática.** Los cambios vuelven a S3 solo si los **exportás** (`hsm_archive`).
- El procesamiento ocurre en el file system Lustre; S3 es solo la base.

![[Pasted image 20261005020311.png]]

### Cómo almacena y entrega los datos

- **Metadata** (nombres, timestamps, permisos) → un **Metadata Target (MDT)**.
- **Datos** → repartidos en **Object Storage Targets (OST)** de **1,17 TiB**. De ese reparto sale el rendimiento.
- Tamaño mínimo **1,2 TiB**, luego incrementos de **2,4 TiB**. El rendimiento es un baseline **por TiB**, así que más storage = más servidores = más [[throughput]].
- Cada **storage server** tiene **caché en memoria**. Las escrituras y las primeras lecturas dependen del disco; las lecturas repetidas salen de la caché y dependen de la red.
- Corre en **una sola AZ** ([[az-resilient|AZ resilient]]) y llega a la VPC por **una única ENI**.
- Escala a **cientos de GB/s** con latencia **< 1 ms**.
- **Backups a S3**, manuales y automáticos, con retención de **0 a 35 días**.

![[Pasted image 20261005023353.png]]

---

## Integración con otros servicios

- [[S3]]: repositorio de Lustre (lazy load + export) y destino de sus backups.
- [[VPC]]: ENIs en tus subnets; acceso híbrido con VPN o Direct Connect.
- [[KMS]]: cifrado en reposo de FSx for Windows.
- **AWS Directory Service / AD on-premises**: el user store de FSx for Windows.
- **WorkSpaces**: escritorios virtuales que usan FSx for Windows como file share.
- [[EKS]]: FSx for Lustre (y ONTAP) como persistent storage de pods.

## Gotchas y trampas del examen

- **SMB, Windows, Active Directory, DFS, "versiones anteriores"** → **FSx for Windows**. **[[nfs|NFS]] / Linux compartido** → [[EFS]].
- **HPC, ML, "procesar datos de S3 a muy alto rendimiento"** → **FSx for Lustre**.
- **Lustre Persistent no es multi-AZ.** Protege ante fallas de **hardware**, no ante la caída de la **AZ**. Scratch no replica nada.
- **Lustre no se sincroniza con S3.** Lo nuevo en el bucket se ve, pero se carga recién en el primer acceso. Lo que escribís en Lustre no vuelve solo a S3: hay que exportarlo.
- FSx for Windows puede usar el **AD on-premises**: no hace falta migrar el directorio a AWS.
- FSx for Windows y Lustre son **POSIX vs permisos de Windows**: el tipo de permisos también delata la respuesta.

## Ver también

[[EFS]] · [[efs-vs-fsx]] · [[storage-types]] · [[hpc]] · [[smb]] · [[active-directory]] · [[lazy-loading]]

> Fuente: raw/notas curso mejorado/11 Advanced Storage/ (11.04–11.08). Correcciones de rendimiento con doc oficial de AWS (sin clipping en `raw/`).
