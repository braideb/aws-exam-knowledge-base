---
title: PaaS (Platform as a Service)
category: glossary
tags: [modelos-de-servicio, paas, shared-responsibility]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.12 Modelo de responsabilidad compartida (Shared Responsibility Model).md"]
updated: 2026-10-03
---

# PaaS — Platform as a Service

> **En una línea:** AWS te da la plataforma lista; vos solo ponés **aplicación y datos**.

## Definición

El proveedor gestiona infraestructura, sistema operativo, runtime y contenedor; vos solo llevás tu código y tus datos. Menos control que [[iaas]], mucho menos trabajo operativo.

Ejemplos en AWS: Elastic Beanstalk, [[RDS]] (motor gestionado), [[ECS]] Fargate.

## Dónde aparece

- [[shared-responsibility-model]] — tabla de capas por modelo de servicio
- También en: [[iaas]] · [[saas]] · [[serverless]]

## Dato de examen

- Parchear el **motor** de RDS → AWS lo aplica, **vos elegís la ventana** de mantenimiento. Es el ejemplo canónico de responsabilidad compartida en PaaS.

## Ver también

[[iaas]] · [[saas]] · [[serverless]] · [[shared-responsibility-model]]
