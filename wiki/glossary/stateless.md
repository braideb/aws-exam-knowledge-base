---
title: Stateless (aplicación)
category: glossary
tags: [arquitectura, escalado, sesiones]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.23 Horizontal vs Vertical Scaling.md"]
updated: 2026-09-23
---

# Stateless (aplicación)

> **En una línea:** ningún servidor guarda estado propio del usuario, así que **da igual a cuál de todos te toque conectarte**.

## Definición

El estado que importa es la **sesión**. Si cada instancia guarda la suya localmente, mover al cliente de instancia lo desloguea. La solución son las **off-host sessions**: la sesión vive en un lugar central y compartido (DynamoDB, ElastiCache, una base) y todas las instancias la leen de ahí. Recién entonces los servidores son intercambiables.

> No confundir con [[stateless-firewall]], que describe una NACL y es otra cosa.

## Dónde aparece

- [[horizontal-vs-vertical-scaling]] — es el requisito que el escalado horizontal impone a la app
- [[dva-development]] — patrones de aplicación
- También en: [[ec2-cheat-sheet]]

## Dato de examen

- **Escalar horizontalmente exige una app stateless**; escalar verticalmente no toca la aplicación. Ese es el trade-off que se pregunta.
- Si el escenario dice *"los usuarios se deslogean cuando agregamos instancias"*, la respuesta es **mover las sesiones fuera del host**, no cambiar el balanceador ni el tipo de escalado.

## Ver también

[[horizontal-vs-vertical-scaling]] · [[idempotency]] · [[stateless-firewall]]
