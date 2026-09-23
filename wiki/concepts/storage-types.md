---
title: Tipos de almacenamiento en AWS
category: concept
tags: [storage, ebs, s3, efs, performance, iops]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.04 Storage Refresh.md"]
updated: 2026-09-22
---

# Tipos de almacenamiento en AWS

## Definición

Antes de elegir un servicio de almacenamiento hay que separar **tres ejes** que se confunden todo el tiempo: **cómo se conecta**, **cuánto dura** y **cómo se presenta**.

### Cómo se conecta

| | **DAS** (Direct Attached Storage) | **NAS** (Network Attached Storage) |
|---|---|---|
| Qué es | Discos físicos pegados al equipo | Volúmenes creados aparte, conectados **por red** |
| En AWS | **Instance Store** (discos del EC2 host) | **[[EBS]]** |
| Rendimiento | El más alto: sin salto de red | Muy bueno, pero pasa por la red |
| Riesgo | Falla el disco o el host → se pierde | Sobrevive a los problemas del host |

### Cuánto dura

- **[[ephemeral-storage|Ephemeral]]**: temporal, atado al hardware. No podés confiar en que persista. → Instance Store.
- **Persistente**: existe como recurso propio y vive **más allá** del dispositivo al que está conectado. → EBS.

> **Instance Store = efímero. EBS = persistente.** Es el mismo eje que aparece en el ciclo de vida de las instancias de [[EC2]].

### Cómo se presenta

| Categoría | Qué te entrega | ¿Booteable? | Servicio en AWS |
|---|---|---|---|
| **Block** | Bloques direccionables en crudo; el OS pone el file system encima | **Sí** | [[EBS]], Instance Store |
| **File** | Un file system ya armado, con carpetas para navegar | No | EFS |
| **Object** | [[object-storage\|Objetos]] planos con key y metadatos | No | [[S3]] |

- **Block storage** no trae estructura propia: le presentás el volumen al servidor y es el **sistema operativo** el que crea el file system (NTFS, ext4, XFS) y lo monta como `C:` o como volumen raíz. La mayoría de las instancias EC2 usan un volumen EBS como **boot volume**.
- **File storage** es lo que da un file server tradicional: ya viene con la estructura hecha. Por eso mismo no es booteable.
- **Object storage** es lo más abstracto: una colección plana, sin jerarquía, donde recuperás cada objeto por su **key**. Escala enormemente y lo pueden leer millones de clientes a la vez, pero en general **no se monta** y **no es booteable**.

## Cómo aplica en AWS

### Rendimiento: las tres métricas

- **IO (block) size** — el tamaño de cada operación de lectura/escritura.
- **[[iops|IOPS]]** — operaciones por segundo.
- **[[throughput]]** — datos por segundo, en MB/s.

Se relacionan por una cuenta simple:

> **IO block size × IOPS = throughput**

Con bloques de 16 KB y 100 IOPS: 16 × 100 = 1.600 KB/s ≈ **1,6 MB/s**.

### El tamaño de bloque depende del tipo de disco

| Tipo | Tamaño de bloque | Efecto |
|---|---|---|
| SSD de EBS (gp2/gp3, io1/io2) | **16 KB** | Muchas operaciones chicas → se miden en IOPS |
| HDD de EBS (st1/sc1) | **1 MB** | 500 IOPS = **500 MB/s** → se miden en throughput |

Ese salto explica por qué `st1` publica su rendimiento en **MB/s por TB** y no en IOPS, y por qué comparar "IOPS" entre un SSD y un HDD sin mirar el bloque lleva a conclusiones equivocadas.

## Patrones comunes

```
¿Qué necesito?
├── Bootear un sistema operativo        → block (EBS SSD)
├── Que varios servidores compartan     → file (EFS)
├── Guardar archivos/backups, web       → object (S3)
├── Máximo rendimiento, datos tirables  → block efímero (Instance Store)
└── Datos que tienen que sobrevivir     → block persistente (EBS)
```

## Preguntas de examen frecuentes

- *"Necesito que varias instancias monten el mismo file system"* → **EFS** (file). EBS se adjunta a una instancia a la vez en el caso típico.
- *"Bootear desde esto"* → tiene que ser **block**, y además **SSD**: los HDD `st1`/`sc1` no sirven como boot volume ([[ebs-volume-types]]).
- *"Millones de clientes leyendo los mismos archivos por HTTP"* → **S3** (object), no EBS.
- Cuenta típica: un volumen con bloques de 16 KB y tope de 3.000 IOPS da ~47 MB/s — puede que choques con el límite de IOPS mucho antes que con el de throughput, o al revés.

## Ver también

[[EBS]] · [[S3]] · [[instance-store-vs-ebs]] · [[iops]] · [[throughput]] · [[ephemeral-storage]]
