---
title: EFS vs FSx for Windows vs FSx for Lustre
category: comparison
tags: [efs, fsx, lustre, storage, file-storage, nfs, smb]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/11 Advanced Storage/11.08 Comparativa - EFS vs FSx for Windows vs FSx for Lustre.md", "raw/notas curso mejorado/11 Advanced Storage/11.01 EFS — Architecture.md", "raw/notas curso mejorado/11 Advanced Storage/11.04 FSx for Windows File Server.md", "raw/notas curso mejorado/11 Advanced Storage/11.06 FSx for Lustre — Concepts.md", "raw/notas curso mejorado/11 Advanced Storage/11.07 FSx for Lustre — Storage, Performance y Arquitectura.md"]
updated: 2026-10-05
---

# EFS vs FSx for Windows vs FSx for Lustre

Los tres son **file storage** compartido y gestionado ([[storage-types]]): un file system que muchos clientes montan a la vez, a diferencia de [[EBS]] (block, una instancia) y de [[S3]] (object, no se monta). Los tres viven en la VPC y se alcanzan desde on-premises por VPN o Direct Connect. Los separan el **protocolo**, el **sistema operativo** de los clientes y el **caso de uso**.

## Tabla comparativa

| | [[EFS]] | [[FSx]] for Windows File Server | [[FSx]] for Lustre |
|---|---|---|---|
| Protocolo | [[nfs\|NFS]] v4 | [[smb\|SMB]] | Cliente Lustre |
| Clientes | Linux (EC2, Lambda, ECS/EKS) y macOS | Windows | Linux |
| Permisos | [[posix\|POSIX]] | Modelo de Windows + [[active-directory\|Active Directory]] | [[posix\|POSIX]] |
| Caso típico | File share Linux compartido | File shares corporativos de Windows | [[hpc\|HPC]] / ML sobre datos de S3 |
| Resiliencia | Regional ([[multi-az\|multi-AZ]]) u One Zone | Single-AZ o Multi-AZ | **Siempre una AZ**: Scratch sin replicación, Persistent replicado en la AZ |
| Acceso en la VPC | Un mount target por AZ (SG, puerto 2049) | ENIs | Una [[eni\|ENI]] |
| Rendimiento | Elastic / Bursting / Provisioned | Hasta 2 GB/s (12 GB/s en algunas regiones) | Cientos de GB/s, < 1 ms |
| Integración clave | Lambda (access points), containers | AD, DFS, VSS, WorkSpaces | Repositorio S3 con [[lazy-loading\|lazy load]] |
| Palabras disparadoras | "NFS", "shared Linux file system" | "SMB", "Windows", "Active Directory", "DFS" | "HPC", "high performance", "machine learning", "Lustre" |

## Cuándo usar X vs Y

| Escenario | Respuesta |
|---|---|
| Varias instancias EC2 Linux comparten archivos | **EFS** |
| Lambda necesita storage persistente y compartido más allá de `/tmp` | **EFS** (con access point) |
| Containers de ECS/Fargate necesitan un volumen compartido | **EFS** |
| File shares para usuarios Windows con el AD de la empresa | **FSx for Windows** |
| Los usuarios restauran versiones anteriores de un archivo sin pedirle a un admin | **FSx for Windows** (VSS) |
| Entrenar un modelo leyendo un dataset enorme que está en S3 | **FSx for Lustre** (repositorio S3) |
| Procesamiento temporal donde solo importa la velocidad | **FSx for Lustre Scratch** |
| Datos HPC de largo plazo que tienen que sobrevivir a fallas de hardware | **FSx for Lustre Persistent** |

## Trampa típica del examen

- **"Shared file system" no alcanza para elegir.** Hay que buscar el sistema operativo o el protocolo: Linux/NFS → EFS; Windows/SMB/AD → FSx for Windows; HPC → Lustre.
- **POSIX aparece en dos columnas.** EFS y Lustre usan permisos POSIX; el desempate es el caso de uso (file share general vs HPC).
- **Lustre no es multi-AZ**, ni siquiera Persistent. Si el escenario exige sobrevivir a la caída de una AZ, Lustre no es la respuesta.
- **Montar S3 como file system** sigue siendo el distractor de siempre: S3 es [[object-storage|object storage]]. Lustre puede **mostrar** un bucket como file system, pero copia los datos al file system en el primer acceso.

## Ver también

[[EFS]] · [[FSx]] · [[storage-types]] · [[instance-store-vs-ebs]] · [[s3-storage-classes]]
