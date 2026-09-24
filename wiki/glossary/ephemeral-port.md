---
title: Ephemeral port
category: glossary
tags: [networking, tcp, nacl, firewall]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.05 Stateful vs Stateless Firewalls.md", "raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/05.06 Network Access Control Lists (NACLs).md"]
updated: 2026-09-24
---

# Ephemeral port

> **En una línea:** el puerto **temporal y aleatorio** que el cliente usa como origen de una conexión, y al que vuelve la respuesta.

## Definición

Al abrir una conexión TCP, el cliente elige un puerto de origen al azar (el rango depende del SO; en AWS se usa **`1024–65535`**) y apunta a un **well-known port** del servidor (`443`, `22`…). La respuesta del servidor va **de vuelta a ese ephemeral port**, no al puerto de la aplicación.

## Dónde aparece

- [[security-groups-vs-nacls]] — por qué las NACLs necesitan una regla para la respuesta
- [[VPC]] — reglas de NACL; el NAT Gateway usa los puertos 1024–65535
- [[vpc-flow-logs]] — el puerto alto que aparece en `srcport`/`dstport` y delata a la NACL
- También en: [[dva-troubleshooting]] · [[vpc-cheat-sheet]] · [[stateful-firewall]] · [[stateless-firewall]]

## Dato de examen

- En una **NACL** ([[stateless-firewall|stateless]]), permitir el `443` inbound **no alcanza**: hace falta una regla **outbound `1024–65535`** para la respuesta. Olvidarla = "la conexión se abre pero el cliente nunca recibe respuesta".
- En un **SG** ([[stateful-firewall|stateful]]) no hace falta pensar en ephemeral ports.

## Ver también

[[stateless-firewall]] · [[stateful-firewall]]
