---
title: Placement Groups — Cluster vs Spread vs Partition
category: comparison
tags: [ec2, placement-groups, rendimiento, resiliencia, hpc]
exam: [SAA-C03, DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/09 Advanced EC2/09.08 Placement Groups — Overview.md", "raw/notas curso mejorado/09 Advanced EC2/09.09 Cluster Placement Groups.md", "raw/notas curso mejorado/09 Advanced EC2/09.10 Spread Placement Groups.md", "raw/notas curso mejorado/09 Advanced EC2/09.11 Partition Placement Groups.md"]
updated: 2026-09-30
---

# Placement Groups — Cluster vs Spread vs Partition

Por defecto, AWS decide en qué host de la AZ va cada instancia [[EC2]]. Un **placement group** cambia ese criterio: junta las instancias físicamente para ganar **rendimiento**, o las separa para ganar **resiliencia**.

## Tabla comparativa

| | **Cluster** | **Spread** | **Partition** |
|---|---|---|---|
| Objetivo | **Rendimiento** máximo | **Resiliencia** de cada instancia | Resiliencia por **grupo** de instancias ([[fault-domain\|fault domains]]) |
| Ubicación física | Mismo rack, a veces el mismo host | Cada instancia en un **rack distinto** (red y energía propias) | Cada partición en racks propios; **las particiones no comparten hardware** |
| AZs | **Una sola** (queda fijada por la primera instancia) | Varias | Varias |
| Límite | — | **7 instancias por AZ** | **7 particiones por AZ**, instancias **ilimitadas** |
| Quién decide la ubicación | AWS | AWS | Vos (eligiendo la partición) o EC2 |
| Red | **10 Gbps single-stream** (5 Gbps fuera), latencia mínima, PPS máximo | Normal | Normal |
| Resiliencia | **Baja**: falla el hardware y caen todas | Máxima | Alta, si la app la aprovecha |
| Restricciones | Instance types compatibles; requiere [[enhanced-networking]] para el rendimiento | **No** admite Dedicated Instances ni Dedicated Hosts | — |
| Caso de uso | [[hpc\|HPC]], análisis científico, baja latencia entre nodos | Pocas instancias críticas: domain controllers, réplicas de file servers | Apps [[topology-aware]]: **HDFS, HBase, Cassandra** |

## Detalles por tipo

**Cluster.** Se recomienda lanzar todas las instancias **al mismo tiempo** y con el **mismo instance type**, para que AWS reserve capacidad de una vez. Si se agregan instancias más tarde, puede no haber capacidad en ese rack. Puede abarcar varias VPCs (con [[vpc-peering|peering]]), pero pierde rendimiento.

**Spread.** Cada instancia tiene su propio [[blast-radius]]: si falla un rack, cae una sola instancia. El tope de 7 por AZ existe porque la cantidad de racks aislados de cada AZ es limitada.

**Partition.** Sirve cuando hacen falta **más de 7 instancias por AZ** con separación de hardware. El aislamiento es entre **particiones**, no entre instancias: si ponés 10 instancias en una partición y esa partición falla, perdés las 10. EC2 **expone a qué partición pertenece cada instancia**, y las apps topology-aware usan esa información para replicar en particiones distintas. Ejemplo: 75 instancias con replicación ×3 → cada copia en una partición diferente.

## Cuándo usar X vs Y

| Escenario | Respuesta |
|---|---|
| Latencia de red mínima y máximo throughput entre nodos (HPC, MPI) | **Cluster** + enhanced networking (o EFA) |
| Un puñado de instancias críticas que no pueden caer juntas | **Spread** |
| Cientos de instancias de Cassandra/HDFS repartidas en fault domains | **Partition** |
| Spread con más de 7 instancias por AZ | No se puede: **Partition** |
| Cluster repartido en dos AZs | No se puede: un cluster PG es de **una sola AZ** |

## Trampa típica del examen

- **"Alta disponibilidad" + "baja latencia" en la misma pregunta**: cluster da latencia, pero **no** alta disponibilidad. Leé cuál de las dos se prioriza.
- **Spread vs Partition:** spread = AWS garantiza instancia por instancia, con tope de 7. Partition = sin tope de instancias, pero la que distribuye es la app.
- "No hay capacidad al agregar instancias a un cluster PG" → había que **lanzarlas todas juntas**. *(Complemento, no viene del curso: la doc sugiere detener y volver a iniciar todas las instancias del grupo.)*
- Spread con **Dedicated Hosts** → no es compatible.

> 📖 Lectura profunda: [[09.08 Placement Groups — Overview]] · [[09.09 Cluster Placement Groups]] · [[09.10 Spread Placement Groups]] · [[09.11 Partition Placement Groups]] · [Doc: Placement groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html)
