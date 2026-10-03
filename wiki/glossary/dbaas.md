---
title: DBaaS (Database as a Service)
category: glossary
tags: [rds, modelos-de-servicio, databases]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.03 RDS — Architecture.md"]
updated: 2026-10-03
---

# DBaaS (Database as a Service)

> **En una línea:** un servicio donde pagás y recibís **una base de datos** lista para usar, sin gestionar el servidor.

## Definición

Es el modelo [[paas|PaaS]] aplicado a bases de datos. El curso marca un matiz: [[RDS]] **no es exactamente** DBaaS sino **"database server as a service"**. Te da una **DB instance** (un servidor) en la que podés crear **varias bases**, y vos elegís tamaño, storage y Multi-AZ. Aurora Serverless se acerca más a la idea pura de pagar por la base y no por el servidor.

## Dónde aparece

- [[RDS]] — qué te entrega realmente
- [[databases-on-ec2]] — el extremo opuesto: todo lo gestionás vos

## Dato de examen

RDS = AWS gestiona hardware, SO, motor, parches y backups; vos no tenés acceso al SO. Si el enunciado exige acceso al SO → [[databases-on-ec2|EC2]].

## Ver también

[[paas]] · [[iaas]] · [[shared-responsibility-model]]
