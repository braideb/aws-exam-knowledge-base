---
title: Escalado horizontal vs vertical
category: comparison
tags: [escalado, ec2, arquitectura, sesiones, load-balancer]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.23 Horizontal vs Vertical Scaling.md"]
updated: 2026-09-22
---

# Escalado horizontal vs vertical

Dos formas de que un sistema crezca cuando sube la carga. **Vertical** = el mismo componente, más grande. **Horizontal** = más componentes iguales.

| | **Vertical** | **Horizontal** |
|---|---|---|
| Cómo escala | Instancia **más grande** (`t3.large` → `t3.xlarge`) | **Más** instancias |
| ¿Downtime al escalar? | **Sí** — el resize requiere reiniciar | **No** |
| ¿Límite superior? | **Sí** — el tamaño máximo de instancia | No, en teoría |
| ¿Hay que tocar la aplicación? | **No** | **Sí**: tiene que ser [[stateless]] |
| Granularidad | Baja: saltos que duplican | Alta: de a una instancia |
| Costo | Sobreprecio de los tamaños grandes (no crece lineal) | Menor: instancias chicas |
| Requiere | — | Un **Load Balancer** que reparta el tráfico |

## Cuándo usar cada uno

### Vertical

Su ventaja es que **no requiere cambios de código**: si una app corre en una instancia, corre igual en una más grande. Es la salida rápida para software que no fue diseñado para distribuirse — monolitos, bases de datos tradicionales, licencias atadas a un servidor.

Sus tres costos: **downtime** en cada resize, un **techo físico** que existe y del que no se pasa, y un precio que **crece más que proporcionalmente** hacia los tamaños grandes de cada familia ([[ec2-instance-types]]).

### Horizontal

Resuelve los tres problemas anteriores: no hay interrupción al agregar o quitar instancias, no hay techo, sale más barato y es más granular (si tenés 5 instancias chicas y necesitás un poco más, agregás una más).

El precio es **arquitectónico**: la aplicación tiene que ser [[stateless|stateless]].

## El problema de las sesiones

Es el punto central, y el que más cae.

Cuando un usuario inicia sesión, el estado de esa interacción es la **sesión**. Si cada instancia la guarda localmente, mover al cliente de una instancia a otra lo deslogea — porque la instancia nueva no sabe nada de él.

La solución son las **off-host sessions**: la sesión se guarda en un lugar **externo y centralizado** (DynamoDB, ElastiCache, una base de datos) y todas las instancias la leen de ahí. Recién entonces los servidores son **intercambiables** y el escalado horizontal funciona de verdad.

```
Sesión en el host (stateful)          Sesión fuera del host (stateless)

  Cliente → LB → Instancia A            Cliente → LB → Instancia A ┐
                 [sesión]                              Instancia B ┼→ [sesión]
                                                       Instancia C ┘
  Cambia de instancia → deslogueado     Cambia de instancia → todo sigue igual
```

## Trampa típica del examen

- *"Los usuarios se deslogean cuando agregamos instancias"* → la respuesta es **mover las sesiones fuera del host**, no cambiar el balanceador, ni activar sticky sessions como solución de fondo, ni volver a escalado vertical.
- *"Escalar sin downtime"* → **horizontal**. El vertical siempre implica reiniciar.
- *"No podemos modificar la aplicación"* → **vertical**, asumiendo el downtime y el techo.
- *"Ya estamos en el tamaño de instancia más grande"* → se acabó el vertical; la única salida es **horizontal**.
- El escalado horizontal **necesita un load balancer**: si una opción propone agregar instancias sin nada que reparta el tráfico, está incompleta.

## Ver también

[[EC2]] · [[stateless]] · [[ec2-instance-types]] · [[ha-ft-dr]] · [[idempotency]]
