---
title: Dynamic port mapping
category: glossary
tags: [ecs, networking, alb, containers]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.05 ECS — Cluster Types (EC2 y Fargate).md"]
updated: 2026-09-28
---

# Dynamic port mapping

> **En una línea:** en ECS con network mode `bridge`, Docker le asigna a cada container un **puerto efímero del host** y ECS lo registra solo en el target group del ALB.

## Definición

Si el host port del port mapping queda en `0` (o vacío), cada task recibe un puerto distinto del host. Así entran **varias copias de la misma task en una instancia** sin chocar por el puerto. ECS registra cada `instancia:puerto` en el target group automáticamente.

## Dónde aparece

- [[ECS]] — network modes y ALB
- [[ecs-ec2-vs-fargate]] — `bridge` vs `awsvpc`
- También en: [[dva-troubleshooting]]

## Dato de examen

- Con dynamic port mapping, el **security group de las instancias** tiene que permitir el rango de [[ephemeral-port|ephemeral ports]] desde el SG del ALB. Si no, los health checks fallan.
- En **Fargate** no aplica: es siempre `awsvpc`, cada task tiene su propia IP y el target group es de tipo `ip`.

## Ver también

[[ephemeral-port]] · [[eni]]
