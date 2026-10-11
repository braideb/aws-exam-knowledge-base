---
title: CloudFormation — cfn-init, cfn-signal y señales
category: concept
tags: [cloudformation, iac, cfn-init, cfn-signal, cfn-hup, bootstrapping, ec2]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/12 CloudFormation/12.08 CreationPolicy, WaitConditions y cfn-signal.md", "raw/notas curso mejorado/12 CloudFormation/12.09 cfn-init (CloudFormation Init).md", "raw/notas curso mejorado/12 CloudFormation/12.10 cfn-hup.md", "raw/notas curso mejorado/12 CloudFormation/12.11 Demostración - cfn-signal, cfn-init y cfn-hup.md"]
updated: 2026-10-10
---

# CloudFormation — cfn-init, cfn-signal y señales

## Definición

El problema que resuelve este cluster: **`CREATE_COMPLETE` a veces miente**. CFN marca un recurso como completo apenas AWS lo crea, pero si además corrés [[ec2-bootstrapping|bootstrapping]] (user data que instala software), la instancia llega a `CREATE_COMPLETE` **mucho antes** de terminar —y sigue en `CREATE_COMPLETE` aunque el bootstrapping **falle**—. Cuatro herramientas cierran la brecha: **CreationPolicy / WaitCondition** (pausar), **cfn-signal** (avisar), **cfn-init** (configurar por desired state) y **cfn-hup** (reaplicar en updates).

## Cómo aplica en AWS

### Señales: CreationPolicy y WaitCondition

Configurás CFN para que **retenga** el recurso hasta recibir *N* **señales de éxito** dentro de un **Timeout** (formato H:M:S, **máximo 12 h**). La señal la manda **cfn-signal** (del paquete `aws-cfn-bootstrap`) desde la instancia.

| Resultado | Efecto |
|---|---|
| Llegan todas las señales de éxito a tiempo | `CREATE_COMPLETE` (de verdad) |
| Llega una señal de fallo | `CREATE_FAILED` inmediato |
| Vence el timeout | `CREATE_FAILED` |

Dos formas de señalizar:

| | **CreationPolicy** | **WaitCondition** |
|---|---|---|
| Qué es | Atributo **dentro** del recurso | Recurso lógico **independiente** |
| Pensado para | EC2 / Auto Scaling groups | Integrar con un sistema **externo** |
| Extra | — | Usa un **WaitHandle** que genera una [[presigned-url\|URL prefirmada]]; puede recibir un JSON leíble con `!GetAtt` |

> **Para el examen:** AWS **recomienda CreationPolicy** para la mayoría de los casos (vive pegada al recurso, más fácil de administrar). Comando típico al final del user data: `/opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource ASG --region ${AWS::Region}`.

### cfn-init — desired state

**`AWS::CloudFormation::Init`** guarda la configuración como **metadata** pegada al recurso, y el helper **`cfn-init`** (preinstalado en las AMIs) la aplica. La diferencia conceptual con el user data es el corazón del tema:

- **User data = procedural (el CÓMO):** lista de comandos, paso a paso. Es función de EC2, no de CFN.
- **cfn-init = desired state (el QUÉ):** declarás el estado final. Es nativo de CFN, **multiplataforma** e **[[idempotency|idempotente]]**: si Apache ya está, no lo reinstala.

cfn-init procesa las secciones de cada *config key* **en orden fijo**: `packages` → `groups` → `users` → `sources` → `files` → `commands` → `services`. Dentro de **commands**, el orden es **alfabético** por nombre.

### cfn-hup — reaplicar en updates

cfn-init corre **una sola vez**. Un `UpdateStack` que cambie la metadata **no** la reaplica solo. **`cfn-hup`** es un daemon (lo instalás vos en el bootstrap) que vigila la metadata del recurso y, al detectar un cambio, vuelve a correr cfn-init. Con cfn-init + cfn-hup, un `UpdateStack` también actualiza la config del SO de una instancia **ya corriendo**.

## Patrones comunes

```
user data (mínimo)
  └─ cfn-init  ──► aplica la metadata AWS::CloudFormation::Init (desired state)
  └─ cfn-signal ─► avisa a la CreationPolicy si terminó OK/mal
cfn-hup (daemon) ─► ante UpdateStack que toca la metadata, re-dispara cfn-init
```

Logs donde mirar cuando un bootstrap falla: `/var/log/cloud-init-output.log` y `/var/log/cfn-init-cmd.log`.

## Preguntas de examen frecuentes

- *"La stack dice CREATE_COMPLETE pero la app no está lista / falló el bootstrap"* → faltó **cfn-signal** + **CreationPolicy**.
- *"Configuración declarativa, idempotente y multiplataforma dentro de CFN"* → **cfn-init**, no user data procedural.
- *"Que un UpdateStack actualice la config de instancias ya corriendo"* → **cfn-hup** (instalado en el bootstrap inicial).
- *"Máximo tiempo de espera de señales"* → **12 horas**.
- *"Señalizar desde un sistema externo con datos de vuelta"* → **WaitCondition** (+ WaitHandle); si es EC2/ASG, **CreationPolicy**.

> 📖 Lectura profunda: [[12.08 CreationPolicy, WaitConditions y cfn-signal]] · [[12.09 cfn-init (CloudFormation Init)]] · [[12.10 cfn-hup]] · [[12.11 Demostración - cfn-signal, cfn-init y cfn-hup]]

## Ver también

[[wiki/services/CloudFormation]] · [[ec2-bootstrapping]] · [[desired-state-configuration]] · [[idempotency]] · [[EC2]]
