---
title: cloud-init
category: glossary
tags: [ec2, bootstrapping, user-data, linux]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/09 Advanced EC2/09.01 Bootstrapping EC2 con User Data.md"]
updated: 2026-10-03
---

# cloud-init

> **En una línea:** el software dentro del SO de la instancia que **lee el user data y lo ejecuta** al primer arranque.

## Definición

[[EC2]] no ejecuta el user data: solo lo deja disponible en `169.254.169.254/latest/user-data`. Quien lo busca y lo corre **como root** es un proceso del sistema operativo, que en las AMIs Linux modernas es **cloud-init**. Por eso el script tiene que ser válido para ese SO: EC2 no lo valida.

## Dónde aparece

- [[ec2-bootstrapping]]: el paso que ejecuta el user data.
- También en: [[EC2]]

## Dato de examen

- El user data corre **una sola vez**, en el primer launch. Si falla, la instancia igual queda `running` y pasa los status checks.

## Ver también

[[golden-ami]] · [[instance-profile]]
