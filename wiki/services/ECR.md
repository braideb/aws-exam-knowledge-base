---
title: Amazon ECR (Elastic Container Registry)
category: service
tags: [ecr, containers, registry, docker, seguridad, cicd]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.08 Elastic Container Registry (ECR).md", "raw/notas curso mejorado/08 Containers, ECS y ECR/08.02 Docker Images, Containers y Registries.md"]
updated: 2026-09-28
---

# Amazon ECR (Elastic Container Registry)

> ⚠️ **Sin contraste con doc oficial:** sale solo de las notas del curso (08.08); no hay clippings de ECR en `raw/doc oficial/`. Los nombres de las acciones IAM (`ecr:GetAuthorizationToken`, `ecr:BatchGetImage`) **no vienen del curso** y hay que verificarlos.

## ¿Qué es?

El **registry managed de container images** de AWS: el equivalente de Docker Hub integrado con [[IAM]]. Guarda las images que después corren [[ECS]], [[EKS]] o cualquier host con Docker. Ver [[containers]] para el modelo de image y registry.

## Casos de uso

- Guardar las images privadas de una organización con acceso controlado por IAM.
- Destino del paso de build de un pipeline de CI/CD (CodeBuild hace build + push).
- Compartir images con otras cuentas o replicarlas a otras regions.
- Escanear las images en busca de vulnerabilidades antes de desplegarlas.

## Características clave

### Estructura

```
Registry (1 público + 1 privado por cuenta)
 └─ Repository (muchos)
     └─ Image (muchas)
         └─ Tag (varios por image, únicos dentro del repository)
```

- **Tag immutability** (por repository): impide sobrescribir un tag existente, así cada tag apunta siempre a la misma image.

### Público vs privado

| Registry | Leer | Escribir |
|---|---|---|
| Public | **Cualquiera** | Con permisos |
| Private | Con permisos | Con permisos |

### Seguridad y operación

| Feature | Detalle |
|---|---|
| Permisos | **IAM** para todo el acceso + **repository policy** (resource-based) por repository, por ejemplo para [[cross-account]] |
| Image scanning | **Basic** o **Enhanced**. Enhanced usa **Amazon Inspector**: detecta problemas del OS **y** de los paquetes de software, layer por layer |
| Métricas | [[near-real-time\|Near real-time]] en [[CloudWatch]] (autenticación, push/pull) |
| Auditoría | Toda la API en [[CloudTrail]] |
| Eventos | A [[EventBridge]], para workflows event-driven sobre images |
| Replicación | **Cross-region** y **cross-account** |
| Lifecycle policies | Borran images automáticamente (sin tag, o las más viejas pasado un número) para no pagar storage de más |

### Push de una image

```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
docker tag app:latest <account-id>.dkr.ecr.<region>.amazonaws.com/app:latest
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/app:latest
```

1. `get-login-password` → token temporal que `docker login` usa con el usuario `AWS`.
2. `docker tag` → el nombre tiene que ser la **URI del repository**: `<account-id>.dkr.ecr.<region>.amazonaws.com/<repo>:<tag>`.
3. `docker push`.

## Integración con otros servicios

- [[ECS]] — el pull lo hace el **task execution role** (no el [[task-role|task role]]).
- [[EKS]] — registry de las images de los pods.
- CodeBuild — build + tag + push en el pipeline; necesita **privileged mode** para construir images de Docker.
- Amazon Inspector — enhanced scanning.
- [[CloudTrail]], [[CloudWatch]], [[EventBridge]] — auditoría, métricas y eventos.

## Gotchas y trampas del examen

- ECS no puede hacer pull desde ECR → permisos del **task execution role** (`ecr:GetAuthorizationToken`, `ecr:BatchGetImage`…), no del task role.
- Otra cuenta necesita hacer pull → **repository policy** (resource-based), además de IAM del lado de esa cuenta.
- "Garantizar que `v1` siempre sea la misma image" → **tag immutability**.
- "Detectar CVEs en los paquetes de la image" → **enhanced scanning (Inspector)**; basic no cubre los paquetes de software.
- "Dejar de pagar images viejas o sin tag" → **lifecycle policy**, no un script.
- CodeBuild no puede ejecutar `docker build` → falta **privileged mode** en el proyecto.
- `docker login` contra ECR usa un token temporal de `aws ecr get-login-password`, no las access keys directamente.

> 📖 Lectura profunda: [[08.08 Elastic Container Registry (ECR)]]
