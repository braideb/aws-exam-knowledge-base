---
title: CloudTrail
category: service
tags: [cloudtrail, auditoria, api-logging, trails, governance, seguridad]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.13 CloudTrail.md", "raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.14 Precios.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.14 Route 53 (R53) — Fundamentos.md", "raw/doc oficial/Understanding CloudTrail events.md", "raw/doc oficial/Validating CloudTrail log file integrity.md"]
updated: 2026-09-24
---

# CloudTrail

## ¿Qué es?

Registra las **acciones de API** de la cuenta: quién hizo qué, cuándo y desde dónde. Son "las cámaras de seguridad de AWS" — no le importa qué hace tu app por dentro ([[CloudWatchLogs]]), le importa quién apretó qué botón.

> **Todo en AWS es una llamada a la API, incluso clickear en la consola** (la consola es un cliente que llama a la API en tu nombre). Por eso CloudTrail ve absolutamente todo, sin importar si la acción vino de la consola, la CLI, un SDK o [[CloudFormation]].

### Qué hay dentro de un evento

Los campos que se miran al investigar:

| Campo | Qué dice |
|---|---|
| `eventTime` | Cuándo pasó |
| `userIdentity` | **Quién**: tipo (`IAMUser`, `AssumedRole`, `Root`, `AWSService`), ARN y nombre de sesión |
| `eventSource` / `eventName` | Servicio y operación (`ec2.amazonaws.com` / `TerminateInstances`) |
| `sourceIPAddress` | Desde dónde. Si dice `cloudformation.amazonaws.com`, lo hizo un **servicio**, no una persona |
| `requestParameters` | Con qué parámetros se llamó |
| `responseElements` | Qué devolvió AWS |
| `errorCode` | Si falló, por qué — los `AccessDenied` acá son **oro** para diagnosticar permisos |

Cuando la acción la hizo alguien asumiendo un rol, `userIdentity` muestra `assumed-role/NombreDelRol/NombreDeSesion`. **Por eso importa poner nombres de sesión identificables**: sin eso la auditoría dice "alguien con el rol de admin borró la base" y ahí se termina el rastro ([[IAM]]).

## Características clave

### Comportamiento por defecto (gratis)

- **Event History**: últimos **90 días**, habilitado por defecto, sin costo.
- Solo **Management Events**; **no** se guarda en S3.
- Es **por region**: hay que estar parado en la region correcta de la consola para ver sus eventos.
- Para más → crear **Trails**.

### Tipos de evento

| Tipo | Qué registra | ¿Default? |
|---|---|---|
| **Management** | Operaciones de **[[control-plane\|control plane]]** (crear EC2, VPC…) y eventos no-API como `ConsoleLogin` | ✅ |
| **Data** | Operaciones de **[[data-plane\|data plane]]** dentro de un recurso (GetObject en S3, `Invoke` de Lambda, item-level de DynamoDB, `Publish` de SNS, mensajes SQS) | ❌ opt-in, **costo extra**, volumen enorme |
| **Network activity** | Llamadas de API que atraviesan un **VPC endpoint** — las ve el dueño del endpoint (`eventType: AwsVpceEvent`; error `VpceAccessDenied` = bloqueado por la [[endpoint-policy\|endpoint policy]]) | ❌ opt-in, costo extra |
| **Insights** | Patrones **inusuales** de actividad: picos de call rate o de error rate vs. el baseline histórico de la cuenta. Emite un evento al **empezar** y otro al **terminar** la anomalía | ❌ opt-in, costo extra |

Detalles de data events (doc oficial): se seleccionan por **resource type** con event selectors — básicos solo para S3 objects/Lambda/DynamoDB; el resto (S3 Access Points, Object Lambda, S3 Express, tablas, etc.) requiere **advanced event selectors**. Los logs **no vienen ordenados** cronológicamente.

### Log file integrity validation

Para probar en una auditoría forense que **nadie tocó los logs**: al habilitarla, CloudTrail hashea cada log file (**SHA-256**) y entrega cada hora un **digest file** firmado (**RSA**) con los hashes de la última hora. Cada digest referencia y firma al **anterior** (cadena) → modificar, borrar o falsificar un log (o un digest) es detectable; también permite afirmar "no se entregaron logs en tal período".

- Los digest van **al mismo bucket S3** del trail pero en carpeta separada (permite policies distintas). Par de claves **por region**; la validación se corre con la **AWS CLI** (`validate-logs`).
- Habilitar la feature solo **entrega** los digest; validar es un paso aparte. Refuerzo recomendado: **MFA Delete** en el bucket ([[S3]]).

### Trails

- CloudTrail es **regional**: un trail registra su region.
- **Single-region** (hoy solo por CLI/API) vs. **all-regions** (un trail lógico que cubre todas, incluidas regiones futuras — recomendado).
- Destinos: **bucket S3** (JSON comprimido, retención indefinida, solo pagás storage) y/o **CloudWatch Logs** (habilita [[metric-filter|metric filters]] + alarms sobre actividad de API).
- **Costos**: la **primera copia de management events es gratis por trail**; las **copias adicionales se cobran por evento** (un segundo trail que registre lo mismo ya cuesta). Data events e Insights se cobran siempre. Ver [[observability-costs]].
- Se pueden usar **los dos a la vez**, y es lo habitual: S3 para retención larga y análisis, CloudWatch Logs para alertas en el momento.
- **Organizational trail**: desde la management account de [[Organizations]], audita **todas** las cuentas en un solo lugar. Las member accounts **ven sus propios eventos pero no pueden modificar ni apagar** el trail. Combinado con un SCP que deniegue `cloudtrail:StopLogging`, queda una auditoría que ninguna cuenta miembro puede desactivar.

> **Buena práctica de arquitectura:** el bucket de logs vive en **otra cuenta** (la *log archive account* de la organización), con permiso de **escritura pero no de borrado**. Así, quien comprometa la cuenta de producción **no puede destruir la evidencia**.

### CloudTrail Lake

Alternativa moderna a trail + S3 + Athena: guarda los eventos en un **event data store gestionado**, con retención de años, y permite consultarlos **con SQL** desde la consola sin montar nada. Más caro por evento, pero elimina la infraestructura de consulta.

> Aparece como respuesta cuando el escenario pide *"consultar el historial de actividad sin administrar infraestructura"*.

### Global Service Events

> [!danger] Punto de examen garantizado: **IAM, STS, CloudFront** (y Route 53) son servicios globales que registran sus eventos en **`us-east-1`**. Un trail en otra region sin la opción de global events habilitada **no los ve** (creándolo desde la consola viene habilitada por defecto).

![[Pasted image 20260708225948.png]]

## Integración con otros servicios

- [[S3]] — almacenamiento de logs de trails; data events de S3.
- [[CloudWatchLogs]] — metric filters y alarms sobre eventos.
- [[Organizations]] — organizational trail.
- [[KMS]] — cifrado de logs; auditoría de uso de claves.
- [[observability-costs]] — qué es gratis y qué no en auditoría y monitoreo.

## Gotchas y trampas del examen

- **NO es tiempo real** (~15 min de delay). "Detectar X en tiempo real" con CloudTrail = respuesta **incorrecta** → [[EventBridge]].
- "¿Quién creó este IAM user?" con trail solo en `sa-east-1` → no se ve; está en `us-east-1`.
- Data events apagados por defecto, con costo — activarlos explícitamente si el escenario los pide.
- "Demostrar que los logs no fueron alterados" → **log file integrity validation** (digest files SHA-256 + firma), no versioning ni ACLs.
- "Detectar un volumen anómalo de llamadas a una API" → **Insights events** (no una alarm a mano sobre cada API).
- CloudTrail = *quién hizo qué* (auditoría) / [[CloudWatch]] = *cómo está el sistema* (salud) / Config = *cómo estaba configurado*. CloudTrail registra **acciones**; Config registra **estados** a lo largo del tiempo. Ante un incidente se usan las tres: **Config dice qué cambió, CloudTrail dice quién lo cambió, CloudWatch dice qué efecto tuvo**.
- "Reaccionar automáticamente a una acción" (ej. detectar que abrieron el puerto 22 al mundo y revertirlo) → **[[EventBridge]]**, que recibe los eventos con latencia mucho menor. No leer el trail.
- "Consultar años de historial con SQL sin montar infraestructura" → **CloudTrail Lake** (no trail + Athena).
- "El atacante podría borrar los logs" → bucket en **otra cuenta**, sin permiso de delete, + `MFA Delete` + log file validation.

## Demos del curso

- [Implementing an Organizational Trail](https://learn.cantrill.io/courses/1101194/lectures/25527525)
