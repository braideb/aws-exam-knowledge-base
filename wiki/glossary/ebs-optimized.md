---
title: EBS-optimized
category: glossary
tags: [ec2, ebs, performance, red]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.03 EC2 Instance Types.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.07 EBS Volume Types - Provisioned IOPS SSD (io1, io2, Block Express).md"]
updated: 2026-09-24
---

# EBS-optimized

> **En una línea:** una instancia con **ancho de banda de red dedicado** para hablar con EBS, separado del tráfico normal.

## Definición

[[EBS]] es almacenamiento **por red**, así que todo el I/O de disco compite con el tráfico de la aplicación. Una instancia EBS-optimized reserva capacidad para ese canal. Aun así existe un **techo por instancia** (depende del tipo y tamaño de instancia) que puede ser menor que lo que aguantan los volúmenes: ahí la instancia, y no el disco, es el cuello de botella.

## Dónde aparece

- [[ec2-instance-types]] — qué definís al elegir un tipo
- [[ebs-volume-types]] — el tope por instancia de los volúmenes io1/io2
- [[instance-store-vs-ebs]] — el límite de ~260.000 [[iops|IOPS]] por instancia
- También en: [[EBS]] · [[EC2]] · [[dva-troubleshooting]] · [[ec2-cheat-sheet]] · [[virtualization]] · [[throughput]]

## Dato de examen

- Si el escenario dice "aprovisioné 64.000 IOPS pero mido mucho menos", la causa probable es el **ancho de banda de la instancia**, no el volumen.
- Para llegar al tope por instancia hace falta **combinar varios volúmenes** (RAID 0): uno solo no alcanza.

## Ver también

[[iops]] · [[throughput]] · [[ec2-instance-types]]
