---
title: AWS X-Ray
category: service
tags: [x-ray, tracing, observabilidad, microservices, troubleshooting]
exam: [DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/07 Monitoring and logging/07.07 AWS X-Ray — Service Map.md"]
updated: 2026-09-24
---

# AWS X-Ray

> ⚠️ **Sin contraste con doc oficial:** esta página sale solo de las notas del curso ([[07.07 AWS X-Ray — Service Map]]); no hay clippings de X-Ray en `raw/doc oficial/`. Los datos que no vienen de la slide del curso (daemon en UDP 2000, sampling por defecto, annotations vs metadata) están marcados como complemento y conviene verificarlos.

## ¿Qué es?

El servicio de **[[distributed-tracing|distributed tracing]]** de AWS: sigue cada request a medida que atraviesa los servicios de una aplicación distribuida (microservicios, [[serverless]]) y arma una **vista end-to-end** con tiempos y errores, para encontrar la **causa raíz** de problemas de performance. Es la tercera pata de la observabilidad junto a las métricas ([[CloudWatch]]) y los logs ([[CloudWatchLogs]]).

## Casos de uso

- "¿En qué servicio se pierde el tiempo de esta request?" — latencia en cadenas API Gateway → Lambda → DynamoDB.
- Encontrar qué dependencia devuelve errores (4xx/5xx, [[throttling]]) dentro de una arquitectura de microservicios.
- Ver el **mapa** real de dependencias de una aplicación.
- Rastrear **una request individual** — lo que no se puede hacer con una métrica sin caer en [[high-cardinality|alta cardinalidad]].

## Características clave

### Cómo funciona

1. Al entrar la request se genera un **trace ID**, que viaja en el **tracing header** (`X-Amzn-Trace-Id`).
2. Cada servicio que participa envía **segments** a X-Ray: host/IP, request, response, trabajo hecho (tiempos) e issues.
3. Un segment puede tener **subsegments** para más detalle (una llamada a DynamoDB o a una API externa dentro de una Lambda).
4. X-Ray junta todos los segments del mismo trace ID → el **trace** end-to-end.

| Término | Qué es |
|---|---|
| **Trace** | Todos los segments de una request (mismo trace ID) |
| **Segment** | La parte del trabajo que aporta un servicio |
| **Subsegment** | Detalle más fino dentro de un segment |
| **Service graph** | Los datos de todos los servicios combinados |
| **Service map** | La vista visual del service graph: nodos, llamadas, latencias y errores |

![[Pasted image 20260923231558.png]]

![[Pasted image 20260923231759.png]]

### Cómo se habilita en cada servicio

| Servicio | Cómo |
|---|---|
| **EC2** | Instalar el **X-Ray daemon/agent** + SDK en la app |
| **ECS** | Daemon como contenedor en la task (sidecar) |
| **Lambda** | Toggle **Active tracing** |
| **Elastic Beanstalk** | Daemon **preinstalado**, se activa en la config |
| **API Gateway** | Opción **por stage** |
| **SNS / SQS** | SNS soporta active tracing; SQS propaga el trace header |

En **todos** los casos hacen falta **permisos IAM** para enviar datos: `xray:PutTraceSegments` y `xray:PutTelemetryRecords` (managed policy `AWSXRayDaemonWriteAccess`) en el [[instance-profile|instance role]] o [[execution-role|execution role]].

### Complementos para DVA (no vienen del curso)

- El daemon escucha en **UDP 2000** y reenvía en lotes a la API de X-Ray.
- **Annotations** = key/value **indexados** → sirven para **filtrar y buscar traces** (filter expressions). **Metadata** = datos **no indexados** → solo para guardar contexto.
- **Sampling**: no se trazan todas las requests. Regla por defecto: la **primera request de cada segundo + 5%** del resto; configurable con sampling rules sin redeployar.

> ⚠️ Outdated: AWS está llevando la instrumentación hacia **OpenTelemetry** (AWS Distro for OpenTelemetry, ADOT) como reemplazo de los SDKs y del daemon propios de X-Ray. Los conceptos (trace, segment, service map, annotations) no cambian y son los que evalúa el examen.

## Integración con otros servicios

- [[CloudWatch]] — las trazas se consultan desde la consola de CloudWatch, junto a métricas y logs.
- Lambda — el caso más preguntado: con active tracing solo falta el permiso en el execution role. (Sin página de servicio todavía; el ángulo de red está en [[lambda-in-vpc]].)
- [[EC2]] — requiere daemon + IAM role (igual que el CloudWatch Agent).
- [[dva-troubleshooting]] — la Task 2 del dominio pide explícitamente **annotations para tracing**.

## Gotchas y trampas del examen

- "Ver qué servicio de una cadena de microservicios agrega latencia" → **X-Ray**, no CloudWatch metrics ni CloudTrail.
- "Instrumenté la app en EC2 pero no aparecen traces" → falta el **daemon** o los **permisos IAM** del instance role.
- "Lambda no envía traces" → activar **Active tracing** y verificar el **execution role**.
- "Buscar/filtrar traces por `userId` o por cliente" → **annotation** (indexada), no metadata.
- "Rastrear una request individual" → X-Ray o logs; **no** una métrica con una [[dimension]] por request.
- X-Ray **no** es auditoría ("quién llamó a la API") → eso es [[CloudTrail]].

## Demos del curso

- [DEMO — Lambda & AWS X-Ray](https://learn.cantrill.io/courses/1101194/lectures/46260482)

> 📖 Lectura profunda: [[07.07 AWS X-Ray — Service Map]]
