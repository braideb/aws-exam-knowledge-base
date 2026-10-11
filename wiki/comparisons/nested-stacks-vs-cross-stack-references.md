---
title: Nested Stacks vs Cross-Stack References
category: comparison
tags: [cloudformation, iac, nested-stacks, cross-stack, stacksets]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/12 CloudFormation/12.12 Nested Stacks.md", "raw/notas curso mejorado/12 CloudFormation/12.13 Cross-Stack References.md"]
updated: 2026-10-10
---

# Nested Stacks vs Cross-Stack References

Las dos resuelven el mismo dolor —una stack única topa con el límite de **500 recursos** y con el **aislamiento** (`!Ref` no cruza el borde de una stack)— pero apuntan a cosas distintas. La frase que ordena todo:

> **Una template no es una stack.** Nested stacks = reutilizás el **código** (la template). Cross-stack references = reutilizás la **stack** (sus recursos reales).

| | **Nested Stacks** | **Cross-Stack References** |
|---|---|---|
| Qué reutilizás | La **template** (como una función/módulo de Lego) | La **stack** ya desplegada (sus recursos reales) |
| Cada uso… | Crea **recursos nuevos** (10 root stacks = 10 VPCs) | **Consume** el recurso compartido (1 VPC para todas) |
| Mecanismo | Una **root/parent stack** crea nested stacks; les pasa **parameters** y recibe **outputs** | La stack dueña hace `Export` de un output; la otra lo trae con **`!ImportValue`** |
| Ciclo de vida | **Acoplado**: todo se crea/actualiza/borra junto | **Desacoplado**: VPC de vida larga, apps de vida corta |
| Alcance | — | **Solo misma región y misma cuenta** |
| Cuándo | Recursos que **comparten ciclo de vida**; empaquetar un producto repetible | Arquitecturas **orientadas a servicios**; un componente compartido consumido por muchos |

## Cuándo usar X vs Y

- **Nested stacks** — querés una pieza **repetible**: la misma template de VPC sirve para cientos de entornos de clientes, y cada uso levanta su propia infraestructura aislada. Reutilizás el **plano**.
- **Cross-stack references** — querés **consumir** algo que ya existe: una `shared-vpc` (mismos NAT gateways y subnets) que muchas apps de corta duración usan sin recrearla, porque tienen **ciclos de vida distintos**. Reutilizás el **edificio**.

> Para repartir la **misma** infraestructura a **muchas cuentas/regiones**, no es ninguna de estas dos: son los **StackSets** (ver [[wiki/services/CloudFormation]]).

## Trampa típica del examen

- *"Reutilizar una template para desplegar muchos entornos idénticos"* → **nested stacks** (reutilizás código, cada uno con recursos nuevos).
- *"Varias stacks que comparten una misma VPC"* → **cross-stack references** (`Export` + `!ImportValue`).
- **Cross-stack references NO cruzan región ni cuenta.** Si el escenario es multi-cuenta/multi-región → StackSets.
- El nombre de un `Export` debe ser **único por región y cuenta**.
- `!Ref` cruza el borde de la stack **solo** en el caso root→nested (y eso acopla sus ciclos de vida); para todo lo demás, `!ImportValue`.

## Ver también

[[wiki/services/CloudFormation]] · [[cloudformation-template]] · [[cloudformation-intrinsic-functions]] · [[Organizations]]
