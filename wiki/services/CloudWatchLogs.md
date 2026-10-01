---
title: CloudWatch Logs
category: service
tags: [cloudwatch-logs, logging, log-groups, metric-filters, subscriptions, monitoring]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.12 CloudWatch Logs.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.14 Precios.md", "raw/notas curso mejorado/07 Monitoring and logging/07.05 CloudWatch Logs — Architecture.md", "raw/notas curso mejorado/07 Monitoring and logging/07.06 CloudWatch Logs — Subscriptions y Aggregation.md", "raw/doc oficial/Working with log groups and log streams - Amazon CloudWatch Logs.md", "raw/doc oficial/Creating metrics from log events using filters - Amazon CloudWatch Logs.md", "raw/notas curso mejorado/09 Advanced EC2/09.06 Logging en EC2 con CloudWatch Agent.md", "raw/notas curso mejorado/09 Advanced EC2/09.07 Demostración - Logging y métricas con CloudWatch Agent.md"]
updated: 2026-09-30
---

# CloudWatch Logs

## ¿Qué es?

Servicio **público** para **almacenar, supervisar y analizar logs** — desde servicios AWS, on-premises u otras nubes. Un log = `[timestamp] + [mensaje]`.

Tiene **dos caras**, y el examen las separa:
- **Ingestion** — meter los logs en el sistema (log groups, streams, retención, metric filters).
- **Subscription** — entregarlos a otros productos (Lambda, Kinesis, Firehose, OpenSearch).

> **Es regional**: los logs van a la region donde corre el servicio que los genera, y los log groups no se ven desde otra. **Excepción:** servicios globales como [[Route53]] (query logging) envían a **`us-east-1`**. Centralizar logs de varias regiones o cuentas requiere [[subscription-filter|subscription filters]].

## Características clave

### Arquitectura

```
Log Group   ← retención, permisos y cifrado (KMS) se fijan ACÁ
└── Log Stream  ← secuencia de events de UNA fuente
    └── Log Event  ← timestamp + raw message
```

Ejemplo: 10 instancias EC2 enviando `/var/log/messages` → **1 log group**, **10 log streams**. La retención se configura una vez, a nivel group.

**Regla de diseño:** un log group **por tipo de log**, un log stream **por fuente**.

![[Pasted image 20260708223126.png]]

![[Pasted image 20260923225616.png]]

### Cómo llegan los datos

| Fuente | Cómo |
|---|---|
| Servicios AWS | Integraciones nativas (Lambda, VPC…) con IAM roles |
| SO de EC2 / on-premises / apps | **Unified CloudWatch Agent** |
| Dentro del código | AWS SDK |

#### Logs del SO de EC2: el CloudWatch Agent

El interior de una instancia es **opaco** para CloudWatch y CloudWatch Logs: sin el agente no llegan ni los logs del SO ni los de las aplicaciones. El **unified CloudWatch Agent**, que reemplazó al viejo CloudWatch Logs agent, manda **logs y métricas a la vez**. Necesita tres cosas:

| # | Qué | Detalle |
|---|---|---|
| 1 | **Instalar el agente** | `dnf install amazon-cloudwatch-agent` |
| 2 | **Configurarlo** | Qué métricas y qué archivos de log. El wizard genera un `config.json`, que puede guardarse en **[[SSMParameterStore\|Parameter Store]]** (`AmazonCloudWatch-linux`) para reutilizarlo en todas las instancias |
| 3 | **Darle permisos** | Un [[instance-profile\|instance role]] con `CloudWatchAgentServerPolicy` (y acceso a SSM si la config vive ahí). Nunca access keys en la instancia |

Organización: **un log group por archivo de log** (`/var/log/secure`, `/var/log/httpd/access_log`…) y **un log stream por instancia** (instance ID). A escala, la instalación y la config se automatizan con [[CloudFormation]] o con user data ([[ec2-bootstrapping]]).

> **"Instalé el agente y no llega nada"** — casi siempre falta una de dos cosas: un **IAM role** en la instancia con `logs:CreateLogStream` y `logs:PutLogEvents`, o el **archivo de configuración** del agente que le dice qué archivos leer y a qué log group mandarlos.

### Metric Filters

Patrones que escanean los log groups y **generan métricas** al coincidir (ej: contar `ERROR`). **[[metric-filter|Metric filter]] → métrica → alarm** → notificación. Es lo que convierte a Logs de depósito pasivo en monitoreo activo.

Anatomía (doc oficial): **filter pattern** (qué buscar) + **metric name/namespace** (dónde publicar) + **metric value** (qué publicar: `1` para contar, o un número extraído del log, ej. bytes) + opcionales **default value** y **dimensions**.

- **No son retroactivos**: solo publican datapoints de eventos **posteriores** a la creación del filter.
- Si el filtro **no encuentra coincidencias no publica un cero**: no publica nada. Por eso la alarm asociada debe configurarse con **`treatMissingData: notBreaching`**, o queda permanentemente en `INSUFFICIENT_DATA`. (Del lado del filter, la otra mitad de la solución es el *default value = 0*.)
- **Default value = 0** evita métricas "agujereadas" en períodos con logs pero sin matches (si no llegan logs en el minuto, no se publica nada igual). ⚠️ Con dimensions asignadas **no se puede** usar default value.
- Las **[[dimension|dimensions]]** extraídas del log crean **una variación nueva de la métrica por cada par único** — cuidado con la [[high-cardinality|cardinalidad]]/costo (se facturan como [[custom-metric|custom metrics]]).
- La **unit** se fija al crear el filter; cambiarla después **no tiene efecto**.
- [[percentile|Percentile]] statistics disponibles solo si la métrica **nunca publica valores negativos**. Solo funcionan en log groups de clase **Standard**.

### Sacar los logs: export a S3 vs subscriptions

| | **Export a S3** (`CreateExportTask`) | **Subscription filter** |
|---|---|---|
| Latencia | **Hasta 12 h** — no es real time | Real time o [[near-real-time\|near real time]] según destino |
| Modo | Tarea puntual sobre un rango de tiempo | Continuo, por log group |
| Destino | Un bucket de [[S3]] | Lambda, Kinesis Data Streams, Firehose, OpenSearch |
| Uso típico | Archivar históricos | Procesar/centralizar en vivo |

> ⚠️ Outdated: el curso dice que el export solo admite buckets con **SSE-S3**. Hoy **también SSE-KMS** (la key policy debe permitir a CloudWatch Logs usar la key). Solo puede haber **una export task activa por cuenta**.

### Subscriptions

Un [[subscription-filter|subscription filter]] se crea sobre un log group y define: **pattern** (qué eventos), **ARN del destino**, **distribution** (cómo se reparten los datos) y el **IAM role** que CloudWatch Logs usa para escribir en el destino. Máx **2 subscription filters por log group**.

| Destino | Latencia | Para qué |
|---|---|---|
| **Lambda** (custom) | Real time | Entregar a cualquier lado con código propio |
| **OpenSearch** (ex Elasticsearch) | Real time | Nativo, vía una Lambda gestionada por AWS |
| **Kinesis Data Streams** | Real time | Base de la agregación multi-cuenta |
| **Firehose** (Amazon Data Firehose, ex Kinesis Data Firehose) | **Near real time** (buffer) | A S3 / terceros sin código |

![[Pasted image 20260923230418.png]]

**Log aggregation multi-cuenta:** cada cuenta apunta su subscription filter a un **Kinesis Data Stream central**; **Firehose** lee del stream y persiste en **S3**. Para el salto [[cross-account]], la cuenta central crea un **destination** de CloudWatch Logs (envuelve al stream) con una **destination policy** que autoriza a las cuentas origen.

![[Pasted image 20260923230724.png]]

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
- [[vpc-flow-logs]] — destino habitual de los flow logs cuando hacen falta **alarmas** (metric filters) o búsquedas con Logs Insights. Las alternativas son S3 (archivado barato + Athena) y Firehose (casi tiempo real).
- [[observability-costs]] — qué se cobra y cómo controlarlo.
- [[CloudTrail]] — puede enviar sus eventos aquí para alarmar sobre actividad de API.
- [[S3]] — destino de exportación/archivado; también S3 Access Logs pueden entregarse acá (opción moderna).
- [[KMS]] — cifrado del log group con una key propia.
- [[EC2]] — el CloudWatch Agent manda los logs del SO y de las apps; su configuración puede vivir en [[SSMParameterStore]].

## Gotchas y trampas del examen

- Retención default = infinita → gotcha de costos; configurarla siempre.
- "La factura de Logs se disparó" → primero **bajar la verbosidad** del logging, no borrar logs viejos ([[observability-costs]]).
- "Retener logs 7 años al menor costo" → **archivar en S3/Glacier**, no dejarlos en CloudWatch Logs.
- "Alertar cuando aparece X en los logs" → **[[metric-filter|metric filter]] + alarm + SNS**.
- Logs de una app custom u on-premises → **CloudWatch Agent** (no hay magia nativa).
- "La misma config del agente en 100 instancias" → guardarla en **Parameter Store** y cargarla con `amazon-cloudwatch-agent-ctl -a fetch-config -c ssm:<nombre>`.
- "Enviar logs a S3 **en (casi) tiempo real**" → **subscription + Firehose**, no `CreateExportTask` (hasta 12 h).
- "Procesar cada evento de log **en tiempo real**" → subscription con **Lambda** o Kinesis Data Streams; Firehose es *near* real time.
- "Centralizar logs de varias cuentas" → subscription filters → **Kinesis Data Stream central** (destination + destination policy) → Firehose → S3.
- No encontrás los logs de Route 53 en tu region → están en **us-east-1**.

## Demos del curso

- [Logging and metrics with CloudWatch Agent — Part 1](https://learn.cantrill.io/courses/1101194/lectures/27895417) · [Part 2](https://learn.cantrill.io/courses/1101194/lectures/29448612): instalar el agente, crear el rol `CloudWatchRole`, capturar `/var/log/secure` y los logs de Apache, y guardar la config en SSM

> 📖 Lectura profunda: [[03.12 CloudWatch Logs]] · [[07.05 CloudWatch Logs — Architecture]] · [[07.06 CloudWatch Logs — Subscriptions y Aggregation]] · [[09.06 Logging en EC2 con CloudWatch Agent]] · [[09.07 Demostración - Logging y métricas con CloudWatch Agent]]
