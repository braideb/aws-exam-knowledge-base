---
title: HA vs FT vs DR
category: concept
tags: [resiliencia, high-availability, fault-tolerance, disaster-recovery]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.13 HA vs. FT vs. DR.md"]
updated: 2026-07-23
---

# High Availability vs. Fault Tolerance vs. Disaster Recovery

## Definición

Tres conceptos que se confunden todo el tiempo:

- **High Availability (HA)** → **minimizar** las interrupciones (maximizar uptime).
- **Fault Tolerance (FT)** → **operar a través** de los fallos, sin que el usuario note nada.
- **Disaster Recovery (DR)** → políticas y procedimientos para recuperarse **cuando HA y FT no alcanzaron**.

![[Pasted image 20260627164248.png]]

## Detalle de cada uno

### HA — lo que NO es

HA **no garantiza ausencia de fallos**. Si un componente falla y su reemplazo tarda unos segundos (y los usuarios lo notan), **sigue siendo HA**: el objetivo es solo maximizar el uptime.

| [[availability|Disponibilidad]] | Downtime/año | Por mes |
|---|---|---|
| 99% (two 9's) | ~3,65 días | ~7,2 h |
| 99.9% (three 9's) | ~8,77 horas | ~43 min |
| 99.99% (four 9's) | ~52,6 min | ~4,3 min |
| 99.999% (five 9's) | ~5,26 minutos | ~26 s |

**Cómo se acumula la disponibilidad:**
- **En serie** (todos deben funcionar): se **multiplican**. Web 99.9% → App 99.9% → DB 99.9% = **99.7%** (cada capa baja el total).
- **En paralelo** (basta uno): la *falla* se multiplica. Dos servidores de 99% → fallan ambos = 1%×1% = 0.01% → disponibilidad **99.99%**.
- Moraleja: la redundancia solo sirve si elimina el **[[single-point-of-failure|punto único de falla]]** (dos web detrás de un LB en una sola AZ siguen dependiendo de esa AZ).

### FT

El sistema **sigue funcionando correctamente con fallos activos**. Más caro y difícil que HA: HA acepta una interrupción breve, FT no.

> Analogía: HA = aeropuerto con varias pistas (te reacomodás). FT = avión con 4 motores (falla uno **en vuelo** y seguís volando).

### DR

Qué pasa si el edificio se inunda/incendia. Se mide con:

- **RPO** (Recovery Point Objective): cuántos **datos** tolerás perder (mira hacia atrás → frecuencia de backups).
- **RTO** (Recovery Time Objective): cuánto **tiempo** tolerás estar caído (mira adelante → velocidad de recuperación).
- Regla económica: bajar RPO a cero exige replicación **síncrona**; bajar RTO a minutos exige infra **ya encendida** en otra region. Cuanto más chicos, exponencialmente más caro.

**Las 4 estrategias de DR (de más barata a más cara):**

| Estrategia | Cómo funciona | RTO/RPO | Costo |
|---|---|---|---|
| **Backup & Restore** | Solo backups en otra region; se levanta todo de cero | Horas–días | 💲 |
| **Pilot Light** | Datos replicándose; cómputo apagado pero listo | Decenas de min | 💲💲 |
| **Warm Standby** | Copia reducida **corriendo** en otra region; se escala | Minutos | 💲💲💲 |
| **Multi-Site Active/Active** | Dos regiones sirviendo tráfico simultáneo | Casi cero | 💲💲💲💲 |

## Cómo aplica en AWS

- HA: [[multi-az|multi-AZ]] ([[global-infrastructure]]), Auto Scaling + ELB, RDS Multi-AZ ([[failover]] 60–120 s).
- FT: redundancia activa-activa en cada componente ([[S3]], DynamoDB, Aurora con réplicas).
- DR: backups/replicación cross-region ([[S3]] CRR); un plan de DR **no probado no es un plan** (game days).

## Preguntas de examen frecuentes

- Palabras que delatan cada concepto:
  - "minimizar downtime", "recuperación rápida", "failover automático" → **HA**
  - "sin interrupción", "cero downtime", "el usuario no debe notar nada" → **FT**
  - "después de un desastre", "otra region", "restaurar" → **DR**
- Escenario "no puede perder ni una lectura ni por segundos" (ej: monitoreo de pacientes) → **FT**, no HA (HA acepta un failover de 60 s).
- Elección de estrategia DR: "el más económico" → Backup & Restore; "RTO de minutos con costo controlado" → Pilot Light / Warm Standby; "sin interrupción" → Active/Active.
- RPO/RTO chicos = más caro; el examen espera que elijas la estrategia DR que *justo* cumple los números.
