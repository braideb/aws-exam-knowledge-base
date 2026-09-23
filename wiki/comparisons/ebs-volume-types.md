---
title: Tipos de volumen EBS (gp2 / gp3 / io1 / io2 / st1 / sc1)
category: comparison
tags: [ebs, storage, iops, throughput, performance]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.06 EBS Volume Types - General Purpose SSD (gp2 y gp3).md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.07 EBS Volume Types - Provisioned IOPS SSD (io1, io2, Block Express).md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.08 EBS Volume Types - HDD (st1 y sc1).md"]
updated: 2026-09-22
---

# Tipos de volumen EBS

Seis tipos vigentes de [[EBS]], en tres familias. El eje que los separa es **cómo se obtiene el rendimiento**: ligado al tamaño (gp2, HDD), fijo (gp3) o aprovisionado aparte (io1/io2).

## La tabla

| | **gp2** | **gp3** | **io1** | **io2** | **io2 Block Express** | **st1** | **sc1** |
|---|---|---|---|---|---|---|---|
| Medio | SSD | SSD | SSD | SSD | SSD | HDD | HDD |
| Tamaño | 1 GB – 16 TB | 1 GB – 16 TB | 4 GB – 16 TB | 4 GB – 16 TB | 4 GB – **64 TB** | 125 GB – 16 TB | 125 GB – 16 TB |
| [[iops\|IOPS]] máx. | **16.000** | **16.000** | **64.000** | **64.000** | **256.000** | 500 | 250 |
| [[throughput\|Throughput]] máx. | **250 MB/s** | **1.000 MB/s** | 1.000 MB/s | 1.000 MB/s | **4.000 MB/s** | 500 MB/s | 250 MB/s |
| Rendimiento base | 3 IOPS/GB (mín. 100) con [[burst-credit\|créditos]] | **3.000 IOPS + 125 MB/s fijos** | Aprovisionado | Aprovisionado | Aprovisionado | 40 MB/s por TB | 12 MB/s por TB |
| Ratio máx. IOPS/GB | 3 | — (independiente) | **50** | **500** | **1.000** | — | — |
| ¿Créditos de burst? | **Sí** | No | No | No | No | Sí (250 MB/s por TB) | Sí (80 MB/s por TB) |
| ¿Bootea? | **Sí** | **Sí** | **Sí** | **Sí** | **Sí** | **No** | **No** |
| Tamaño de bloque | 16 KB | 16 KB | 16 KB | 16 KB | 16 KB | **1 MB** | **1 MB** |

## Cuándo usar cada uno

### gp2 vs gp3 — los de propósito general

**gp2** funciona con un **balde de [[burst-credit|créditos]]**: se rellena a **3 IOPS por GB** (mínimo 100), el balde arranca lleno con **5,4 millones** de créditos y permite ráfagas de hasta **3.000 IOPS**. Mientras haya fichas vas rápido; cuando se vacían caés al baseline.

Dos umbrales que hay que saber:
- **1 TB** — a 1.000 GB el baseline es 1.000 × 3 = 3.000 IOPS, justo el tope de burst. Por encima de 1 TB **el sistema de créditos deja de importar**.
- **~5,33 TB** — donde gp2 alcanza su tope de 16.000 IOPS.

**gp3** elimina los créditos: **3.000 IOPS y 125 MB/s fijos para cualquier tamaño**, con opción de pagar hasta 16.000 IOPS y 1.000 MB/s. Es ~**20% más barato** que gp2 en el precio base.

> Si solo necesitás hasta 3.000 IOPS, **gp3 en vez de gp2 es una obviedad**: mismo rendimiento, más barato y sin baldes que vigilar.

> [!info] Actualización posterior al curso
> Lo que el curso anticipaba ya pasó: **gp3 es hoy la recomendación por defecto de AWS** para volúmenes de propósito general. Los números no cambiaron, y el modelo de créditos de gp2 sigue vigente — los dos siguen siendo material de examen.

### io1 / io2 / io2 Block Express — los de IOPS aprovisionadas

Lo que los define: las **IOPS se configuran de forma independiente del tamaño**, y la latencia es **baja y consistente** (lo predecible importa tanto como lo rápido). Pagás por **tamaño + IOPS aprovisionadas**.

El **ratio IOPS/GB** es lo que sube en cada variante: io1 hasta 50, io2 hasta 500, Block Express hasta 1.000. Ese ratio es lo que permite el caso de uso típico: **un volumen chico que necesita rendimiento altísimo**.

> [!info] Actualización posterior al curso
> Desde fines de 2023 todos los volúmenes io2 nuevos son en realidad **io2 Block Express**, así que en la práctica los dos se fusionaron. Para el examen alcanza con el modelo de tres variantes.

### st1 / sc1 — los HDD

Discos duros reales, con partes mecánicas. **El I/O se mide en bloques de 1 MB**, no de 16 KB: por eso 500 IOPS en un HDD son **500 MB/s**. También usan créditos.

- **st1 (Throughput Optimized)** — el HDD "rápido". Para acceso **secuencial**: big data, data warehouses, procesamiento de logs. Malo para acceso aleatorio.
- **sc1 (Cold HDD)** — el más barato de EBS. Para datos **fríos**, de acceso poco frecuente, con pocos escaneos por día.

Existe un tercer HDD, **Magnetic (standard)**, pero es *legacy* y no se usa.

## Trampa típica del examen

- **Solo los SSD botean.** Cualquier escenario que mencione boot volume o root volume descarta `st1` y `sc1` de entrada. Es el descarte más rentable de la comparación.
- **"El volumen se puso lento después de un rato"** → gp2 con el balde de créditos en cero. Solución: agrandar el volumen (más baseline) o migrar a gp3.
- **"Volumen chico con muchísimas IOPS"** → io1/io2. gp2 no puede: está atado a 3 IOPS/GB, así que para 16.000 IOPS necesitarías ~5,33 TB.
- **"Aprovisioné 64.000 IOPS pero mido menos"** → el cuello de botella es la **instancia**, no el volumen ([[ebs-optimized]]). Hay un tope de ~260.000 IOPS por instancia, y para acercarse hace falta combinar varios volúmenes.
- **"Secuencial / streaming / logs / lo más barato"** → HDD. Si además dice "acceso poco frecuente" o "datos fríos" → **sc1**; si dice throughput alto → **st1**.
- Comparar IOPS entre un SSD y un HDD sin mirar el tamaño de bloque lleva a conclusiones equivocadas: 500 IOPS de HDD mueven más datos que 3.000 de SSD.

## Ver también

[[EBS]] · [[instance-store-vs-ebs]] · [[storage-types]] · [[iops]] · [[throughput]] · [[burst-credit]]
