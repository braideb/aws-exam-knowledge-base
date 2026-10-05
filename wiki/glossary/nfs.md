---
title: NFS (Network File System)
category: glossary
tags: [storage, file-storage, efs, linux, protocolo]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/11 Advanced Storage/11.01 EFS — Architecture.md", "raw/notas curso mejorado/11 Advanced Storage/11.05 FSx — Key Features and Benefits.md", "raw/notas curso mejorado/11 Advanced Storage/11.08 Comparativa - EFS vs FSx for Windows vs FSx for Lustre.md"]
updated: 2026-10-05
---

# NFS (Network File System)

> **En una línea:** el protocolo estándar de Linux/Unix para **montar un file system por red**, como si fuera una carpeta local.

## Definición

El cliente monta el file system remoto en una carpeta de su árbol (`/mnt/datos`, `/var/www/...`) y lo usa como si fuera local. Muchos clientes pueden montar el mismo file system a la vez. [[EFS]] implementa **NFS v4** y usa el **puerto TCP 2049**.

## Dónde aparece

- [[EFS]]: el protocolo del servicio; el SG del mount target tiene que permitir 2049
- [[efs-vs-fsx]]: NFS (EFS) vs SMB (FSx for Windows)
- También en: [[FSx]] · [[ec2-cheat-sheet]] · [[ec2-cheat-sheet]]

## Dato de examen

- **"NFS"** o **"shared Linux file system"** en el enunciado → **EFS**.
- EFS no monta → revisá que el security group del mount target acepte **2049** desde el SG del cliente.

## Ver también

[[smb]] · [[posix]]
