---
title: EC2 Bootstrapping (User Data) y AMI baking
category: concept
tags: [ec2, user-data, bootstrapping, ami, deployment, cloud-init]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/09 Advanced EC2/09.01 Bootstrapping EC2 con User Data.md", "raw/notas curso mejorado/09 Advanced EC2/09.02 Boot Time to Service Time y AMI Baking.md"]
updated: 2026-09-30
---

# EC2 Bootstrapping (User Data) y AMI baking

## Definición

**Bootstrapping** es que un sistema se configure solo al arrancar. En [[EC2]] se hace con el **user data**: un bloque de datos (normalmente un script) que le pasás a la instancia al lanzarla y que el sistema operativo **ejecuta como root**. Así la instancia llega sola a un estado configurado, sin depender de una AMI que ya traiga todo listo.

El contrapunto es el **AMI baking** ([[golden-ami]]): hacer el trabajo **antes** y guardarlo en una AMI. Las dos técnicas atacan el mismo problema, el **boot time to service time**.

## Cómo aplica en AWS

### User data

| Dato | Valor |
|---|---|
| Endpoint | `http://169.254.169.254/latest/user-data` (el mismo [[ec2-instance-metadata\|IMDS]] que `.../meta-data/`) |
| Cuándo se ejecuta | **Una sola vez, en el primer lanzamiento**. No se repite en reboot ni aunque lo modifiques |
| Tamaño máximo | **16 KB** (lo pesado se descarga de [[S3]] desde el script) |
| Modificarlo | Solo con la instancia **stopped** |
| Quién lo ejecuta | Un proceso del SO, como **root**. En las AMIs modernas, [[cloud-init]] |
| Formato en Linux | Script con shebang (`#!/bin/bash`) |

- **EC2 no valida ni interpreta** el user data: lo entrega tal cual. Si el script borra el boot volume, lo borra igual.
- **No es seguro:** cualquier proceso que pueda leer el metadata lo lee **en texto plano**. Los secretos van en [[SSMParameterStore|Parameter Store (SecureString)]] o Secrets Manager, y el script los lee al arrancar con el [[instance-profile|instance role]].

### Arquitectura

```
AMI ──launch──► instancia + EBS (block device mapping de la AMI)
                    │
EC2 ── user data ──►│  el SO lee 169.254.169.254/latest/user-data
                    │  y lo ejecuta (cloud-init, como root)
                    ▼
               running ── lista para el servicio (si el script anduvo)
```

> **Si el script falla, la instancia igual queda `running` y pasa los status checks.** EC2 solo entrega el user data: no sabe si el script funcionó. El resultado típico es una instancia accesible pero **mal configurada**.

### Boot time to service time

**Boot time to service time** es lo que tarda una instancia desde el lanzamiento hasta poder atender clientes:

- **Aprovisionamiento** de AWS: con una AMI de AWS, minutos.
- **Post-launch time**: instalar y configurar después de lanzar. A mano son minutos u horas, como la [[06.16 Demostración - Instalación manual de WordPress|instalación manual de WordPress]].

## Patrones comunes

| Enfoque | Post-launch time | Flexibilidad | Cuándo |
|---|---|---|---|
| **Bootstrapping** (user data) | Medio: el trabajo se hace en cada lanzamiento | **Alta**: cambiás el script, no la AMI | Configuración que varía por entorno o por instancia |
| **AMI baking** ([[golden-ami]]) | **Mínimo** | Baja: cualquier cambio obliga a hornear otra AMI | Arranque rápido y determinista (por ejemplo, autoscaling) |
| **Combinado (óptimo)** | Bajo | Alta | Bake de la parte lenta y bootstrapping de la configuración final |

Ejemplo del combinado: si un proceso es **90% instalación y 10% configuración**, el 90% se hornea en la AMI y el 10% se resuelve con user data.

## Preguntas de examen frecuentes

- *"Cambié el user data y reinicié, pero no pasó nada"* → el user data **solo corre en el primer launch**.
- *"La instancia está `running` y pasa 2/2 checks, pero la app no responde"* → el user data probablemente **falló**. EC2 no lo valida.
- *"Pasar la contraseña de la DB por user data"* → **no**: se lee en texto plano desde el IMDS. Usá Parameter Store SecureString o Secrets Manager, más un instance role.
- *"Minimizar el tiempo hasta que la instancia esté en servicio"* → **AMI baking**. *"…sin perder flexibilidad de configuración"* → **baking + bootstrapping**.
- *"Script de user data de más de 16 KB"* → que el user data descargue el script de S3.

## Demos del curso

- [WordPress installation — Part 1 (con user data)](https://learn.cantrill.io/courses/1101194/lectures/27895409)
- [WordPress installation — Part 2](https://learn.cantrill.io/courses/1101194/lectures/29447330), para ver después de [[CloudFormation]]

> 📖 Lectura profunda: [[09.01 Bootstrapping EC2 con User Data]] · [[09.02 Boot Time to Service Time y AMI Baking]]

## Ver también

[[EC2]] · [[golden-ami]] · [[ec2-instance-metadata]] · [[SSMParameterStore]] · [[dva-deployment]]
