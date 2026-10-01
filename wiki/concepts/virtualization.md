---
title: Virtualización
category: concept
tags: [ec2, virtualizacion, hypervisor, nitro, performance]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.01 Virtualization 101.md", "raw/notas curso mejorado/09 Advanced EC2/09.12 Enhanced Networking (SR-IOV, ENA, EFA).md"]
updated: 2026-09-30
---

# Virtualización

## Definición

**Virtualizar** es correr más de un sistema operativo sobre una misma pieza de hardware físico. Es lo que vende [[EC2]] como servicio, y entender cómo evolucionó explica varias características del producto.

El problema de origen: el **kernel** de un OS corre en **privileged mode** y espera ser el único dueño del hardware. Las aplicaciones corren en **user mode** y le piden cosas al OS mediante **system calls**. Si ponés varios OS sobre el mismo hardware, todos pelean por ese acceso privilegiado y el resultado son cuelgues.

## Cómo aplica en AWS

### Los cuatro escalones

| Enfoque | Cómo resuelve el privilegio | Costo |
|---|---|---|
| **Emulated virtualization** | El [[hypervisor]] intercepta cada llamada privilegiada y la traduce al vuelo (**binary translation**). El guest cree que está sobre hardware real | **Lentísimo** — hasta la mitad de la velocidad nativa |
| **Para-virtualization** | Se **modifica el código del guest OS** para que llame al hypervisor (**hypercalls**) en vez de al hardware, + drivers paravirtualizados | Rápido, pero el OS debe estar adaptado a ese hypervisor |
| **Hardware-assisted** | La **CPU** incorpora instrucciones de virtualización: atrapa (*trap*) la instrucción privilegiada y la redirige al hypervisor | Muy poca pérdida. Queda el cuello de botella del **I/O** |
| **SR-IOV** | El **dispositivo** (no solo la CPU) se vuelve consciente: una tarjeta se presenta como varias mini-tarjetas reales, una por guest | El hypervisor ya no traduce I/O |

### El cuello de botella del I/O

Lo que más importa en una VM suele ser el **I/O**: red y disco. Las VMs creen tener hardware físico, pero son **dispositivos lógicos** que terminan compartiendo una única pieza física del host. Esa capa de software intermedia golpea el rendimiento y consume CPU del host justo en las cargas más transaccionales.

**SR-IOV** (*Single Root I/O Virtualization*) lo resuelve dejando que cada guest hable **directo** con su porción de hardware. En EC2 se llama **[[enhanced-networking|Enhanced Networking]]** y aporta más ancho de banda, **latencia más baja y más constante** bajo carga, y menos CPU del host consumida en I/O.

### Nitro

AWS tiene su propia plataforma: **[[nitro|Nitro]]**, que mueve red, almacenamiento y seguridad a hardware dedicado. Es la razón técnica de varias cosas que sí se preguntan:

- El **cifrado de [[EBS]] no tiene impacto de rendimiento**: lo hace el host, no el sistema operativo ni la aplicación.
- Las generaciones más nuevas de instancias soportan límites de I/O mucho más altos ([[ebs-optimized]]).
- Las instancias rinden prácticamente como si corrieran sobre metal desnudo.

## Patrones comunes

```
Sin virtualizar   → 1 OS, todo el hardware, acceso privilegiado directo
Emulated          → hypervisor traduce TODO         (lento)
Para-virtualized  → el guest colabora (hypercalls)  (rápido, OS modificado)
Hardware-assisted → la CPU colabora                 (rápido, I/O pendiente)
SR-IOV / Nitro    → el hardware colabora            (casi nativo)
```

## Preguntas de examen frecuentes

Este tema **no se pregunta directamente** en el DVA-C02: es el andamiaje conceptual. Lo que sí aparece, y se explica desde acá:

- *"¿El cifrado de EBS degrada el rendimiento?"* → **No**, ocurre en el host (Nitro), fuera del OS.
- *"Necesito latencia de red baja y predecible entre instancias"* → **Enhanced Networking** (+ cluster [[placement-groups|placement group]]). Adaptadores: **ENA** (hasta 100 Gbps), Intel 82599 VF (viejo, 10 Gbps) y **EFA** para HPC/MPI.
- Por qué una instancia [[iaas|IaaS]] te deja controlar el SO pero no el hypervisor: es la línea del [[shared-responsibility-model]].
- La `n` en un type como `R5dn` indica justamente networking mejorado ([[ec2-instance-types]]).

## Ver también

[[EC2]] · [[containers]] (el paso siguiente: aislar apps sin un OS por app) · [[hypervisor]] · [[enhanced-networking]] · [[nitro]] · [[iaas]] · [[shared-responsibility-model]]
