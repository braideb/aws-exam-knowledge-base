---
title: S3 Storage Classes
category: comparison
tags: [s3, storage-classes, glacier, intelligent-tiering, lifecycle, costos]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/04 S3/04.08 S3 Object Storage Classes.md", "raw/notas curso mejorado/04 S3/04.09 S3 Lifecycle Configuration.md", "raw/doc oficial/Understanding and managing Amazon S3 storage classes - Amazon Simple Storage Service.md", "raw/doc oficial/Transitioning objects using Amazon S3 Lifecycle - Amazon Simple Storage Service.md"]
updated: 2026-07-25
---

# Comparativa: S3 Storage Classes

> Complementa la página de [[S3]] con la dimensión económica del servicio.

## Los cuatro parámetros que las diferencian

Cada storage class es una combinación distinta de estos cuatro:

1. **Precio de almacenamiento** (GB/mes) — cuánto cuesta guardar.
2. **Precio y modo de recuperación** — cuánto cuesta y cuánto tarda leer.
3. **[[durability|Durabilidad]] y AZs** — cuántas copias en cuántas AZs.
4. **Mínimos facturables** — duración mínima y tamaño mínimo por objeto.

> **La regla del negocio de S3:** cuanto más barato es guardar, **más caro y lento es recuperar, y más largos son los mínimos**. Ninguna clase gana en todo — se elige según el patrón de acceso.

Tres cosas que **solo** Standard tiene: sin tarifa de recuperación, sin duración mínima, sin tamaño mínimo. Además: transferencia **hacia** S3 gratis, **desde** S3 por GB, y requests por cada 1.000.

Detalles de durabilidad que aparecen en el curso: 11 nueves significa que con **10.000.000 de objetos** la expectativa es perder **1 objeto cada 10.000 años**; la replicación usa **Content-MD5 y CRC** para detectar y reparar corrupción; y el **`HTTP/1.1 200 OK`** de un PUT **es la confirmación de que el objeto quedó almacenado de forma duradera**.

## Tabla comparativa

Todas las clases tienen **11 nueves de [[durability|durabilidad]]**; cambian [[availability|disponibilidad]], costos y acceso.

| Clase | AZs | Min. duración | Min. tamaño | Retrieval fee | Primer byte | Uso |
|---|---|---|---|---|---|---|
| **Standard** | ≥3 | — | — | No | ms | Datos calientes, importantes, **default** |
| **Standard-IA** | ≥3 | 30 d | 128 KB | Sí | ms | Acceso infrecuente, largo plazo |
| **One Zone-IA** | **1** | 30 d | 128 KB | Sí | ms | Infrecuente + **reemplazable** (réplicas, intermedios) |
| **Glacier Instant** | ≥3 | 90 d | 128 KB | Sí (mayor) | ms | Archivo con acceso instantáneo raro (~trimestral) |
| **Glacier Flexible** | ≥3 | 90 d | —* | Sí + retrieval job | min–horas | Archivo frío (~1/6 del precio de Standard) |
| **Glacier Deep Archive** | ≥3 | 180 d | —* | Sí + retrieval job | horas–días | Archivo "congelado": retención legal, backups |
| **Intelligent-Tiering** | ≥3 | — | — | **No** (fee de monitoreo por cada 1.000 objetos) | ms–horas | Patrón de acceso **desconocido/cambiante** |

> \* Glacier Flexible/Deep Archive no tienen mínimo facturable de objeto, pero **cada objeto suma ~40 KB de overhead de metadata** (32 KB a tarifa Glacier + 8 KB a tarifa Standard) — por eso millones de objetos chicos siguen siendo mala idea. El curso lo simplifica como "mínimo 40 KB"; la doc oficial lo detalla como overhead.

Retrieval jobs de Glacier Flexible: **Expedited** (1–5 min) / **Standard** (3–5 h) / **Bulk** (5–12 h). Deep Archive: Standard 12 h / Bulk hasta 48 h. El objeto restaurado queda disponible en una **copia temporal** (facturada a tarifa Standard); la storage class del objeto **sigue reportándose como Glacier** — para el cambio permanente hay que cambiar la clase.

![[Pasted image 20260716001707.png]]

## Cuándo usar cada una

- **Standard es el default**: solo migrás con una razón específica.
- Frecuencia conocida y baja → IA (o One Zone-IA si el dato es **regenerable**).
- Archivo puro → Glacier según cuánto tolerás esperar.
- **No sabés el patrón** → Intelligent-Tiering (sin retrieval fee; mueve objetos solo según uso real).

**Disponibilidad diseñada por clase** (la durabilidad es 11 nueves en todas; esto es *availability*): Standard **99.99%** · Standard-IA/Glacier IR/Intelligent-Tiering **99.9%** · One Zone-IA **99.5%** · Express One Zone 99.95%.

**Tiers internos de Intelligent-Tiering** — cinco, y cada uno equivale conceptualmente a una clase suelta:

| Tier interno | Equivale a | Notas |
|---|---|---|
| Frequent Access | Standard | default |
| Infrequent Access | Standard-IA | 30 días sin acceso |
| Archive Instant Access | Glacier Instant | 90 días, automático, sigue en ms |
| Archive Access | Glacier Flexible | **opt-in**, necesita retrieval |
| Deep Archive Access | Glacier Deep Archive | **opt-in**, latencia de horas |

Objetos **<128 KB no se monitorean** (quedan en Frequent, sin fee). El cargo de monitoreo es **por objeto**, así que para millones de objetos muy chicos puede no compensar.

> **Intelligent-Tiering vs. Lifecycle — la confusión típica:** Intelligent-Tiering mueve objetos según el **acceso real**, automáticamente y **en ambas direcciones**. Una lifecycle rule mueve según la **edad**, en **una sola dirección** (hacia clases más frías), y la configurás vos.
>
> Si sabés que un log a los 30 días ya no se toca → **lifecycle**. Si no tenés idea de cuándo se accede a cada objeto → **Intelligent-Tiering**.

Otras clases que pueden aparecer como distractores: **Express One Zone** (directory buckets, latencia de ms de un dígito — ver [[S3]]) y **Reduced Redundancy Storage** (legacy, NO usar — menor durabilidad).

## Lifecycle Configuration

Reglas por bucket que **transicionan** (cascada hacia clases más frías, nunca hacia arriba) o **expiran** objetos. La cascada:

```
Standard → Standard-IA → Intelligent-Tiering → One Zone-IA
        → Glacier Instant → Glacier Flexible → Glacier Deep Archive
```

**Criterios para elegir a qué objetos aplica:** [[prefix|prefijo]] (`logs/`, `temp/`), **tags** (reglas finas sin depender de los nombres), **tamaño** (útil justamente para *no* transicionar los chiquitos), o todo el bucket.

**Las cuatro acciones de expiración** — las dos últimas son limpieza de basura que casi siempre conviene activar:

| Acción | Qué borra |
|---|---|
| **Expire current versions** | Le pone un [[delete-marker]] a la versión actual tras X días |
| **Permanently delete noncurrent versions** | Borra **de verdad** las versiones viejas tras X días |
| **Delete expired delete markers** | Limpia delete markers que quedaron sin ninguna versión debajo |
| **Abort incomplete multipart uploads** | Borra las partes de subidas nunca completadas tras X días |

> **El patrón estándar** para un bucket con versioning que no debe crecer sin control: expirar noncurrent versions a los **30-90 días**, borrar delete markers huérfanos, y **abortar multipart incompletos a los 7 días**.

![[Pasted image 20260716003516.png]]

Restricciones de examen:
- **30 días mínimo** en Standard antes de transicionar a IA/One Zone-IA.
- Una sola regla no puede encadenar IA → Glacier dentro de los 30 días siguientes (usar dos reglas).
- Objetos chicos: los **mínimos facturables** (128 KB/40 KB) pueden hacer que transicionar **cueste más**.
- Doc oficial (2024): por defecto los objetos **<128 KB ya no transicionan** salvo configuración explícita.
- **Transicionar cuesta plata**: cada objeto que cambia de clase genera una **request facturada**. Mover millones de objetos chiquitos puede costar más (requests + penalización del mínimo de 128 KB) que el ahorro de almacenamiento buscado → para objetos muy chicos, dejarlos en Standard o usar Intelligent-Tiering.
- ⏱️ **Las transiciones no son instantáneas**: se procesan **en lotes, ~1 vez al día**, así que un objeto que "cumple 30 días" puede tardar **24-48 h** en moverse. No es un error, es cómo funciona el batch.

## Trampa típica del examen

- "Datos críticos e irreemplazables" + One Zone-IA → ❌ (una AZ destruida = datos perdidos).
- "Acceso instantáneo" + Glacier Flexible/Deep Archive → ❌ (necesitan retrieval job); **Glacier Instant** sí es instantáneo. De las **tres clases con "Glacier" en el nombre, solo Instant** da acceso inmediato: es fría en precio, caliente en velocidad.
- En Glacier Flexible/Deep Archive los objetos **no se pueden hacer públicos**, y lo que "se ve" en el bucket son **punteros**, no los objetos.
- "Standard-IA para datos que en realidad se acceden seguido" → ❌: las tarifas de recuperación por GB **superan** lo ahorrado en almacenamiento. Y los mínimos (30 días, 128 KB) la hacen pésima para objetos chicos o de vida corta.
- "Patrón impredecible, sin sorpresas de costo" → **Intelligent-Tiering**.
- Lifecycle de Standard a IA a los 10 días → ❌ inválido (mínimo 30).
