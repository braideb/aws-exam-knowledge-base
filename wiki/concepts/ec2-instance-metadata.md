---
title: Instance Metadata Service (IMDS)
category: concept
tags: [ec2, seguridad, iam, imds, ssrf, credenciales]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.24 Instance Metadata (IMDS).md"]
updated: 2026-09-22
---

# Instance Metadata Service (IMDS)

## Definición

El **Instance Metadata Service** le da a una instancia [[EC2]] información **sobre sí misma**: su networking, su entorno y —lo más importante para el examen— las [[temporary-credentials|credenciales temporales]] del [[instance-profile|IAM role]] que tenga asociado.

Se accede siempre igual, desde cualquier instancia, por una **IP fija**:

> **`169.254.169.254`**

Es una dirección link-local: no sale de la instancia y no consume red de la VPC. Memorizarla conviene — aparece en casi todos los exámenes de AWS.

## Cómo aplica en AWS

La ruta base es `http://169.254.169.254/latest/meta-data/`. Desde ahí se consulta:

| Categoría | Ejemplos |
|---|---|
| Networking | `public-ipv4`, `local-ipv4`, `public-hostname`, `mac` |
| Entorno | `placement/availability-zone`, `instance-id`, `instance-type`, `ami-id` |
| Seguridad | `security-groups` |
| **Credenciales** | `iam/security-credentials/<role>` → access key, secret y **token de sesión** |

Esa última fila es la razón de ser del tema: **así es como el SDK de AWS obtiene credenciales dentro de una instancia**, sin que haya access keys guardadas en disco. Es el mecanismo detrás de los [[instance-profile|instance profiles]].

### IMDSv1: el problema

En **IMDSv1** alcanza un `GET` simple:

```bash
curl http://169.254.169.254/latest/meta-data/public-ipv4
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/mi-role
```

> ⚠️ El servicio **no requiere autenticación y no está cifrado**. Cualquier proceso dentro de la instancia puede consultarlo. Si un atacante logra ejecutar código en la instancia —o, peor, si encuentra un **SSRF** en una app web que corre ahí— puede pedirle al servidor que consulte esa URL y **robar las credenciales del role**.

### IMDSv2: la mitigación

> [!warning] El curso solo cubre IMDSv1
> **IMDSv2 es hoy el default en instancias nuevas** y es material de examen. Es **orientado a sesión**: primero se pide un token con un `PUT`, y después ese token viaja en un header en cada consulta.

```bash
# 1) obtener el token (TTL en segundos, máx. 21600)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# 2) usarlo en cada request
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/
```

Por qué esto corta el ataque:

- Un SSRF clásico solo sabe pedir una **URL por `GET`**. No puede hacer un `PUT` ni agregar un **header personalizado**, así que nunca obtiene el token.
- La respuesta del token trae un **hop limit** que impide que el pedido salga de la instancia (por ejemplo, a través de un proxy o un contenedor mal configurado).

Se exige por instancia con **`HttpTokens: required`** (el modo opuesto, `optional`, acepta las dos versiones).

## Patrones comunes

```
App web en EC2 con IAM role
        │
        ├── IMDSv1 (GET simple)     → un SSRF alcanza para robar las credenciales
        └── IMDSv2 (PUT + header)   → el SSRF no puede armar el request
```

## Preguntas de examen frecuentes

- *"¿Cómo obtiene credenciales el SDK dentro de una instancia?"* → del **IMDS**, vía el instance profile. Nunca hardcodear access keys.
- *"Proteger las credenciales del role frente a un SSRF"* → **exigir IMDSv2** (`HttpTokens: required`). Es la respuesta esperada, por encima de "usar un WAF" o "rotar las claves".
- *"¿Cómo sabe una instancia en qué AZ está?"* → `placement/availability-zone` del IMDS.
- Dato fino: el tráfico al IMDS **no aparece en los [[vpc-flow-logs|VPC Flow Logs]]** — es una de las cosas que explícitamente no capturan.
- Las credenciales que entrega son **temporales y rotan solas**: no hay que renovarlas a mano.

## Demos del curso

- [Viendo los metadatos en una instancia EC2](https://learn.cantrill.io/courses/1101194/lectures/27806481) — consultar `public-ipv4` y `public-hostname` con `curl`, y el script `ec2-metadata` (`-a` ami-id, `-z` availability zone, `-s` security groups).

## Ver también

[[EC2]] · [[instance-profile]] · [[temporary-credentials]] · [[IAM]] · [[dva-security]]
