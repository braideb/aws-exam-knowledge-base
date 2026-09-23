---
title: EC2 Purchase Options
category: comparison
tags: [ec2, costos, spot, reserved, savings-plans, dedicated]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.18 EC2 Purchase Options - On-Demand y Spot.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.19 Reserved Instances y Savings Plans.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.20 Dedicated Hosts y Dedicated Instances.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.21 Capacity Reservations.md"]
updated: 2026-09-22
---

# EC2 Purchase Options

El término oficial de AWS es **purchase options** (no "tipos de lanzamiento"). Son formas distintas de pagar la misma capacidad de [[EC2]].

## La tabla

| Opción | Descuento | ¿Compromiso? | ¿Garantiza capacidad? | ¿Puede interrumpirse? | Para qué |
|---|---|---|---|---|---|
| **On-Demand** | — (precio base) | No | No | No | Duración desconocida, sin tolerancia a cortes |
| **Spot** | El mayor (hasta ~90%) | No | No | **Sí, AWS te la quita** | Batch, render, testing |
| **Reserved (regional)** | Alto | 1 o 3 años | **No** | No | Uso constante y conocido |
| **Reserved (zonal)** | Alto | 1 o 3 años | **Sí**, en esa AZ | No | Uso constante + garantía de lanzar |
| **Savings Plans** | Hasta 66% / 72% | 1 o 3 años (por $/hora) | No | No | Uso constante con flexibilidad |
| **On-Demand Capacity Reservation** | **Ninguno** | No | **Sí**, en esa AZ | No | Garantizar capacidad sin atarse |
| **Dedicated Instances** | — | No | No | No | Compliance: no compartir hardware |
| **Dedicated Hosts** | — | Opcional | Sí (es tu host) | No | Licencias por socket/core |

## Las opciones, una por una

### On-Demand

El default y el punto medio. Se cobra **por segundo** mientras la instancia está *running*. Corre sobre hardware **compartido** con otros clientes (aislado a nivel de seguridad y virtualización).

> Matiz que cae: la **instancia** se cobra solo mientras corre, pero los **recursos asociados** —sobre todo los volúmenes [[EBS]]— se siguen cobrando aunque esté detenida.

### Spot

Capacidad libre a precio de remate. Vos declarás un **precio máximo**, pero eso es un **techo, no lo que pagás**: siempre pagás el **Spot price** vigente. Si el Spot price sube por encima de tu máximo, AWS **termina** tus instancias.

> **Regla de oro:** nunca para cargas que no toleren interrupciones, ni de largo plazo, ni que necesiten procesamiento constante y confiable.

### Reserved Instances

Compromiso de 1 o 3 años a cambio de descuento. Se aplica automáticamente a las instancias que "coincidan". Dos ejes:

- **Alcance:** **regional** (flexibilidad de facturación en cualquier AZ, **sin** reserva de capacidad) o **zonal** (mismo descuento **+ capacidad reservada** en esa AZ, pero solo ahí).
- **Pago:** **No Upfront** (menor descuento) → **Partial Upfront** → **All Upfront** (mayor descuento; 3 años All Upfront es el mejor descuento posible en AWS).

Cobertura **parcial**: una reserva de `T3.large` aplicada a una `T3.xlarge` cubre la porción que coincide.

> ⚠️ Se puede comprar una reserva y **no usarla** — la seguís pagando igual. La variante original se llama hoy **Standard Reserved**.

> [!info] Actualización posterior al curso
> Las **Scheduled Reserved Instances** (reserva con ventana horaria, ej. 5 h todos los días, mínimo 1.200 h/año) **ya no se ofrecen a clientes nuevos**. El curso las cubre y aparecieron históricamente en el examen; el concepto sigue siendo útil para entender el resto.

### Savings Plans

Igual que una reserva, pero el compromiso es de **gasto por hora**, no de tipo de instancia. Mucho más flexible.

- **Compute Savings Plan** — hasta **66%**, aplica a **EC2, Fargate y Lambda**.
- **EC2 Savings Plan** — hasta **72%**, solo EC2, con flexibilidad de size y OS.

Se cobra la tarifa reducida hasta agotar el monto comprometido por hora; lo que exceda va a precio On-Demand.

### Dedicated Hosts vs Dedicated Instances

| | **Dedicated Host** | **Dedicated Instances** |
|---|---|---|
| Qué pagás | **El host completo** | Cada instancia (+ tarifa por región) |
| Quién comparte el hardware | Nadie | Solo instancias **tuyas** |
| Visibilidad de sockets/cores | **Sí** | No |
| [[host-affinity\|Host Affinity]] | **Sí** | No |
| Motivo típico | **Licencias por socket o core** | **Compliance / regulación** |

Ver también [[dedicated-tenancy]].

### Capacity Reservations

Cuando hay escasez, AWS asigna capacidad en este orden: **1) reservas → 2) On-Demand → 3) Spot**.

Lo central: **facturación y capacidad son dos cosas separadas** que se pueden usar juntas o por separado. Una **On-Demand Capacity Reservation** reserva capacidad en una AZ concreta **sin** compromiso de 1–3 años y **sin** descuento — y **pagás por esa capacidad la uses o no**.

## Trampa típica del examen

- **"Garantizar que voy a poder lanzar"** (un evento con fecha, un DR, un proceso crítico) → hace falta **capacidad reservada**: reserva **zonal** u **On-Demand Capacity Reservation**. Una reserva **regional** o un **Savings Plan** solo dan descuento, **no garantizan nada**.
- **Spot**: las palabras que lo delatan son "batch", "tolerante a fallos", "puede reintentarse", "el costo es lo más importante". Las que lo descartan: "no puede interrumpirse", "producción 24/7", "crítico".
- Tu precio máximo en Spot es un **techo**, no lo que pagás. Distractor frecuente.
- **Reserved vs Savings Plan**: si el escenario menciona **Fargate o Lambda** además de EC2 → **Compute Savings Plan**. Si ata el descuento a un tipo de instancia concreto → Reserved.
- **Dedicated Host vs Dedicated Instances**: decide el **motivo**. Licencias por socket/core → **Host**. "No podemos compartir hardware por regulación" → **Instances**.
- Detener la instancia **no frena el costo de EBS**.

## Ver también

[[EC2]] · [[observability-costs]] · [[dedicated-tenancy]] · [[host-affinity]] · [[ec2-instance-types]]
