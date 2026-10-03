---
title: Shared Responsibility Model
category: concept
tags: [seguridad, fundamentos, iaas, paas, saas]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.12 Modelo de responsabilidad compartida (Shared Responsibility Model).md"]
updated: 2026-10-03
---

# Shared Responsibility Model

## Definición

Modelo que divide la responsabilidad de seguridad entre AWS y el cliente:

- **AWS** → seguridad **OF** the cloud (*de* la nube): hardware, infraestructura global, compute/storage/database/networking, y el software que corre esos servicios.
- **Cliente** → seguridad **IN** the cloud (*en* la nube): sus datos, plataforma/aplicaciones/[[IAM]], configuración de SO/red/firewall, [[encryption-at-rest|cifrado client-side y server-side]], protección del tráfico.

> Mnemotecnia: **AWS asegura la nube; vos asegurás lo que ponés dentro de la nube.**

## Cómo aplica en AWS

La línea de corte depende del modelo de servicio. Con **[[iaas|IaaS]]** ([[EC2]]) el corte está en el [[hypervisor]]: de ahí para abajo AWS, de ahí para arriba vos.

| Capa | On-Prem | DC Hosted | [[iaas\|IaaS]] | [[paas\|PaaS]] | [[saas\|SaaS]] |
|---|---|---|---|---|---|
| Interface / Application / Data | Vos | Vos | Vos | Vos | AWS |
| Runtime / Container / O.S. | Vos | Vos | Vos | AWS | AWS |
| Hypervisor / Servers / Infra | Vos | Vos | AWS | AWS | AWS |
| Facilities | Vos | **Proveedor** | AWS | AWS | AWS |

*DC Hosted* = tus servidores en el datacenter de un tercero (colocation): el proveedor solo pone el edificio, la energía y la refrigeración.

![[Pasted image 20260627160129.png]]

![[Pasted image 20260627160619.png]]

## Ejemplos concretos para no confundirse

- Parchear el SO de una EC2 → **tuyo** (IaaS).
- Parchear el motor de [[RDS]] → AWS aplica, vos elegís la ventana.
- Parchear Lambda/DynamoDB → **AWS** (managed/[[serverless]]).
- Configurar un Security Group → **siempre tuyo**.
- Seguridad física del datacenter → **siempre AWS**.

## Preguntas de examen frecuentes

El examen da un escenario y pregunta "¿de quién es la responsabilidad?". Método: identificar el **modelo de servicio** (IaaS/PaaS/SaaS) y ubicar la capa en la tabla.
