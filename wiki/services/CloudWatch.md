---
title: CloudWatch
category: service
tags: [cloudwatch, monitoring, metrics, alarms, dimensions, namespaces, resolution]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.11 CloudWatch — Basics.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.14 Precios.md", "raw/notas curso mejorado/07 Monitoring and logging/07.01 CloudWatch — Architecture Concepts.md", "raw/notas curso mejorado/07 Monitoring and logging/07.02 CloudWatch Data (Namespace, Datapoint, Metric, Dimensions).md", "raw/notas curso mejorado/07 Monitoring and logging/07.03 CloudWatch — Resolution, Retention y Statistics.md", "raw/notas curso mejorado/07 Monitoring and logging/07.04 CloudWatch Alarms.md", "raw/doc oficial/Metrics concepts - Amazon CloudWatch.md", "raw/doc oficial/Using Amazon CloudWatch alarms.md", "raw/notas curso mejorado/09 Advanced EC2/09.06 Logging en EC2 con CloudWatch Agent.md"]
updated: 2026-09-30
---

# CloudWatch

## ¿Qué es?

El servicio de **monitoreo** de AWS: métricas, logs y eventos. Es **público**, así que recolecta datos desde AWS, on-premises y otras nubes. Sus tres tareas: **Metrics**, **[[CloudWatchLogs|Logs]]** y **Events** (hoy [[EventBridge]]).

> **CloudWatch vs. [[CloudTrail]] (no confundir):** CloudWatch = *performance* ("¿cómo está funcionando?" → CPU al 90%, errores en el log). CloudTrail = *auditoría* ("¿quién hizo qué?" → quién borró el bucket). Si la pregunta es "quién hizo X", la respuesta es **CloudTrail**.

## Características clave

### Arquitectura: endpoint público, integración nativa y agent

- CloudWatch tiene un **endpoint en la zona pública de AWS**. Desde una [[VPC]] se llega por un **Internet Gateway** (o NAT) o, sin salir a internet, por un **interface endpoint** ([[vpc-endpoints]]).
- Muchos servicios publican métricas **de forma nativa** (EC2 sin configurar nada). Pero la integración nativa solo ve **lo que EC2 ve desde afuera** de la instancia.
- Lo que solo es visible **dentro del SO** (memoria, disco por filesystem, procesos) requiere el **CloudWatch Agent**. On-premises y aplicaciones publican vía agent o **API** (`PutMetricData`) → son [[custom-metric|custom metrics]].
- Encima de los datos: **dashboards**, **anomaly detection** y **alarms**.

![[Pasted image 20260923021337.png]]

### Jerarquía de conceptos

```
Namespace (AWS/EC2, CATAGRAM)
└── Metric (CPUUtilization)         ← serie temporal
    └── Datapoint (timestamp + value + unit opcional) ← UNA medición
        + Dimensions (InstanceId=i-xxx)  ← separan las fuentes
```

- **Namespace**: contenedor que aísla métricas del mismo nombre. `AWS/service` para lo nativo; el prefijo `AWS/` está **reservado**, los propios no pueden usarlo.
- **Metric**: el *tipo de dato* medido — no es un servidor. Se identifica por **Namespace + MetricName + Dimensions**: cambiar una [[dimension]] es, para CloudWatch, **otra métrica**.
- **Datapoint**: cada medición individual (timestamp, value y, opcional, **unit**: `Percent`, `Count`…).
- **Dimensions**: pares nombre/valor que distinguen de dónde vino cada datapoint (CPU de una instancia puntual) y permiten **agregar** (promedio de todas las `t3.small` o de un ASG). Esa agregación la hace AWS **solo en ciertas métricas nativas**, no en las custom.

![[Pasted image 20260627153212.png]]

![[Pasted image 20260627154339.png]]

### Resolution, retención y statistics (memorizar)

| | Standard | [[high-resolution-metric\|High resolution]] |
|---|---|---|
| Granularidad | **60 s** | **1 s** |
| Quién la usa | Métricas nativas de AWS | Solo custom metrics (`StorageResolution = 1`) |
| Períodos de lectura válidos | múltiplos de 60 s | 1 / 5 / 10 / 30 s o múltiplos de 60 s |
| Costo | Normal | Mayor |

| Resolución del datapoint | Retención |
|---|---|
| < 60 s (high-resolution) | 3 horas |
| 1 min | 15 días |
| 5 min | 63 días |
| 1 hora | **455 días (15 meses)** |

Los datos se van **agregando con el tiempo**: un dato de 1 min se ve con ese detalle 15 días, luego a 5 min hasta el día 63 y a 1 hora hasta el 455.

- Máx **30 dimensions** por métrica; timestamps hasta 2 semanas atrás.
- Métricas nativas (CPU, red) vs. las que **requieren CloudWatch Agent**: **memoria RAM, disco por filesystem, procesos** — pregunta clásica.
- **Basic vs Detailed monitoring de EC2** (muy preguntado en DVA): basic = datapoints cada **5 min** (gratis); detailed = cada **1 min** (con costo). Detailed **no** es high resolution: para bajar de 1 min hace falta una custom metric.
- **Statistics**: Average, Min, Max, Sum, SampleCount y **[[percentile|percentiles]]** (p95, p99). Se pueden publicar **statistic sets** (agregados pre-calculados) para no mandar cada datapoint.

### Qué se cobra

Las **métricas básicas son gratis**; se cobran **detailed monitoring, métricas custom y alarmas**. Cada combinación única de métrica + dimensions se factura **por separado** → una [[high-cardinality|dimensión de alta cardinalidad]] (un ID por request) multiplica el costo por miles. Panorama completo en [[observability-costs]].

### Alarms

Asociadas a **una métrica**; estados:

| Estado | Significado |
|---|---|
| **OK** | Dentro del umbral |
| **ALARM** | Umbral cruzado → acción: SNS, Auto Scaling, EventBridge, EC2 action (stop/terminate/reboot/recover), **Systems Manager** (OpsItem/automation) |
| **INSUFFICIENT_DATA** | No hay datos suficientes (recién creada, o la métrica dejó de publicar) |

![[Pasted image 20260627154310.png]]

**Parámetros** (ejemplo del curso: `HIGHCPU`, CPU > 50% en 2 períodos de 60 s):

| Parámetro | Qué define |
|---|---|
| **Period** | Duración de cada período evaluado |
| **Evaluation Periods (N)** | Cuántos períodos recientes se miran |
| **Datapoints to Alarm (M)** | Cuántos de los N deben romper el umbral ("M out of N": 3 de 5 ignora picos aislados) |
| **Condition** | Comparación + threshold |

De la doc: existen **metric alarms, composite alarms** (combinan varias con AND/OR para reducir ruido) y **log alarms** (query de Logs Insights programada); una alarm dispara **al cambiar de estado**, no continuamente (excepción: acciones de Auto Scaling se reintentan por minuto).

- **Missing data**: se configura cómo tratarla (notBreaching/breaching/ignore/missing) — un volumen EBS desasociado cae en `INSUFFICIENT_DATA`.
- **Resolución de la alarm**: sobre una métrica high resolution se puede crear una **high resolution alarm** de **10 o 30 s**; si no, el período es múltiplo de 60 s. Sobre una métrica standard, nunca menos de 60 s.
- Una **composite alarm solo puede notificar vía SNS** — no ejecuta acciones EC2/Auto Scaling.

## Integración con otros servicios

- [[EC2]] — métricas nativas + agent para las de SO. Los **status checks** (System e Instance) son métricas de CloudWatch, y una alarma sobre el System Status es lo que dispara el **auto-recovery**.
- [[EBS]] — métricas de [[iops|IOPS]], [[throughput]] y **balance de [[burst-credit|créditos de burst]]** de gp2: el lugar donde se ve venir que un volumen va a caer a su baseline.
- [[CloudWatchLogs]] — [[metric-filter|metric filters]] generan métricas desde logs.
- [[XRay]] — la tercera pata de la observabilidad (trazas); sus datos se ven desde la consola de CloudWatch.
- [[EventBridge]] — reaccionar a eventos y cron; también destino de alarm actions.
- [[vpc-endpoints]] — interface endpoint para publicar métricas sin salir a internet.
- SNS / Auto Scaling — destinos de las alarm actions.

## Gotchas y trampas del examen

- "No veo la memoria RAM de mi EC2" → instalar el **CloudWatch Agent** (con su IAM role). El interior de la instancia es opaco para CloudWatch; el agente unificado manda métricas del SO **y** logs ([[CloudWatchLogs]]).
- Un datapoint ≠ un servidor: es una medición; las **dimensions** identifican la fuente.
- Métrica custom con una dimensión por usuario/request → **explosión de cardinalidad y costo**; eso va a logs, no a métricas.
- Alertar solo si CPU alta **Y** latencia alta → **composite alarm**.
- "Reaccionar en menos de un minuto" → **custom metric high resolution + high resolution alarm** (10/30 s). Detailed monitoring no alcanza (1 min).
- Métricas de alta resolución (1 s) existen pero se retienen solo **3 horas**; "ver datos de 1 min de hace 20 días" → imposible, ya se agregaron a 5 min.
- Instancias en subnet privada **sin NAT** que no publican métricas → falta un **interface endpoint** de CloudWatch (o salida a internet).
- El promedio esconde la cola: para latencia, alarmar sobre **p95/p99**, no sobre Average.

## Demos del curso

- [Demo de CloudWatch](https://learn.cantrill.io/courses/1101194/lectures/25301525)

> 📖 Lectura profunda: [[01.11 CloudWatch — Basics]] · [[07.01 CloudWatch — Architecture Concepts]] · [[07.02 CloudWatch Data (Namespace, Datapoint, Metric, Dimensions)]] · [[07.03 CloudWatch — Resolution, Retention y Statistics]] · [[07.04 CloudWatch Alarms]]
