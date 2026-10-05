---
title: EFS (Elastic File System)
category: service
tags: [efs, storage, file-storage, nfs, linux, lambda, vpc]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/11 Advanced Storage/11.01 EFS — Architecture.md", "raw/notas curso mejorado/11 Advanced Storage/11.02 EFS — Performance, Throughput y Storage Classes.md", "raw/notas curso mejorado/11 Advanced Storage/11.03 Demostración - Implementación de EFS.md", "raw/notas curso mejorado/11 Advanced Storage/11.08 Comparativa - EFS vs FSx for Windows vs FSx for Lustre.md"]
updated: 2026-10-05
---

# EFS — Elastic File System

## ¿Qué es?

El file storage compartido de AWS para Linux: una implementación gestionada de **[[nfs|NFS]] v4**. Creás un **file system**, que vive **separado** de cualquier instancia y tiene ciclo de vida propio, y lo montás en una carpeta de **muchos clientes Linux a la vez**: instancias EC2, Lambda y contenedores de ECS/EKS (también macOS). Todos ven y escriben los mismos datos, sin tener que sincronizar nada entre instancias.

## Casos de uso

- Varias instancias EC2 que necesitan **los mismos archivos**: contenido estático de una web, uploads de un CMS, home directories.
- **Storage persistente y compartido para containers** ([[ECS]], también en Fargate, y [[EKS]]): lo que el container escribe en su capa R/W es [[ephemeral-storage|efímero]] y se pierde con él.
- **Lambda** que necesita más espacio que `/tmp`, o compartir datos **entre invocaciones** y entre funciones.
- Cualquier file share de Linux que tenga que sobrevivir a la caída de una AZ (despliegue Regional).

## Características clave

### Arquitectura: file system + mount targets

- EFS se aprovisiona **dentro de una VPC**, así que es un **servicio privado**. Desde afuera se llega solo por redes híbridas: VPC peering, VPN o Direct Connect.
- El file system se expone mediante **mount targets**, uno por AZ. Cada uno vive en una subnet, toma una **IP privada** de su rango y tiene su **security group**.
- Para alta disponibilidad: **un mount target en cada AZ** donde haya clientes.
- Los permisos son **[[posix|POSIX]]**, los mismos que entiende cualquier Linux.
- Desde **on-premises** se monta con la **IP del mount target**, no con el nombre DNS de EFS. Ese nombre lo resuelve el DNS de la VPC, y una red on-premises no lo ve salvo que tengas DNS híbrido.

![[Pasted image 20261004234702.png]]

### Regional vs One Zone

| | Regional (default, recomendado) | One Zone |
|---|---|---|
| Dónde están los datos | Varias AZs | Una sola AZ |
| Mount targets | Uno por AZ | Uno |
| Resiliencia | [[region-resilient\|Region resilient]] | [[az-resilient\|AZ resilient]]: si cae la AZ, no hay acceso |
| Costo | Más caro | Más barato |

### Performance modes

| Modo | Para qué | Nota |
|---|---|---|
| **General Purpose** | Sensible a la latencia: web servers, CMS, home directories | **Default** y lo recomendado para casi todo |
| **Max I/O** | Muy paralelo: big data, media, análisis científico | **Legacy**. Más operaciones agregadas, pero con **más latencia** |

### Throughput modes

| Modo | Cómo funciona |
|---|---|
| **Elastic** (default) | Escala solo con la carga. Pagás lo que leés y escribís; no hay nada que aprovisionar ni créditos |
| **Bursting** | El [[throughput]] escala con el **tamaño** del file system, con [[burst-credit\|créditos de burst]]. Los créditos son de throughput, **no de [[iops\|IOPS]]** como en gp2 |
| **Provisioned** | Fijás el throughput **independientemente del tamaño**, aunque el file system sea chico |

Los dos ejes no son independientes: **Elastic fuerza General Purpose**, y Max I/O solo existe con Bursting o Provisioned.

### Storage classes y lifecycle

| Clase | Para qué |
|---|---|
| **Standard** (default) | Datos de uso diario |
| **Infrequent Access (IA)** | Un par de accesos por trimestre |
| **Archive** | Pocos accesos al año; la más barata |

Como en [[s3-storage-classes|S3]], las **lifecycle policies** mueven los datos Standard → IA → Archive. *(Complemento, no viene de la slide)*: el criterio son los **días desde el último acceso** (por ejemplo, 30 días sin acceso → IA), y IA y Archive cobran un **retrieval fee por GB leído**.

### Access points *(complemento: el curso no los cubre)*

Un **access point** es una puerta de entrada al file system por aplicación. Impone una **identidad POSIX** (UID/GID) y un **directorio raíz** propio para todo lo que entra por él. Combinado con IAM policies, le da a cada aplicación (varias Lambdas, varios containers) una vista aislada del mismo file system. Lambda monta EFS **siempre a través de un access point**.

## Integración con otros servicios

- [[EC2]]: el cliente clásico. Se monta con `amazon-efs-utils` y una línea en `/etc/fstab`, con `tls` para cifrar en tránsito (ver la demo).
- **Lambda** *(complemento, doc de Lambda)*: la función tiene que estar **conectada a la VPC** del file system ([[lambda-in-vpc]]). Monta a través de un access point en una ruta que empieza con **`/mnt/`**, y su [[execution-role|execution role]] necesita los permisos de VPC. Conviene tener mount targets en todas las AZs que use la función.
- [[ECS]] / [[EKS]]: volúmenes persistentes compartidos entre tasks o pods. En Fargate es la opción natural.
- [[VPC]]: mount targets en subnets y security groups. El SG del mount target tiene que aceptar **TCP 2049** desde el SG del cliente.
- [[KMS]]: cifrado en reposo *(complemento)*.

## Gotchas y trampas del examen

- **"La instancia no puede montar el EFS"** → el security group del mount target no permite **2049 (NFS)** desde el SG de la instancia.
- **"Varias instancias necesitan el mismo file system"** → EFS. Un volumen [[EBS]] se conecta a una sola instancia en el caso típico, y [[S3]] **no se monta** como file system.
- **EFS es Linux (NFS).** Si el escenario dice Windows, [[smb|SMB]] o [[active-directory|Active Directory]] → [[FSx]] for Windows ([[efs-vs-fsx]]).
- **One Zone ≠ resiliente.** Más barato, pero no sobrevive a la pérdida de la AZ.
- **Max I/O es legacy.** Si una opción propone Max I/O para un web server o un CMS, es distractor: General Purpose.
- **Lambda + EFS** exige que la función esté en la VPC, un access point y el puerto 2049 abierto en los dos SGs.
- **On-premises** monta por la **IP** del mount target, no por el DNS de EFS.
- IA y Archive son baratos para guardar, pero **cobran por leer**: no sirven para datos que se acceden seguido.
- El lifecycle **se parece** al de S3 pero no es igual: EFS cuenta **días desde el último acceso**, mientras que las transiciones de S3 cuentan **días desde la creación** del objeto ([[s3-storage-classes]]).

## Demos del curso

- [Implementación de EFS — Parte 1](https://learn.cantrill.io/courses/3028179/lectures/66797497)
- [Implementación de EFS — Parte 2](https://learn.cantrill.io/courses/3028179/lectures/66797498)

Un mismo file system montado en dos instancias EC2 vía `/etc/fstab` (`efs _netdev,tls`). Lo que escribe una instancia lo ve la otra. Los comandos están en [[11.03 Demostración - Implementación de EFS]].

## Ver también

[[FSx]] · [[efs-vs-fsx]] · [[storage-types]] · [[EBS]] · [[lambda-in-vpc]] · [[nfs]] · [[posix]]

> Fuente: raw/notas curso mejorado/11 Advanced Storage/ (11.01–11.03, 11.08). Complemento sobre Lambda: [doc de Lambda — Configuring Amazon EFS file system access](https://docs.aws.amazon.com/lambda/latest/dg/configuration-filesystem-efs.html).
