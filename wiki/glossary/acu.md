---
title: ACU (Aurora Capacity Unit)
category: glossary
tags: [aurora, serverless, capacidad]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.15 Aurora Serverless.md", "raw/doc oficial/Understanding how ACU minimum and maximum range impacts scaling in Amazon Aurora Serverless v2.md", "raw/doc oficial/ServerlessV2ScalingConfiguration - Amazon Relational Database Service.md", "raw/doc oficial/Performance and scaling for Aurora serverless.md"]
updated: 2026-10-03
---

# ACU (Aurora Capacity Unit)

> **En una línea:** la unidad de capacidad de Aurora Serverless: ~**2 GiB de memoria** más su CPU y red correspondientes.

## Definición

En Aurora Serverless no elegís un tipo de instancia: fijás un **mínimo y un máximo de ACUs** y Aurora escala entre esos valores según la carga. Pagás las ACUs usadas **por segundo**. En v2 escala en pasos de **0,5 ACU**, de 0,5 a **256** (128 en versiones viejas), y con **mínimo 0** se habilita el **auto-pause** ¹. El máximo de ACUs también fija `max_connections` ¹.

¹ Complemento de la doc oficial.

## Dónde aparece

- [[Aurora]] — Aurora Serverless v1/v2
- [[databases-cheat-sheet]]

## Dato de examen

*"Base que no se usa de noche y tiene picos impredecibles de día"* → **Aurora Serverless v2** con un rango de ACUs amplio (y auto-pause si tolera ~15 s de arranque).

## Ver también

[[serverless]] · [[target-tracking-scaling]]
