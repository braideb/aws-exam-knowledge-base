---
title: Amazon EKS (Elastic Kubernetes Service)
category: service
tags: [eks, kubernetes, containers, compute, fargate]
exam: [SAA-C03, DVA-C02, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.09 Kubernetes 101.md", "raw/notas curso mejorado/08 Containers, ECS y ECR/08.10 Elastic Kubernetes Service (EKS) 101.md"]
updated: 2026-09-28
---

# Amazon EKS (Elastic Kubernetes Service)

> ⚠️ **Sin contraste con doc oficial:** sale solo de las notas del curso (08.09–08.10); no hay clippings de EKS en `raw/doc oficial/`. Para DVA-C02, EKS pesa bastante menos que [[ECS]]: lo que más se pregunta es cuándo elegirlo y cómo se le dan permisos a un pod.

## ¿Qué es?

**Kubernetes managed por AWS**: es el mismo Kubernetes open source de cualquier otro lado, extendido para integrarse con AWS. AWS administra el **[[control-plane|control plane]]** (y su etcd), que escala solo y corre en **varias AZs**. Vos elegís cómo proveer los **nodes** donde corren los pods.

## Casos de uso

- Ya tenés workloads en **Kubernetes** y querés moverlos a AWS sin reescribirlos.
- Querés containers **sin [[vendor-lock-in]]**: Kubernetes es **cloud agnostic**.
- Correr el mismo modelo en AWS, en **Outposts** (una pequeña versión de AWS on-premises), con **EKS Anywhere** (clusters on-premises o en cualquier lado) o con **EKS Distro** (la distribución open source).

## Características clave

### Kubernetes: lo mínimo

| Pieza | Qué es |
|---|---|
| **Cluster** | Recursos de cómputo highly available organizados como una sola unidad |
| **Control plane** | Administra el cluster: scheduling, apps, scaling, deploys |
| **Node** | VM o servidor físico que funciona como worker; ahí corren los pods |
| **Pod** | La **unidad más chica**: 1+ containers con storage y red compartidos. Lo común es 1 container por pod; varios solo si están **tightly coupled**. **No son permanentes** |
| **Service** | Abstracción de una app que corre en 1+ pods |
| **Job** | Crea pods ad hoc hasta que la tarea termina |
| **Ingress** | Entrada desde afuera: Ingress ⇒ routing ⇒ Service ⇒ pods. El **ingress controller** lo implementa (en AWS, el **AWS Load Balancer Controller**, con ALB/NLB) |
| **Persistent Volume (PV)** | Volumen cuyo ciclo de vida va más allá de cualquier pod |

**Componentes del control plane:** `kube-apiserver` (el front end; lo que usan nodes y demás, escala horizontalmente), `etcd` (key-value store HA, backing store del cluster), `kube-scheduler` (asigna un node a los pods que no lo tienen, según recursos, affinity y constraints), `kube-controller-manager` (node, job, endpoint, service account/token controllers) y `cloud-controller-manager` (opcional, integra con el cloud provider).

**En cada node:** un container runtime (containerd o Docker), el `kubelet` (agente que habla con el control plane por la **Kubernetes API**) y `kube-proxy` (networking y reglas para llegar a los pods desde dentro y fuera del cluster).

### Qué administra AWS y qué vos

**EKS cluster = EKS control plane (AWS) + EKS nodes.**

| Nodes | Qué implica |
|---|---|
| **Self-managed** | Instancias EC2 que administrás vos; precio normal de EC2, por segundo |
| **Managed node groups** | Siguen siendo EC2, pero EKS aprovisiona y maneja su ciclo de vida |
| **Fargate** | Sin instancias: no elegís instance type ni escalás grupos. Los **Fargate profiles** definen **qué pods** arrancan en Fargate. Similar a ECS Fargate |

Hay tipos de node para Windows, GPU, Inferentia, Bottlerocket, Outposts y Local Zones.

### Permisos de los pods

Los nodes tienen su IAM role, pero lo correcto es darle a **cada pod solo lo que necesita** ([[least-privilege]]), igual que el [[task-role|task role]] en ECS. Se hace con **[[irsa|IRSA]]** o con **EKS Pod Identity** (la opción más nueva y simple): un IAM role se asocia a un **service account** de Kubernetes, y los pods que lo usan reciben [[temporary-credentials|credenciales temporales]] de ese role.

### Storage

El storage de un pod es [[ephemeral-storage|efímero]]. Para persistencia, EKS usa como storage providers **EBS, EFS, FSx for Lustre y FSx for NetApp ONTAP**.

### Arquitectura de red

| Parte | Dónde vive |
|---|---|
| Control plane (managed, [[multi-az\|multi-AZ]]) | **VPC administrada por AWS**, fuera de tu cuenta |
| Worker nodes | **Tu VPC** |
| ENIs del control plane | **Inyectadas en tu VPC**, para hablar con los nodes |
| Public endpoint | Por donde el admin opera el cluster (`kubectl`) |
| Load balancer | En tu VPC; por ahí entran los usuarios de la app, que **nunca** hablan con el control plane |

El tráfico kube-api (kubelet ↔ kube-apiserver) va por las ENIs inyectadas o por el public endpoint, según cómo esté configurado el acceso al endpoint del cluster.

## Integración con otros servicios

- [[ECR]] — images de los pods.
- ELB — cualquier load balancer que pida Kubernetes (ingress, services).
- [[IAM]] — IRSA / Pod Identity. `AssumeRoleWithWebIdentity` es la llamada STS que usan los pods con IRSA.
- [[VPC]] — networking de nodes y pods; ENIs del control plane en tu VPC.
- [[EBS]], EFS, FSx — persistent storage.

## Gotchas y trampas del examen

- Containers + "ya usamos Kubernetes", "cloud agnostic" o "evitar vendor lock-in" → **EKS**. Sin requisito de Kubernetes → **[[ECS]]** (más simple, nativo de AWS).
- "Darle a un pod permiso para S3" → **IRSA / EKS Pod Identity**, no el role del node (le daría el permiso a todos los pods del node).
- "Correr pods sin administrar nodes" → **Fargate** con un **Fargate profile**.
- El **control plane y etcd los administra AWS** y son multi-AZ; los nodes (salvo Fargate) siguen siendo EC2 que pagás.
- Los pods son efímeros: datos que tienen que sobrevivir → persistent volume sobre EBS/EFS/FSx.

> 📖 Lectura profunda: [[08.09 Kubernetes 101]] · [[08.10 Elastic Kubernetes Service (EKS) 101]]
