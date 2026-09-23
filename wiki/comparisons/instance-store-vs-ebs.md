---
title: Instance Store vs EBS
category: comparison
tags: [ec2, ebs, instance-store, storage, iops]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.09 Instance Store Volumes.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.10 Instance Store vs EBS.md"]
updated: 2026-09-22
---

# Instance Store vs EBS

Los dos son **block storage** que se presenta a la instancia como un disco en crudo. La diferencia de fondo es una sola: el **instance store son discos físicos del EC2 host** y [[EBS]] son **volúmenes por red**. De ahí sale todo lo demás.

| | **Instance Store** | **EBS** |
|---|---|---|
| Dónde vive | Discos físicos **del host** donde corre la instancia | Volúmenes de **red**, independientes del host |
| Persistencia | **[[ephemeral-storage\|Efímero]]** | **Persistente** |
| Rendimiento | El más alto de AWS (sin salto de red) | Muy bueno, pero pasa por la red |
| Cuándo se adjunta | **Solo al lanzar** la instancia | Cuando quieras; `attach`/`detach` en caliente |
| Costo | **Incluido** en el precio de la instancia | Aparte, por **GB-mes** |
| Snapshots | **No** | **Sí**, hacia [[S3]] |
| Cifrado | Sí (en las instancias modernas) | Sí, con [[KMS]] |
| Caso de uso | Caché, scratch, buffers, datos replicados | SO, bases de datos, datos que importan |

> **Instance Store = local + efímero + rapidísimo, se adjunta solo al lanzar.**
> **EBS = por red + persistente, se adjunta y desadjunta cuando quieras.**

## Cuándo se pierde un instance store

| Acción | ¿Se pierden los datos? | Por qué |
|---|---|---|
| **Reboot** | **No** | No cambia de host físico (pero hay que **volver a montar** el volumen) |
| **Stop + start** | **Sí** | La instancia puede mudarse a otro host, y ahí le asignan volúmenes nuevos |
| **Hibernate** | **Sí** | Igual que un stop |
| **Terminate** | **Sí** | — |
| **Falla del host** | **Sí** | Los discos eran de ese host |
| **Auto-recovery** | **Sí** | Mueve la instancia a un host nuevo ([[EC2]]) |

Los datos en **EBS persisten en todos** esos casos.

## Cuándo usar cada uno

| El requisito es… | La respuesta |
|---|---|
| **Persistencia** | **EBS** |
| **Resiliencia** | **EBS** |
| **Storage aislado del ciclo de vida de la instancia** | **EBS** |
| **Resiliencia con replicación hecha por la aplicación** | *Depende* |
| **Alto rendimiento** | *Depende* |
| **Rendimiento súper alto** | **Instance Store** |
| **Costo** | **Instance Store** (ya viene pago con la instancia) |

Los dos *"depende"* son a propósito: si la app ya replica sus datos por su cuenta, o si "alto rendimiento" todavía entra dentro de lo que da EBS, cualquiera de los dos puede ser correcto según el resto del escenario.

## La escalera de IOPS

La forma más rápida de resolver una pregunta que da un número:

| Requisito | Respuesta |
|---|---|
| Barato | **st1** o **sc1** |
| Throughput / streaming | **st1** |
| Bootear | **NO** st1 ni sc1 — solo SSD |
| Hasta **16.000** [[iops\|IOPS]] | **gp2 / gp3** |
| Hasta **64.000** IOPS | **io1 / io2** |
| Hasta **256.000** IOPS | **io2 Block Express** |
| Hasta **~260.000** IOPS | **RAID 0 con varios volúmenes EBS** (el tope por instancia) |
| **Más de 260.000** IOPS | **Instance Store** |

El corte importante es el último: **si el requisito supera lo que EBS puede entregarle a una sola instancia, la única respuesta posible es instance store** — y eso implica aceptar que los datos son efímeros.

## Trampa típica del examen

- **Reboot ≠ stop + start.** Es la distinción que más cae. El reboot conserva los datos (aunque el volumen queda desmontado); el stop + start los pierde.
- **El instance store no se puede agregar después.** Si el escenario dice "quiero sumarle almacenamiento local a una instancia que ya está corriendo", no se puede: hay que relanzarla. EBS sí se adjunta en caliente.
- *"Los datos deben sobrevivir reinicios"* → **EBS**. *"Caché efímero ultrarrápido"* → **Instance Store**.
- Instance store **no tiene snapshots**. Cualquier respuesta que proponga "hacer un snapshot del instance store" es incorrecta.
- El costo del instance store parece cero, pero solo viene con **ciertos tipos de instancia**: es un caso de *"usarlos o perderlos"*, porque ya están pagos.

## Ver también

[[EBS]] · [[ebs-volume-types]] · [[storage-types]] · [[EC2]] · [[ephemeral-storage]]
