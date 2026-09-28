---
title: ECS — EC2 mode vs Fargate
category: comparison
tags: [ecs, fargate, containers, costos, networking, compute]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.05 ECS — Cluster Types (EC2 y Fargate).md", "raw/notas curso mejorado/08 Containers, ECS y ECR/08.06 EC2 vs ECS (EC2) vs Fargate.md"]
updated: 2026-09-28
---

# ECS — EC2 mode vs Fargate

Los dos modos de cluster de [[ECS]]. En los dos existen los mismos componentes de management (scheduling and orchestration, cluster manager, placement engine), las mismas task definitions y service definitions y los mismos registries. Lo que cambia es **dónde corren los containers** y **cuánto administrás vos**.

## Tabla comparativa

| | EC2 mode | Fargate mode |
|---|---|---|
| Dónde corren las tasks | **Container instances** (EC2) en tu VPC, multi-AZ, gestionadas por un **ASG** | **Fargate shared infrastructure** de AWS; cada task se **inyecta en tu VPC** con su [[eni\|ENI]] |
| Qué administrás | Capacidad y disponibilidad de los hosts | Nada a nivel host |
| [[serverless\|Serverless]] | No | Sí |
| Qué pagás | Las **instancias EC2, estén usadas o no** | **vCPU + memoria asignadas** a cada task, por segundo, **aunque la app use menos** |
| Descuentos | Reserved Instances, Savings Plans, Spot | Compute Savings Plans, **Fargate Spot** |
| Escalar la capacidad | Aparte: [[capacity-provider]] + ASG | No aplica |
| Placement strategies / constraints | Sí (`binpack`, `spread`, `random`, `distinctInstance`, `memberOf`) | **No**: AWS ubica las tasks y las reparte entre AZs |
| Network mode | `awsvpc`, `bridge`, … | **Solo `awsvpc`** |
| IP pública | La de la instancia | La task puede tenerla si la subnet asigna IPv4 pública ([[public-subnet]]) |
| Acceso a los hosts | Sí, son tus instancias (podés conectarte) | No |

## Networking: cómo ve el ALB a las tasks

| Network mode | Qué recibe cada task | Target group del ALB |
|---|---|---|
| `awsvpc` (obligatorio en Fargate) | Su **propia ENI e IP** en la VPC | Tipo **`ip`**, apunta directo a cada task |
| `bridge` (EC2 mode) | Un puerto del host | [[dynamic-port-mapping\|Dynamic port mapping]]: host port `0` → Docker asigna un [[ephemeral-port\|puerto efímero]] y ECS registra `instancia:puerto` en el target group |

Con `bridge`, el SG de las instancias tiene que aceptar el **rango de puertos efímeros** desde el SG del ALB.

## Cuándo usar X vs Y

| Escenario | Respuesta |
|---|---|
| Usás containers | **ECS** (y no Docker a mano en EC2) |
| Workload **grande** y **sensible al precio** | **EC2 mode** |
| Workload **grande** y **sensible al overhead** de administración | **Fargate** |
| Workloads **chicos** o **en ráfagas** | **Fargate** |
| Workloads **batch** o **periódicos** | **Fargate** |
| Necesitás controlar el host (instance type concreto, GPU, acceso al host) | **EC2 mode** |

Por qué: EC2 mode gana en precio cuando el cluster está **bien utilizado de forma constante** y se aprovechan RIs o Spot. Fargate gana cuando no querés administrar hosts o la carga es intermitente, porque **no pagás hosts ociosos** entre ráfagas. EC2 mode es el **punto medio** entre Docker a mano en EC2 y Fargate: ECS le saca parte del overhead, pero conservás el de la capacidad (y algo de flexibilidad).

## Trampa típica del examen

- "Sin administrar servidores ni capacidad" → **Fargate**. "Aprovechar Reserved Instances que ya compramos" → **EC2 mode**.
- "Cluster en EC2 mode sin tasks corriendo, ¿por qué sigue costando?" → las container instances se pagan igual.
- "Bajar costos en Fargate" → **right-sizing** de CPU/memoria de la task o **Fargate Spot**. No hay instance type que cambiar.
- "Usar `binpack` en Fargate" → no existe; las placement strategies son solo de EC2 mode.
- "Registrar tasks de Fargate en el target group" → tipo **`ip`**, no `instance`.
- En EC2 mode, tasks que no arrancan por falta de recursos → el service auto scaling **no** agrega instancias; eso lo hace el capacity provider.

> 📖 Lectura profunda: [[08.05 ECS — Cluster Types (EC2 y Fargate)]] · [[08.06 EC2 vs ECS (EC2) vs Fargate]]
