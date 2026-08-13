---
title: CloudWatch Logs
category: service
tags: [cloudwatch-logs, logging, log-groups, metric-filters, monitoring]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.12 CloudWatch Logs.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.14 Precios.md", "raw/doc oficial/Working with log groups and log streams - Amazon CloudWatch Logs.md", "raw/doc oficial/Creating metrics from log events using filters - Amazon CloudWatch Logs.md"]
updated: 2026-07-25
---

# CloudWatch Logs

## ¿Qué es?

Servicio **público** para **almacenar, supervisar y analizar logs** — desde servicios AWS, on-premises u otras nubes. Un log = `[timestamp] + [mensaje]`.

> **Es regional**: los log groups viven en una region y no se ven desde otra. Centralizar logs de varias regiones o cuentas requiere replicarlos explícitamente con **subscription filters**.

## Características clave

### Arquitectura

```
Log Group   ← retención y permisos se fijan ACÁ
└── Log Stream  ← secuencia de events de UNA fuente
    └── Log Event  ← timestamp + mensaje
```

Ejemplo: 10 instancias EC2 enviando `/var/log/messages` → **1 log group**, **10 log streams**. La retención se configura una vez, a nivel group.

**Regla de diseño:** un log group **por tipo de log**, un log stream **por fuente**.

![[Pasted image 20260708223126.png]]

### Cómo llegan los datos

| Fuente | Cómo |
|---|---|
| Servicios AWS | Integraciones nativas (Lambda, VPC…) con IAM roles |
| SO de EC2 / on-premises / apps | **Unified CloudWatch Agent** |
| Dentro del código | AWS SDK |

> **"Instalé el agente y no llega nada"** — casi siempre falta una de dos cosas: un **IAM role** en la instancia con `logs:CreateLogStream` y `logs:PutLogEvents`, o el **archivo de configuración** del agente que le dice qué archivos leer y a qué log group mandarlos.

### Metric Filters

Patrones que escanean los log groups y **generan métricas** al coincidir (ej: contar `ERROR`). Métrica → **alarm** → notificación. Es lo que convierte a Logs de depósito pasivo en monitoreo activo.

Anatomía (doc oficial): **filter pattern** (qué buscar) + **metric name/namespace** (dónde publicar) + **metric value** (qué publicar: `1` para contar, o un número extraído del log, ej. bytes) + opcionales **default value** y **dimensions**.

- **No son retroactivos**: solo publican datapoints de eventos **posteriores** a la creación del filter.
- Si el filtro **no encuentra coincidencias no publica un cero**: no publica nada. Por eso la alarm asociada debe configurarse con **`treatMissingData: notBreaching`**, o queda permanentemente en `INSUFFICIENT_DATA`. (Del lado del filter, la otra mitad de la solución es el *default value = 0*.)
- **Default value = 0** evita métricas "agujereadas" en períodos con logs pero sin matches (si no llegan logs en el minuto, no se publica nada igual). ⚠️ Con dimensions asignadas **no se puede** usar default value.
- Las **[[dimension|dimensions]]** extraídas del log crean **una variación nueva de la métrica por cada par único** — cuidado con la [[high-cardinality|cardinalidad]]/costo (se facturan como custom metrics).
- La **unit** se fija al crear el filter; cambiarla después **no tiene efecto**.
- Percentile statistics disponibles solo si la métrica **nunca publica valores negativos**. Solo funcionan en log groups de clase **Standard**.

### Datos de la doc oficial

- Retención por defecto: **Never Expire** (se cobra para siempre) — configurable de 1 día a 10 años por log group.
- El borrado físico tras expirar puede demorar hasta **72 h**.
- Existen deletion protection y centralización [[cross-account]]/cross-region.

### Costos — es el motor de gasto #1 de la observabilidad

Se cobran **ingesta, almacenamiento y consultas**. La **ingesta por GB** es lo más caro y lo que más crece sin que nadie lo note: una app con logging en **modo debug en producción** puede costar más en logs que en cómputo.

Las dos palancas reales, en orden:
1. **Bajar la verbosidad** de la aplicación — ataca la ingesta, que es el costo dominante.
2. **Configurar retención** en todos los log groups (el default es infinito) y **archivar en [[S3]]** lo que se guarda por cumplimiento.

Borrar logs viejos ataca solo el almacenamiento, no la ingesta. Detalle completo en [[observability-costs]].

## Integración con otros servicios

- [[CloudWatch]] — metric filters → métricas → alarms.
- [[observability-costs]] — qué se cobra y cómo controlarlo.
- [[CloudTrail]] — puede enviar sus eventos aquí para alarmar sobre actividad de API.
- [[S3]] — destino de exportación/archivado; también S3 Access Logs pueden entregarse acá (opción moderna).

## Gotchas y trampas del examen

- Retención default = infinita → gotcha de costos; configurarla siempre.
- "La factura de Logs se disparó" → primero **bajar la verbosidad** del logging, no borrar logs viejos ([[observability-costs]]).
- "Retener logs 7 años al menor costo" → **archivar en S3/Glacier**, no dejarlos en CloudWatch Logs.
- "Alertar cuando aparece X en los logs" → **[[metric-filter|metric filter]] + alarm + SNS**.
- Logs de una app custom u on-premises → **CloudWatch Agent** (no hay magia nativa).
