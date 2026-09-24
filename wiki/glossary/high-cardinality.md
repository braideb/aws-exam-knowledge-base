---
title: High Cardinality (alta cardinalidad)
category: glossary
tags: [cloudwatch, metrics, costos, observabilidad]
exam: [DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.14 Precios.md"]
updated: 2026-09-24
---

# High Cardinality (alta cardinalidad)

> **En una línea:** una etiqueta que toma **muchísimos valores distintos** — y en CloudWatch cada valor cuesta.

## Definición

La *cardinalidad* es la cantidad de valores únicos que puede tomar un campo. Es **alta** cuando ese número no está acotado: un request ID, un user ID, una IP, una URL con parámetros.

El problema en [[CloudWatch]]: cada combinación única de métrica + [[dimension|dimensions]] es **una métrica custom distinta**, y se factura por separado. Una dimensión con 10.000 valores no crea una métrica con 10.000 puntos: crea **10.000 métricas**.

## Dónde aparece

- [[observability-costs]] — tercer motor de costo
- [[CloudWatchLogs]] — las dimensions extraídas por un [[metric-filter]] crean una variación nueva por cada par único
- [[CloudWatch]] — máx **30 dimensions** por métrica
- También en: [[XRay]] · [[custom-metric]] · [[dimension]] · [[distributed-tracing]]

## Dato de examen

- Regla práctica: las **métricas** son para valores acotados (instance ID, tipo de instancia, entorno); los **logs** son para lo de alta cardinalidad (request IDs). Si el escenario necesita rastrear un request individual, la respuesta es **logs o [[XRay|X-Ray]]**, no una métrica con dimensión por request.
- En un metric filter, usar dimensions **impide** configurar el *default value* — otro motivo para no abusar de ellas.

## Ver también

[[dimension]] · [[observability-costs]]
