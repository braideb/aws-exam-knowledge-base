---
title: Golden AMI (AMI baking)
category: glossary
tags: [ec2, ami, deployment, inmutabilidad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.17 Amazon Machine Image (AMI).md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.06 Amazon Machine Image (AMI).md"]
updated: 2026-09-22
---

# Golden AMI (AMI baking)

> **En una línea:** una AMI **preconfigurada con el SO, el software y los ajustes ya listos**, para que las instancias arranquen sirviendo en vez de instalarse solas.

## Definición

El proceso de crearla se llama **AMI baking** y son cuatro pasos: *launch* una AMI base → *configure* la instancia hasta dejarla como querés → *create image* → *re-launch* todas las instancias que necesites. El resultado es un artefacto inmutable: una AMI **no se edita**, se hornea una nueva.

## Dónde aparece

- [[EC2]] — AMI lifecycle
- [[dva-deployment]] — patrón de despliegue inmutable
- [[ec2-cheat-sheet]] — puntos de examen de AMI

## Dato de examen

- El contraste es contra configurar en boot con **user data**: baking tiene **arranque rápido y determinista** (clave para autoscaling), pero exige rehornear ante cualquier cambio; user data arranca más lento pero es más flexible.
- La AMI es **regional** y **privada por defecto**; se comparte con cuentas específicas (permiso *explicit*) o se copia a otra región.

## Ver también

[[EC2]] · [[dva-deployment]]
