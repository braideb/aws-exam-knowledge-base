---
title: Costos de observabilidad y gobierno
category: concept
tags: [costos, optimizacion, cloudwatch, cloudtrail, logs, gobernanza, finops]
exam: [DVA-C02, DOP-C02, SAA-C03]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.14 Precios.md"]
updated: 2026-09-24
---

# Costos de observabilidad y gobierno

## Definición

Los precios cambian, pero **qué se cobra** no. La regla que resume todo el bloque:

> **Todo el andamiaje de identidad y gobierno es gratis. Lo que se paga es *observar*.**

[[IAM]], STS, [[Organizations]] y los SCPs no cuestan nada — son la base sobre la que se construye cualquier cuenta. El dinero se va en métricas, logs y auditoría detallada.

## Qué es gratis y qué no

| Servicio / funcionalidad | Costo |
|---|---|
| **[[IAM]]** (users, groups, roles, policies) | Gratis |
| **STS** ([[temporary-credentials\|credenciales temporales]]) | Gratis |
| **[[Organizations]]** | Gratis |
| **SCPs** | Gratis |
| **[[CloudTrail]] — Event History (90 días)** | Gratis |
| **CloudTrail — management events, primera copia** | **Gratis por trail** |
| **CloudTrail — copias adicionales de management events** | Se cobra por evento |
| **CloudTrail — data events** | Se cobra por evento |
| **CloudTrail — Insights** | Se cobra por evento analizado |
| **[[CloudWatch]] — métricas básicas** | Gratis |
| **CloudWatch — detailed monitoring, [[custom-metric\|métricas custom]], alarmas** | Se cobra |
| **[[CloudWatchLogs]] — ingesta, almacenamiento y consultas** | Se cobra |

## Los tres motores de costo

En orden de importancia práctica:

1. **Ingesta de CloudWatch Logs (por GB).** Es lo más caro y lo que más crece sin que nadie lo note. Una app con logging en **modo debug en producción** puede costar más en logs que en cómputo.
2. **[[data-plane|Data events]] de CloudTrail.** Activarlos para **todos** los buckets de una cuenta con tráfico alto genera volúmenes enormes.
3. **Métricas custom y alarmas.** Cada combinación única de métrica + [[dimension|dimensions]] se cobra **por separado**: una [[high-cardinality|dimensión de alta cardinalidad]] (un ID por request) multiplica el costo por miles.

## Cómo controlarlo

| Palanca | Por qué funciona |
|---|---|
| **Configurar retención** en todos los log groups | El default es **Never Expire** → se paga para siempre |
| **Bajar la verbosidad** de las apps en prod | Ataca la **ingesta**, que es el costo dominante — más efectivo que borrar logs viejos |
| **Archivar en [[S3]]** lo que se guarda por cumplimiento | Storage de S3 (o Glacier, ver [[s3-storage-classes]]) es mucho más barato que CloudWatch Logs |
| **Activar data events selectivamente** | Solo sobre los recursos que importan, con advanced event selectors |
| **Revisar métricas custom** buscando alta cardinalidad | Cada par único es una métrica facturada |
| **AWS Budgets** con alerta | No evita el gasto, pero avisa **antes** de la factura |

## Costos de cómputo y red que sorprenden

Fuera de la observabilidad, tres cargos de [[EC2]] que aparecen en preguntas de optimización:

- **Toda IPv4 pública se cobra** desde feb-2024, esté en uso o no: la [[elastic-ip|Elastic IP]] asociada a una instancia corriendo también. Antes solo se cobraban las ociosas. Salida por NAT Gateway o llegada por [[vpc-endpoints]] evitan asignarlas.
- **Detener una instancia no frena el costo de [[EBS]]**: el volumen se sigue facturando por GB-mes aprovisionado. Los **snapshots**, en cambio, se cobran por **datos usados**, no por el tamaño del volumen.
- **Borrar una AMI no borra sus snapshots** — siguen apareciendo en la factura.

El compromiso a largo plazo (Reserved, Savings Plans) y la capacidad sobrante (Spot) se comparan en [[ec2-purchase-options]].

## Preguntas de examen frecuentes

- "La factura de CloudWatch Logs se disparó, ¿qué hago primero?" → **bajar la verbosidad del logging** y **configurar retención**; borrar logs viejos no ataca la ingesta.
- "Retener logs 7 años al menor costo" → **exportar/archivar a S3 + Glacier**, no dejarlos en CloudWatch Logs.
- "¿Cuánto cuesta habilitar Organizations y SCPs para gobernar 30 cuentas?" → **nada**; el costo es cero (distractor típico).
- "Auditoría de API sin costo" → **Event History de CloudTrail** (90 días, management events). En cuanto el escenario pide **más de 90 días**, S3 o data events, hay costo.
- Un **segundo trail** que registre los mismos management events **ya se cobra** — la gratuidad es de la *primera copia* por trail.
- Métrica custom con una dimensión por usuario/request → **explosión de cardinalidad**; la respuesta correcta agrega la dimensión o usa logs, no métricas.

## Ver también

- [[CloudWatchLogs]] — retención, [[metric-filter|metric filters]] y su cardinalidad
- [[CloudTrail]] — tipos de evento y cuáles son opt-in
- [[dva-troubleshooting]] — dominio 4 del DVA-C02 (Troubleshooting **and Optimization**)
