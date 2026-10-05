---
title: Active Directory (AD)
category: glossary
tags: [identidad, windows, fsx, directory-service]
exam: [SAA-C03, DVA-C02]
sources: ["raw/notas curso mejorado/11 Advanced Storage/11.04 FSx for Windows File Server.md", "raw/notas curso mejorado/11 Advanced Storage/11.08 Comparativa - EFS vs FSx for Windows vs FSx for Lustre.md"]
updated: 2026-10-05
---

# Active Directory (AD)

> **En una línea:** el **directorio de identidades de Microsoft**: donde una empresa Windows guarda usuarios, grupos y equipos, y contra el que se autentican.

## Definición

En AWS hay dos formas de tenerlo. **Managed**: AWS Directory Service te da un AD como servicio. **Self-managed**: el AD lo corrés vos, en EC2 o **on-premises**, que es el que la mayoría de las empresas ya tiene. Los servicios de Windows de AWS lo usan como **user store**.

## Dónde aparece

- [[FSx]]: FSx for Windows **exige** un AD, managed o self-managed (incluido el on-premises)
- [[efs-vs-fsx]]: "Active Directory" como palabra disparadora de FSx for Windows
- También en: [[storage-types]] · [[federation]] · [[posix]] · [[EFS]] · [[IAM]] · [[VPC]]

## Dato de examen

- **"Active Directory"** en un escenario de file shares → **FSx for Windows**.
- FSx for Windows puede integrarse **directamente con el AD on-premises**; no hace falta migrarlo a Directory Service.

## Ver también

[[smb]] · [[federation]]
