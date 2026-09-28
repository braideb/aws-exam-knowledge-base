---
title: Amazon ECS (Elastic Container Service)
category: service
tags: [ecs, containers, fargate, compute, orquestacion, deployment]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.04 ECS — Concepts.md", "raw/notas curso mejorado/08 Containers, ECS y ECR/08.05 ECS — Cluster Types (EC2 y Fargate).md", "raw/notas curso mejorado/08 Containers, ECS y ECR/08.06 EC2 vs ECS (EC2) vs Fargate.md", "raw/notas curso mejorado/08 Containers, ECS y ECR/08.07 Demostración - Container of cats en Fargate.md"]
updated: 2026-09-28
---

# Amazon ECS (Elastic Container Service)

> ⚠️ **Sin contraste con doc oficial:** esta página sale solo de las notas del curso (módulo 08); no hay clippings de ECS en `raw/doc oficial/`. Los tres roles, el scaling, los deploys, las placement strategies y los network modes vienen de la nota, pero son el material más preguntado de ECS: conviene cruzarlos con la doc cuando haya clippings. **No vienen del curso** y hay que verificarlos: el log driver `awslogs`, los estados `PROVISIONING`/`PENDING` de una task sin capacidad y el ejemplo 100 / 200 de `minimumHealthyPercent` / `maximumPercent`.

## ¿Qué es?

El **orquestador de [[containers]] propio de AWS**: recibe container images y la definición de cómo correrlas, y decide **dónde y cómo** ejecutarlas dentro de un **cluster**. Es managed y tiene dos modos de cluster, **EC2** (containers sobre instancias tuyas) y **Fargate** ([[serverless]]). Resumen del curso: **ECS es a los containers lo que [[EC2]] es a las VMs**.

## Casos de uso

- Correr apps en containers sin armar Docker a mano en EC2.
- Microservicios: cada servicio es una task con su propio scaling y su propio IAM role.
- Workloads batch, periódicos o en ráfagas sobre Fargate (sin pagar hosts ociosos).
- Workloads grandes y constantes sobre EC2 mode, para exprimir Reserved Instances o Spot.

## Características clave

### Los building blocks

| Pieza | Qué define |
|---|---|
| **Cluster** | Dónde corren las tasks y los services (EC2 o Fargate) |
| **Container definition** | **Solo** la image (dónde está) y los **puertos** |
| **Task definition** | La aplicación: uno o más containers, **CPU y memoria**, **network mode**, compatibilidad (EC2/Fargate), **volúmenes** y el **[[task-role\|task role]]** |
| **Task** | Una copia en ejecución de la task definition. Por sí sola **no escala ni tiene HA** |
| **Service** (service definition) | **Cuántas copias** de la task, reemplazo de tasks caídas, scaling, load balancer delante |

- Task ≠ container: una task puede tener varios containers. El caso típico es el **[[sidecar]]**, como el daemon de [[XRay]] corriendo al lado de la app.
- Storage persistente y compartido entre tasks → **EFS** (funciona también en Fargate).
- Para una copia suelta alcanza con una task. Para algo de larga duración y business critical → **service**.

### Los tres roles

| Rol | Lo usa | Para |
|---|---|---|
| **[[task-role\|Task role]]** | Tu código dentro del container | Llamar a S3, DynamoDB, etc. con [[temporary-credentials\|credenciales temporales]] |
| **Task execution role** | El ECS agent / Fargate | **Pull** de la image desde [[ECR]], **logs** a [[CloudWatchLogs]], leer **secrets** (Secrets Manager / Parameter Store) referenciados en la task definition |
| **Container instance role** | El ECS agent (**solo EC2 mode**) | Es el [[instance-profile]] de las instancias; las registra en el cluster |

El task role es la best practice para darle permisos a un container, igual que el [[execution-role|execution role]] en Lambda. Según el instructor, aparece en al menos una pregunta del examen.

### Cluster types

| | EC2 mode | Fargate mode |
|---|---|---|
| Dónde corren | Container instances (EC2) en tu VPC, gestionadas por un ASG | Fargate shared infrastructure, **inyectadas en tu VPC** con una [[eni\|ENI]] por task |
| Admin overhead | Capacidad y disponibilidad de los hosts: tuyas | Ninguno sobre hosts |
| Pagás | Las instancias, **aunque estén vacías** | vCPU + memoria **asignadas** a la task, por segundo |

Comparación completa, incluidos networking y cuándo elegir cada uno: [[ecs-ec2-vs-fargate]].

### Scaling: dos capas distintas

| Qué escala | Mecanismo |
|---|---|
| **Tasks** de un service | **Service auto scaling** (Application Auto Scaling), típicamente [[target-tracking-scaling\|target tracking]] sobre CPU, memoria o requests por target del ALB |
| **Instancias** del cluster (solo EC2 mode) | **[[capacity-provider\|Capacity provider]]** asociado al ASG (ECS cluster auto scaling) |

### Placement (solo EC2 mode)

| Tipo | Opción | Efecto |
|---|---|---|
| Strategy | `binpack` | Llena cada instancia (CPU o memoria) antes de usar otra → **menos instancias, más barato** |
| | `spread` | Reparte por un atributo, por ejemplo por AZ → **HA** |
| | `random` | Al azar |
| Constraint | `distinctInstance` | Cada task en una instancia distinta |
| | `memberOf` | Solo instancias que cumplan una expresión (instance type, AZ…) |

### Deployments de un service

Se registra una **nueva revisión de la task definition** y se actualiza el service para que la use:

| Tipo | Cómo |
|---|---|
| **[[rolling-deployment\|Rolling update]]** | Reemplaza tasks viejas por nuevas. `minimumHealthyPercent`: mínimo de tasks (% del desired count) que siguen corriendo · `maximumPercent`: máximo que puede haber corriendo a la vez |
| **[[blue-green-deployment\|Blue/green]] con CodeDeploy** | La versión nueva se levanta al lado de la vieja y el tráfico del ALB se pasa de una a otra: todo de una vez, **canary** o **linear** |

## Integración con otros servicios

- [[ECR]] — registry de las images; el pull lo hace el **task execution role**.
- [[IAM]] — task role, execution role e instance role.
- [[CloudWatchLogs]] — logs de los containers (driver `awslogs`), vía execution role.
- [[XRay]] — el daemon corre como [[sidecar]] dentro de la task.
- [[VPC]] — Fargate (y `awsvpc` en general) le da una ENI por task; los SGs se aplican a la task.
- [[EC2]] — container instances en EC2 mode; se pueden usar Reserved Instances y Spot ([[ec2-purchase-options]]).
- ALB — delante de un service; target type `ip` con `awsvpc`, [[dynamic-port-mapping]] con `bridge`.
- CodeDeploy — deploys blue/green; CodeBuild para construir y subir la image (ver [[ECR]]).

## Gotchas y trampas del examen

- **"La task no puede hacer pull de la image"** o **"no aparecen los logs"** → **task execution role**. **"La app no puede leer de S3"** → **task role**. No se le dan permisos a la app vía el rol de la instancia.
- **Service auto scaling agrega tasks, no instancias.** En EC2 mode, tasks nuevas que quedan en `PROVISIONING`/`PENDING` por falta de CPU o memoria → falta un **capacity provider** que escale el ASG.
- "Minimizar la cantidad de instancias" → `binpack`. "Repartir entre AZs para HA" → `spread`. En **Fargate no hay** placement strategies ni constraints.
- Fargate es **siempre `awsvpc`** → target group de **tipo `ip`**. `bridge` + host port `0` → dynamic port mapping, y el SG de las instancias tiene que permitir el rango de [[ephemeral-port|ephemeral ports]] desde el SG del ALB.
- **Costos en Fargate:** pagás lo **asignado**, no lo usado → **right-sizing** de la task o **Fargate Spot** para tasks que toleran interrupciones. En EC2 mode pagás las instancias **aunque no corra ninguna task**.
- Deploy sin downtime con **rolling**: `minimumHealthyPercent` = 100 mantiene toda la capacidad; `maximumPercent` = 200 deja levantar las nuevas antes de bajar las viejas. Probar de a poco con tráfico real y volver atrás rápido → **blue/green con CodeDeploy** (canary/linear).
- Datos que tienen que sobrevivir a la task o compartirse entre tasks → **EFS**, no el filesystem del container.
- Tracing en ECS → el daemon de X-Ray como **sidecar**, con `xray:PutTraceSegments` en el **task role**.

## Demos del curso

- [DEMO — Deploying "container of cats" using Fargate](https://learn.cantrill.io/courses/1101194/lectures/36185027): una task definition con una sola container definition, ejecutada como task suelta (sin service).
- La image se construye en la [demo de Docker en EC2](https://learn.cantrill.io/courses/1101194/lectures/36184903) (ver [[containers]]).

> 📖 Lectura profunda: [[08.04 ECS — Concepts]] · [[08.05 ECS — Cluster Types (EC2 y Fargate)]] · [[08.06 EC2 vs ECS (EC2) vs Fargate]] · [[08.07 Demostración - Container of cats en Fargate]]
