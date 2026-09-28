---
title: Containers (Docker)
category: concept
tags: [containers, docker, virtualizacion, images, registry, compute]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.01 Containers — Virtualización vs Containerization.md", "raw/notas curso mejorado/08 Containers, ECS y ECR/08.02 Docker Images, Containers y Registries.md", "raw/notas curso mejorado/08 Containers, ECS y ECR/08.03 Demostración - Container of cats (Docker en EC2).md"]
updated: 2026-09-28
---

# Containers (Docker)

## Definición

Un **container** es un entorno aislado donde corre una aplicación, igual que una VM, pero **sin un sistema operativo propio**: corre como **un proceso** dentro del **host OS**, gestionado por un **container engine** (Docker es el más conocido). Adentro tiene su propio filesystem y sus procesos hijos aislados, y usa el host OS para networking y file I/O.

| | VM ([[virtualization]]) | Container |
|---|---|---|
| Qué corre | Un **guest OS** completo sobre un [[hypervisor]] | Un **proceso** sobre el host OS + container engine |
| Peso | El OS puede ocupar el 60–70% del disco y buena parte de la RAM de la VM | Solo la app y su runtime (librerías, dependencias) |
| Duplicación | N VMs con el mismo OS = N copias del OS | Un solo OS compartido |
| Arranque / reinicio | Hay que levantar el OS entero | Arranca un proceso |
| Densidad | Pocas por host | **Muchas más por host** |
| Aislamiento | Total | "Gran parte" del de una VM |

## Cómo aplica en AWS

- Correr Docker a mano en [[EC2]] funciona (es la demo del curso), pero todo lo operativo queda de tu lado.
- [[ECS]] es el orquestador propio de AWS: "**ECS es a los containers lo que EC2 es a las VMs**".
- [[EKS]] es Kubernetes managed, para cuando el requisito es Kubernetes o no quedar atado a AWS.
- [[ECR]] es el registry donde se guardan las images.
- Fargate es el modo [[serverless]] de correrlos (en ECS y en EKS): ver [[ecs-ec2-vs-fargate]].

## Patrones comunes

### Image y container: el modelo de layers

```
Dockerfile ──build──▶ Image (layers read-only) ──run──▶ Container = image + R/W layer
```

- Una **image** es un **stack de layers read-only**, no un disco monolítico. Se crea desde un **Dockerfile**: cada paso crea una **fs layer**, y cada layer guarda **solo los cambios** respecto de la de abajo (arquitectura diferencial).
- Toda image parte de una **base image** (`FROM centos:7`) o de `scratch` (vacía).
- Un **container** es una copia en ejecución de una image **+ una R/W layer propia**, donde queda todo lo que escribe (logs, datos).
- N containers de la misma image **comparten las layers read-only**: lo único propio de cada uno es su R/W layer (*shared fs layers = efficient*).
- La analogía con EC2: una instancia es una copia en ejecución de sus volúmenes [[EBS]]; un container es una copia en ejecución de su image.

### Instrucciones del Dockerfile que conviene reconocer

| Instrucción | Para qué |
|---|---|
| `FROM` | Base image |
| `RUN` | Ejecuta comandos al construir; varios comandos encadenados con `&&` en un solo `RUN` = **una sola layer** |
| `ADD` / `COPY` | Mete archivos en la image |
| `EXPOSE` | Documenta el puerto en el que escucha el container |
| `CMD` | Qué se ejecuta al arrancar el container |

### Registry

```
Dockerfile → build → image → push → registry (Docker Hub, ECR) → pull → Docker host(s)
```

Un registry guarda muchas images, y cada host baja solo las que necesita. En AWS el registry es [[ECR]].

### Comandos básicos (demo del curso)

| Comando | Qué hace |
|---|---|
| `docker build -t nombre .` | Construye la image desde el Dockerfile del directorio |
| `docker run -p 80:80 nombre` | Arranca un container y mapea `puerto host:puerto container` |
| `docker tag` + `docker push` | Renombra la image con el destino del registry y la sube |

## Preguntas de examen frecuentes

- "Correr más aplicaciones en el mismo hardware, sin un OS completo por app" → **containers** (densidad).
- "La app corre igual en la laptop del dev y en producción" → containers son **portables** / self-contained.
- "Dónde se guardan las images en AWS" → [[ECR]], no S3.
- Lo que un container escribe vive en su **R/W layer** y se pierde con el container. Para datos persistentes hacen falta volúmenes (en ECS, **EFS**; ver [[ECS]]).

## Demos del curso

- [DEMO — Creating "container of cats" Docker image](https://learn.cantrill.io/courses/1101194/lectures/36184903): instalar Docker en EC2, `docker build`, `docker run -p 80:80` y push a Docker Hub. Para subirla a ECR, ver [[ECR]].

> 📖 Lectura profunda: [[08.01 Containers — Virtualización vs Containerization]] · [[08.02 Docker Images, Containers y Registries]] · [[08.03 Demostración - Container of cats (Docker en EC2)]]
