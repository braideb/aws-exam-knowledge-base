---
title: Distributed Tracing
category: glossary
tags: [x-ray, tracing, observabilidad, microservices]
exam: [DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/07 Monitoring and logging/07.07 AWS X-Ray — Service Map.md"]
updated: 2026-09-24
---

# Distributed Tracing

> **En una línea:** seguir una misma request a través de todos los servicios que toca, con un ID compartido, para ver dónde se va el tiempo o dónde falla.

## Definición

En una aplicación distribuida una request pasa por varios servicios; cada uno por separado solo ve su parte. El tracing propaga un **trace ID** en un header, cada servicio reporta su tramo (un *segment*) y el sistema los une en una vista end-to-end. Es la tercera pata de la observabilidad: **métricas** (cuánto), **logs** (qué pasó), **trazas** (por dónde pasó y cuánto tardó cada tramo). En AWS lo implementa [[XRay]].

## Dónde aparece

- [[XRay]] — el servicio
- [[dva-troubleshooting]] — "interpretar traces" y "annotations para tracing" en los task statements

## Dato de examen

- "¿Qué microservicio agrega la latencia?" → tracing (X-Ray), no métricas ni [[CloudTrail]].

## Ver también

[[high-cardinality]] — rastrear una request individual con una métrica es el anti-patrón; para eso está el tracing.
