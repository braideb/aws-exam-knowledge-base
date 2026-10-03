---
title: Amazon Aurora
category: service
tags: [aurora, rds, databases, sql, serverless, global-database, backtrack, clone, acu]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.13 Aurora — Architecture.md", "raw/notas curso mejorado/10 Databases (SQL)/10.14 Aurora — Restore, Clone y Backtrack.md", "raw/notas curso mejorado/10 Databases (SQL)/10.15 Aurora Serverless.md", "raw/notas curso mejorado/10 Databases (SQL)/10.17 Aurora Global Database.md", "raw/doc oficial/What is Amazon Aurora.md", "raw/doc oficial/What is Amazon Aurora 1.md", "raw/doc oficial/Amazon Aurora storage.md", "raw/doc oficial/Amazon Aurora MySQL database clusters now support up to 256 TiB of storage volume.md", "raw/doc oficial/Amazon Aurora endpoint connections.md", "raw/doc oficial/Custom endpoints for Amazon Aurora.md", "raw/doc oficial/Considerations for custom endpoints in Amazon Aurora.md", "raw/doc oficial/AWS CLI examples for custom endpoints for Amazon Aurora.md", "raw/doc oficial/Backtracking an Aurora DB cluster.md", "raw/doc oficial/Cloning a volume for an Amazon Aurora DB cluster.md", "raw/doc oficial/Amazon Aurora Fast Database Cloning.md", "raw/doc oficial/Using Aurora serverless.md", "raw/doc oficial/Performance and scaling for Aurora serverless.md", "raw/doc oficial/Scaling to Zero ACUs with automatic pause and resume for Aurora serverless.md", "raw/doc oficial/ServerlessV2ScalingConfiguration - Amazon Relational Database Service.md", "raw/doc oficial/Understanding how ACU minimum and maximum range impacts scaling in Amazon Aurora Serverless v2.md", "raw/doc oficial/Using Amazon Aurora Global Database.md", "raw/doc oficial/Promoting a read replica to a DB cluster for Aurora MySQL.md", "raw/doc oficial/Using the Amazon RDS Data API.md", "raw/doc oficial/Enabling the Amazon RDS Data API.md", "raw/doc oficial/Supported Regions and Aurora DB engines for RDS Data API.md", "raw/doc oficial/Amazon RDS Proxy for Aurora.md", "raw/doc oficial/Amazon RDS and Aurora High Availability Guide - Multi-AZ, Read Replicas, RDS Proxy, and Global Database Failover.md", "raw/doc oficial/Differences between Amazon Aurora and Amazon RDS.md"]
updated: 2026-10-03
---

# Amazon Aurora

## ¿Qué es?

Motor relacional **creado por AWS**, compatible con **MySQL y PostgreSQL**. Oficialmente es parte de [[RDS]] (misma consola, API y CLI), pero su arquitectura es distinta: un **cluster** con **una primary + hasta 15 replicas que sí se leen**, todas sobre un **cluster volume compartido** en vez de un EBS por instancia. Tiene variantes **provisioned**, **Serverless** (capacidad en [[acu|ACUs]]) y **Global Database** (multi-región).

## Casos de uso

- Cargas relacionales MySQL/PostgreSQL que necesitan **más rendimiento, failover rápido o muchas réplicas** que RDS.
- **Escalar lecturas y HA a la vez** sin elegir entre una cosa y la otra.
- **Cargas impredecibles, con picos o ociosas** → Aurora **Serverless v2**.
- **DR cross-region con RPO de ~1 s** y lecturas globales de baja latencia → **Global Database**.
- **Copias rápidas para test** de una base grande → **fast clone**.
- **Deshacer un error (un `DELETE` sin `WHERE`) sin restaurar** → **backtrack** (solo Aurora MySQL).

## Características clave

### Cluster y storage

```
            Cluster endpoint ──► Primary (escribe y lee)
            Reader endpoint  ──► Replica 1..15 (leen, son target de failover)
                     │
   ┌─────────────── Cluster volume compartido (SSD) ───────────────┐
   │  AZ-A: 2 copias   ·   AZ-B: 2 copias   ·   AZ-C: 2 copias      │  ← 6 copias, 3 AZs
   └────────────────────────────────────────────────────────────────┘
```

- La primary escribe y el storage replica **[[synchronous-replication|sincrónicamente]] a 6 nodos en 3 AZs**. La replicación es **a nivel storage**: no gasta recursos de las instancias. Se **auto-repara** (detecta segmentos fallados y los recrea desde las otras copias).
- Las replicas leen el **mismo volumen**: agregar o quitar replicas no copia datos. El [[replication-lag|lag]] de las replicas es típicamente **< 100 ms** ¹.
- SSD siempre; no hay storage magnético. **No se aprovisiona tamaño**: crece solo.

> ⚠️ Outdated: el curso da **128 TiB** de máximo y facturación por **[[high-watermark]]** (si liberás espacio, seguís pagando el máximo histórico). En las versiones actuales el máximo es **256 TiB** y el storage **se achica al borrar datos**, así que dejás de pagar lo liberado. En versiones viejas sin *dynamic resizing*, reducir sigue exigiendo dump + restore a un cluster nuevo.

**Configuraciones de storage** ¹: **Aurora Standard** (pagás I/O por cada millón de requests) vs **Aurora I/O-Optimized** (sin cargo por I/O; conviene cuando el I/O es **≥ 25 %** del gasto).

### Replicas y failover

- Hasta **15 Aurora Replicas**; **cualquiera** puede ser promovida.
- **Failover priority tier 0–15**: se promueve la de **menor número**; si empatan, la **más grande**.
- Failover a una replica existente: **~30 s** (la doc: < 60 s, a menudo < 30 s). **Sin replicas**, Aurora tiene que **recrear la primary** (hasta ~10 min ¹). Por eso tener **al menos una replica en otra AZ** es la decisión de HA más importante.

### Endpoints

| Endpoint | Apunta a | Uso |
|---|---|---|
| **Cluster (writer)** | La primary actual (sigue el failover) | Escrituras, DDL, lecturas |
| **Reader** | Balancea entre las replicas | Escalar lecturas |
| **Custom** | Un subconjunto que elegís (tipo `READER` o `ANY`, hasta **5 por cluster** ¹) | Ej.: instancias grandes solo para analytics |
| **Instance** | Una instancia concreta (**no** sigue el failover) | Diagnóstico y tuning |
| **Global writer** | La primary de la región primaria de una Global Database | Escrituras multi-región |

### Costos

- Cómputo por hora, facturado por segundo, **mínimo 10 minutos**. Storage por GB-mes consumido. **I/O por request** (en Standard).
- Backups: gratis hasta el **100 % del storage del cluster**.
- El curso dice que **no hay free tier** porque Aurora no soporta las microinstancias. Una comparación reciente de AWS (generada por IA, sin verificar) indica free tier para Aurora PostgreSQL: tomalo con cuidado.

### Backups, backtrack y fast clone

| | **Restore** (snapshot / [[point-in-time-recovery\|PITR]]) | **Backtrack** | **Fast clone** |
|---|---|---|---|
| Resultado | **Cluster nuevo**, endpoint nuevo | **Mismo** cluster, los datos vuelven atrás **in-place** | Cluster nuevo que **comparte** el storage |
| Velocidad | Lento (horas en bases grandes) | Minutos | Minutos, independiente del tamaño |
| Para | DR, recuperación a un punto | Deshacer un error sin tocar la app | Copias para test, análisis, cambios de schema |

- **Backtrack** ¹: solo **Aurora MySQL**. Se habilita **al crear** el cluster (o al restaurar un snapshot), no después. Ventana **máxima de 72 h**. Rebobina **todo** el cluster, no una tabla, y causa una interrupción breve. No es compatible con Global Database ni con réplicas cross-region.
- **Fast clone**: **[[copy-on-write]]**. Al principio clon y origen comparten las páginas; solo se copia lo que cambia en cualquiera de los dos. Hasta **15 clones copy-on-write** por origen (el 16.º es copia completa) y **solo en la misma región** ¹.

### Aurora Serverless

- Capacidad en **[[acu|ACUs]]** (~2 GiB de memoria + CPU y red ¹) entre un **mínimo y un máximo**. Facturación **por segundo** de ACU + storage. Misma resiliencia de storage (6 copias).
- Las ACU salen de un **warm pool** de AWS. La app se conecta a través de una **proxy fleet** gestionada, así que el escalado no corta conexiones.

| | **v1** (el curso) | **v2** (vigente) |
|---|---|---|
| Escalado | Duplica/divide la capacidad | Incrementos de **0,5 ACU**, casi instantáneo |
| Features | Limitadas | Read replicas, Multi-AZ, Global Database, RDS Proxy, IAM auth |
| Rango ¹ | — | 0,5 a **256 ACU** (128 en versiones viejas) |
| Escalar a cero | Pausa | **Auto-pause** con **mínimo 0 ACU** |

- **Auto-pause** ¹: tras **300 s a 86.400 s** sin conexiones (default 300 s). Mientras está pausada no cobra cómputo, solo storage. Reanudar tarda **~15 s** (≥ 30 s si estuvo pausada más de 24 h): conviene un timeout de cliente mayor y reintentos. **Con RDS Proxy asociado no se pausa** (el proxy mantiene conexiones abiertas).
- Casos de uso: apps de uso infrecuente, apps nuevas sin dimensionar, cargas variables o impredecibles, **dev/test**, multi-tenant.
- `max_connections` se calcula con el **máximo** de ACU: cambiar el máximo exige reiniciar ¹.

### Aurora Global Database

- **1 región primaria (lectura/escritura) + hasta 10 secundarias (solo lectura)**. Replicación [[asynchronous-replication|asíncrona]] **a nivel storage**, típicamente **< 1 s** (RPO ≈ 1 s) y sin impacto en la primaria.
- La región primaria tiene hasta 15 replicas; cada secundaria, **hasta 16**.
- **Global writer endpoint**: la app escribe ahí y el endpoint sigue a la primaria aunque cambie de región.
- Cambiar la región primaria:

| | **Switchover** (antes *managed planned failover*) | **Failover** |
|---|---|---|
| Cuándo | Planificado (rotación de región, simulacro) | Caída de la región primaria |
| Pérdida de datos | **Ninguna** (sincroniza antes) | Unos segundos (lo que tenía de lag) |

- **Write forwarding** ¹: una secundaria acepta escrituras y las **reenvía** a la primaria. No la convierte en multi-writer.
- Limitaciones ¹: sin backtrack, sin Aurora Auto Scaling en secundarias, y la integración de credenciales con Secrets Manager hay que desactivarla para sumar regiones.

### Otros

- **RDS Data API**: consultas SQL por **HTTPS/SDK**, sin conexión persistente y sin tener la Lambda en la VPC. Las credenciales salen de [[SecretsManager]]. La doc actual la habilita para clusters **provisioned y Serverless v2** (el curso la asocia solo a Serverless) ¹.
- **Aurora replica auto scaling**: agrega o quita replicas según la carga.
- **RDS Proxy con Aurora** ([[connection-pooling]]) ¹: el endpoint por defecto va al writer, pero se pueden crear **endpoints read-only** del proxy que van a las replicas.
- Promover a un cluster independiente una **read replica de Aurora MySQL cross-region** (binlog) es manual y reinicia las instancias ¹.

## Integración con otros servicios

- [[RDS]]: misma API, consola, IAM DB auth, RDS Proxy y snapshots. Se migra de RDS MySQL/PostgreSQL a Aurora con un **snapshot restore**.
- [[KMS]]: cifrado del cluster volume, snapshots y replicas.
- [[SecretsManager]]: credenciales del master (*managed rotation*) y Data API.
- [[lambda-in-vpc]]: Lambda → Aurora por la VPC + RDS Proxy, o por la Data API sin VPC.
- [[Route53]]: failover DNS del resto del stack en un DR multi-región.
- [[CloudWatch]]: `AuroraReplicaLag`, `AuroraGlobalDBReplicationLag`, `ACUUtilization`, `ServerlessDatabaseCapacity` ¹.

## Gotchas y trampas del examen

- **Aurora ≠ RDS Multi-AZ cluster**: comparten vocabulario (writer, reader, cluster), pero Aurora tiene **storage compartido** y hasta 15 replicas; el Multi-AZ cluster de RDS tiene **2 readers con storage local**. → [[rds-vs-aurora]]
- **Replicas de Aurora = HA + escalado de lectura a la vez**; la standby de RDS Multi-AZ instance no se lee.
- *"Escalar lecturas sin cambiar la app cada vez que agrego réplicas"* → **reader endpoint**.
- *"Mandar los reportes a instancias grandes separadas"* → **custom endpoint**.
- *"Volver atrás 10 minutos tras un `DELETE` sin `WHERE`, sin cambiar endpoints"* → **backtrack** (Aurora **MySQL**, habilitado de antemano).
- *"Copia de producción para test en minutos, sin pagar el doble de storage"* → **fast clone**.
- *"Carga SQL impredecible con largos períodos sin uso"* → **Aurora Serverless v2** (con auto-pause si tolera ~15 s de arranque).
- *"DR multi-región con RPO de segundos y RTO de ~1 minuto"* → **Global Database** (failover). *"Mover la región primaria sin perder datos"* → **switchover**.
- *"Escribir desde una región secundaria"* → **write forwarding**; sin él la secundaria es read-only.
- *"Lambda que consulta Aurora sin manejar conexiones ni meterse en la VPC"* → **Data API**.
- Un failover de Aurora sin ninguna replica es **lento**: recrea la primary.

¹ Complemento de la doc oficial (clippings en `sources`), no de las notas del curso.

> 📖 Lectura profunda: [[10.13 Aurora — Architecture]] · [[10.14 Aurora — Restore, Clone y Backtrack]] · [[10.15 Aurora Serverless]] · [[10.17 Aurora Global Database]]
