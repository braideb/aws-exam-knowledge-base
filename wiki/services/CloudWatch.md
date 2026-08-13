---
title: CloudWatch
category: service
tags: [cloudwatch, monitoring, metrics, alarms, dimensions, namespaces]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.11 CloudWatch — Basics.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.14 Precios.md", "raw/doc oficial/Metrics concepts - Amazon CloudWatch.md", "raw/doc oficial/Using Amazon CloudWatch alarms.md"]
updated: 2026-07-24
---

# CloudWatch

## ¿Qué es?

El servicio de **monitoreo** de AWS: métricas, logs y eventos. Es **público**, así que recolecta datos desde AWS, on-premises y otras nubes. Sus tres tareas: **Metrics**, **[[CloudWatchLogs|Logs]]** y **Events** (hoy [[EventBridge]]).

> **CloudWatch vs. [[CloudTrail]] (no confundir):** CloudWatch = *performance* ("¿cómo está funcionando?" → CPU al 90%, errores en el log). CloudTrail = *auditoría* ("¿quién hizo qué?" → quién borró el bucket). Si la pregunta es "quién hizo X", la respuesta es **CloudTrail**.

## Características clave

### Jerarquía de conceptos

```
Namespace (AWS/EC2, CATAGRAM)
└── Metric (CPUUtilization)         ← serie temporal
    └── Datapoint (timestamp+value) ← UNA medición
        + Dimensions (InstanceId=i-xxx)  ← separan las fuentes
```

- **Namespace**: contenedor que agrupa métricas (`AWS/service` para lo nativo; propio para lo tuyo).
- **Metric**: el *tipo de dato* medido — no es un servidor.
- **Datapoint**: cada medición individual.
- **[[dimension|Dimensions]]**: pares nombre/valor que distinguen de dónde vino cada datapoint (CPU de una instancia puntual vs. promedio de todas las `t3.small`).

![[Pasted image 20260627153212.png]]

![[Pasted image 20260627154339.png]]

### Datos de la doc oficial (memorizar para el examen)

| Resolución del datapoint | Retención |
|---|---|
| < 60 s (high-resolution) | 3 horas |
| 1 min | 15 días |
| 5 min | 63 días |
| 1 hora | **455 días (15 meses)** |

- Máx **30 dimensions** por métrica; timestamps hasta 2 semanas atrás.
- Métricas nativas (CPU, red) vs. las que **requieren CloudWatch Agent**: **memoria RAM, disco por filesystem, procesos** — pregunta clásica.
- **Basic vs Detailed monitoring de EC2** (muy preguntado en DVA): basic = datapoints cada **5 min** (gratis); detailed = cada **1 min** (con costo). El período de una alarm debe ser ≥ la resolución de la métrica.
- **Statistics**: Average, Min, Max, Sum, SampleCount y **percentiles** (p95, p99). Se pueden publicar **statistic sets** (agregados pre-calculados) para no mandar cada datapoint.
- Períodos válidos: 1/5/10/30 s o múltiplos de 60 (default 60); sub-minuto solo en high-resolution.

### Qué se cobra

Las **métricas básicas son gratis**; se cobran **detailed monitoring, métricas custom y alarmas**. Cada combinación única de métrica + dimensions se factura **por separado** → una [[high-cardinality|dimensión de alta cardinalidad]] (un ID por request) multiplica el costo por miles. Panorama completo en [[observability-costs]].

### Alarms

Asociadas a **una métrica**; estados:

| Estado | Significado |
|---|---|
| **OK** | Dentro del umbral |
| **ALARM** | Umbral cruzado → acción: SNS, Auto Scaling, EC2 action (stop/terminate/reboot/recover), **Systems Manager** (OpsItem/automation) |
| **INSUFFICIENT_DATA** | No hay datos suficientes |

![[Pasted image 20260627154310.png]]

De la doc: existen **metric alarms, composite alarms** (combinan varias con AND/OR para reducir ruido) y **log alarms** (query de Logs Insights programada); una alarm dispara **al cambiar de estado**, no continuamente (excepción: acciones de Auto Scaling se reintentan por minuto).

- **Evaluation periods (M de N)**: la alarm evalúa una ventana de N datapoints y se configura **cómo tratar missing data** (notBreaching/breaching/ignore/missing) — un volumen EBS desasociado cae en `INSUFFICIENT_DATA`.
- Una **composite alarm solo puede notificar vía SNS** — no ejecuta acciones EC2/Auto Scaling.

## Integración con otros servicios

- [[EC2]] — métricas nativas + agent para las de SO.
- [[CloudWatchLogs]] — [[metric-filter|metric filters]] generan métricas desde logs.
- [[EventBridge]] — reaccionar a eventos y cron.
- SNS / Auto Scaling — destinos de las alarm actions.

## Gotchas y trampas del examen

- "No veo la memoria RAM de mi EC2" → instalar el **CloudWatch Agent** (con su IAM role).
- Un datapoint ≠ un servidor: es una medición; las **dimensions** identifican la fuente.
- Métrica custom con una dimensión por usuario/request → **explosión de cardinalidad y costo**; eso va a logs, no a métricas.
- Alertar solo si CPU alta **Y** latencia alta → **composite alarm**.
- Métricas de alta resolución (1s) existen pero se retienen solo 3 horas.

## Demos del curso

- [Demo de CloudWatch](https://learn.cantrill.io/courses/1101194/lectures/25301525)
