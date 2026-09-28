---
title: Ephemeral storage
category: glossary
tags: [storage, ec2, instance-store]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.04 Storage Refresh.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.09 Instance Store Volumes.md"]
updated: 2026-09-28
---

# Ephemeral storage

> **En una línea:** almacenamiento **temporal**, atado al hardware donde corre el recurso: si el recurso se mueve o el hardware falla, los datos desaparecen.

## Definición

Describe la **durabilidad**, no la forma de conectarse. En EC2 el caso es el **instance store**: discos físicos del propio host. Su opuesto es el almacenamiento **persistente**, que existe como recurso propio y sobrevive a la vida de la instancia — [[EBS]].

## Dónde aparece

- [[storage-types]] — efímero vs persistente
- [[instance-store-vs-ebs]] — el eje de la decisión
- [[EC2]] — ciclo de vida de la instancia
- También en: [[ec2-cheat-sheet]] · [[global-infrastructure]] · [[EKS]]

## Dato de examen

- Se pierde con **stop+start, hibernate, terminate o falla del host**. **No** se pierde con un **reboot**, porque el reboot no cambia de host físico (aunque hay que volver a montar el volumen).
- Es la contrapartida de ser el almacenamiento **más rápido de AWS**: se elige cuando los datos son reconstruibles (cachés, scratch, buffers).

## Ver también

[[instance-store-vs-ebs]] · [[storage-types]] · [[durability]]
