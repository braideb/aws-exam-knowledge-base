---
title: Capacity provider
category: glossary
tags: [ecs, scaling, asg, compute]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.05 ECS — Cluster Types (EC2 y Fargate).md"]
updated: 2026-09-28
---

# Capacity provider

> **En una línea:** lo que en ECS **escala la infraestructura del cluster** (el ASG de las container instances) según lo que necesitan las tasks.

## Definición

En EC2 mode hay dos capas de scaling: el **service auto scaling** cambia la cantidad de **tasks**, y el capacity provider asociado al ASG (**ECS cluster auto scaling**) agrega o quita **instancias** para que esas tasks tengan dónde ubicarse.

## Dónde aparece

- [[ECS]] — scaling en dos capas
- [[ecs-ec2-vs-fargate]] — en Fargate no hace falta
- También en: [[dva-troubleshooting]]

## Dato de examen

- "El service escaló pero las tasks nuevas no arrancan por falta de CPU/memoria" → falta un **capacity provider** que escale el ASG. El service auto scaling **no** agrega instancias.

## Ver también

[[target-tracking-scaling]]
