---
title: EC2 Instance Types
category: concept
tags: [ec2, instance-types, performance, costos]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.03 EC2 Instance Types.md"]
updated: 2026-09-24
---

# EC2 Instance Types

## Definición

Elegir un **instance type** y un **size** define, de una sola vez, mucho más que "cuánta CPU y RAM":

- **Cantidad bruta de recursos**: vCPU, memoria, almacenamiento local y de qué tipo.
- **Proporciones entre recursos** (*resource ratios*): uno da mucha CPU y poca memoria; otro, pensado para caché, prioriza memoria por dólar.
- **Costo por segundo**.
- **Ancho de banda de red**, para datos **y para almacenamiento**. Esto es clave con [[EBS]], que es almacenamiento por red: podés aprovisionar volúmenes rapidísimos, pero si la instancia no tiene suficiente ancho de banda, **la instancia es el cuello de botella**. De ahí el concepto de [[ebs-optimized]].
- **Arquitectura de hardware**: ARM (Graviton) vs x86, Intel vs AMD.
- **Capacidades extra**: GPU, FPGA, NVMe local, networking mejorado.

## Cómo aplica en AWS

### Las cinco categorías

| Categoría | Para qué | Ejemplos del curso |
|---|---|---|
| **General Purpose** | El default y casi siempre el punto de partida. Recursos parejos, cargas steady-state | A1, M6g (Graviton), **T3/T3a** (burst pool), M5/M5a/M5n |
| **Compute Optimized** | Mucho procesamiento: media encoding, HPC, modelado científico, gaming, ML | C5, C5n |
| **Memory Optimized** | Mucha memoria por dólar: datasets en memoria, cachés, ciertas DB | R5/R5a, X1/X1e, High Memory (u-Xtb1), z1d |
| **Accelerated Computing** | GPUs y hardware programable | P3 (Tesla v100), G4 (NVIDIA T4), F1 (FPGA), Inf1 |
| **Storage Optimized** | Almacenamiento local enorme y rapidísimo, [[iops\|IOPS]] o [[throughput\|throughput]] secuencial | I3/I3en (NVMe), D2 (dense HDD), H1 |

Las de la familia **T** son el *burst pool* de cómputo: más baratas asumiendo uso bajo con picos ocasionales, y funcionan con el mismo modelo de [[burst-credit|créditos]] que gp2 — si el balde se vacía, la CPU queda limitada al baseline.

> [!info] Actualización posterior al curso
> Los types concretos de arriba son los que existían al grabarse el curso; desde entonces salieron varias generaciones (M7/M8g, C7/C8g, R7/R8g, Graviton 3 y 4, Trn/Inf2…). **No hace falta reaprender el catálogo**: lo que se evalúa es el esquema de nombres y saber a qué categoría pertenece una familia por su letra inicial, y eso no cambió.

### El esquema de nombres

Tomando **`R5dn.8xlarge`**:

| Parte | Qué es | En el ejemplo |
|---|---|---|
| `R` | **Instance Family** | Memory Optimized |
| `5` | **Instance Generation** | 5ª — conviene usar la más nueva disponible |
| `dn` | **Additional Capabilities** | `d` = NVMe local, `n` = networking mejorado ([[enhanced-networking]]) |
| `8xlarge` | **Instance Size** | El tamaño |

El conjunto completo es el **Instance Type**.

## Patrones comunes

Las letras iniciales que conviene reconocer de un vistazo:

```
T  → burst (barato, picos ocasionales)      C  → CPU
M  → general purpose (equilibrado)          R, X, z, u → memoria
I, D, H → storage local                     P, G, F, Inf → GPU / FPGA / ML
```

Y los sufijos más frecuentes: `g` = Graviton (ARM), `a` = AMD, `n` = networking mejorado, `d` = NVMe local.

## Preguntas de examen frecuentes

- *"Carga en memoria, cachés, analítica en tiempo real"* → familia **R** (o X para escala extrema).
- *"Media encoding / HPC / modelado"* → familia **C**.
- *"Uso bajo con picos ocasionales, lo más barato posible"* → familia **T**, aceptando los créditos.
- *"La CPU se degrada después de un rato en una t3"* → **créditos de CPU agotados**; subir de tamaño, pasar a M, o activar *unlimited*.
- *"Aprovisioné mucho IOPS en EBS pero no los alcanzo"* → el límite es de la **instancia** ([[ebs-optimized]]), no del volumen.
- *"Reducir costo sin tocar la aplicación"* → probar **Graviton** (`g`) si el software corre en ARM; si no, revisar [[ec2-purchase-options]].
- Cambiar de type implica **detener la instancia** — es escalado vertical, con su downtime ([[horizontal-vs-vertical-scaling]]).

## Ver también

[[EC2]] · [[ebs-optimized]] · [[horizontal-vs-vertical-scaling]] · [[ec2-purchase-options]] · [[burst-credit]]
