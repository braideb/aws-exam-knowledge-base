---
title: Target tracking scaling
category: glossary
tags: [scaling, auto-scaling, ecs, cloudwatch]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.04 ECS — Concepts.md"]
updated: 2026-09-28
---

# Target tracking scaling

> **En una línea:** una política de scaling que **agrega o quita capacidad para mantener una métrica en un valor objetivo**, como un termostato.

## Definición

Se elige una métrica y un valor (por ejemplo, 60% de CPU promedio) y el servicio calcula solo cuánto escalar para acercarse al objetivo. Es la política más común del **service auto scaling** de ECS (vía Application Auto Scaling), sobre CPU o memoria promedio del service o sobre requests por target del ALB.

## Dónde aparece

- [[ECS]] — service auto scaling
- [[horizontal-vs-vertical-scaling]] — es scaling horizontal automático

## Dato de examen

- "Mantener el uso promedio de CPU del service cerca del 50%" → **target tracking**, sin armar alarmas a mano.

## Ver también

[[capacity-provider]]
