---
title: POSIX
category: glossary
tags: [storage, file-storage, linux, permisos, efs, lustre]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/11 Advanced Storage/11.01 EFS — Architecture.md", "raw/notas curso mejorado/11 Advanced Storage/11.06 FSx for Lustre — Concepts.md", "raw/notas curso mejorado/11 Advanced Storage/11.08 Comparativa - EFS vs FSx for Windows vs FSx for Lustre.md"]
updated: 2026-10-05
---

# POSIX

> **En una línea:** el estándar de interoperabilidad de los sistemas tipo Unix; en storage, quiere decir **permisos de Linux** (usuario, grupo, otros: `rwx`).

## Definición

Un file system con **permisos POSIX** guarda para cada archivo un dueño (**UID**), un grupo (**GID**) y los bits de lectura/escritura/ejecución, así que cualquier distribución de Linux lo entiende sin traducción. Es lo opuesto al modelo de permisos de Windows (ACLs + [[active-directory|Active Directory]]).

## Dónde aparece

- [[EFS]]: los file systems usan permisos POSIX; los access points imponen una identidad POSIX (UID/GID)
- [[FSx]]: FSx for Lustre soporta permisos POSIX (palabra clave de examen)
- [[efs-vs-fsx]]: POSIX (EFS, Lustre) vs Windows (FSx for Windows)
- También en: [[dva-security]]

## Dato de examen

- **POSIX → Linux**: EFS o FSx for Lustre, nunca FSx for Windows. Para desempatar entre los dos, mirá el caso de uso (file share general vs [[hpc|HPC]]).

## Ver también

[[nfs]] · [[smb]]
