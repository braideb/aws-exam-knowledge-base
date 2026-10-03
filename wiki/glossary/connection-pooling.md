---
title: Connection pooling
category: glossary
tags: [rds, rds-proxy, lambda, performance]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.06 RDS Multi-AZ Instance.md", "raw/doc oficial/Amazon RDS Proxy - Amazon Relational Database Service.md", "raw/doc oficial/Using Amazon RDS Proxy with AWS Lambda.md"]
updated: 2026-10-03
---

# Connection pooling

> **En una línea:** mantener un grupo de conexiones a la base **abiertas y reutilizables**, en vez de abrir una nueva por cada request.

## Definición

Abrir una conexión a una base relacional cuesta CPU y memoria, y la base tiene un tope (`max_connections`). Un pool reparte unas pocas conexiones "calientes" entre muchos clientes. En AWS lo da gestionado **RDS Proxy**, clave con **Lambda**, que puede escalar a miles de ejecuciones concurrentes, cada una con su propia conexión. Si un cliente cambia el estado de la sesión, el proxy la **"pinea"** a una conexión y el pool pierde eficiencia ¹.

¹ Complemento de la doc oficial.

## Dónde aparece

- [[RDS]] — RDS Proxy
- [[Aurora]] — RDS Proxy con endpoints read-only
- [[lambda-in-vpc]]
- También en: [[dva-development]]

## Dato de examen

*"Lambda agota las conexiones de RDS"* / *"too many connections"* → **RDS Proxy**. Además acorta el failover, porque no depende del DNS.

## Ver también

[[throttling]] · [[serverless]] · [[failover]]
