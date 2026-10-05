---
title: SMB (Server Message Block)
category: glossary
tags: [storage, file-storage, fsx, windows, protocolo]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/11 Advanced Storage/11.04 FSx for Windows File Server.md", "raw/notas curso mejorado/11 Advanced Storage/11.05 FSx — Key Features and Benefits.md", "raw/notas curso mejorado/11 Advanced Storage/11.08 Comparativa - EFS vs FSx for Windows vs FSx for Lustre.md"]
updated: 2026-10-05
---

# SMB (Server Message Block)

> **En una línea:** el protocolo de **file shares de Windows**, el que está detrás de las rutas `\\servidor\share`.

## Definición

Es el equivalente Windows de NFS: los clientes acceden a carpetas compartidas por red, con los permisos y la autenticación de Windows (normalmente contra [[active-directory|Active Directory]]). En AWS lo implementa [[FSx]] for Windows File Server.

## Dónde aparece

- [[FSx]]: el protocolo de FSx for Windows
- [[efs-vs-fsx]]: SMB (FSx for Windows) vs NFS (EFS)
- También en: [[storage-types]] · [[EFS]] · [[ec2-cheat-sheet]]

## Dato de examen

- Ver **"SMB"** en una pregunta de file systems compartidos casi siempre significa **FSx for Windows**. EFS **no** habla SMB.

## Ver también

[[nfs]] · [[active-directory]]
