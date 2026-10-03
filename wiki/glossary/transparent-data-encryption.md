---
title: TDE (Transparent Data Encryption)
category: glossary
tags: [rds, cifrado, oracle, sql-server, cloudhsm]
exam: [DVA-C02, SAA-C03]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.11 RDS Security (Encryption y TDE).md", "raw/doc oficial/Amazon Relational Database Service - AWS Prescriptive Guidance.md", "raw/doc oficial/Sharing encrypted snapshots for Amazon RDS - Amazon Relational Database Service.md"]
updated: 2026-10-03
---

# TDE (Transparent Data Encryption)

> **En una línea:** cifrado en reposo hecho **por el propio motor** de base de datos, no por el host ni por el storage.

## Definición

En [[RDS]] el cifrado por defecto es KMS + EBS: lo hace el **host** y el motor escribe en claro. Con TDE (**Oracle y SQL Server**) el motor cifra antes de escribir a disco, así que hay que confiar menos en la capa de abajo. **RDS Oracle + CloudHSM** lleva TDE al extremo: las claves están en un HSM que gestionás vos y **AWS nunca las ve**. Se puede combinar con el cifrado de KMS, con algo de costo de rendimiento y dos juegos de claves para gestionar ¹.

¹ Complemento de la doc oficial.

## Dónde aparece

- [[RDS]] — seguridad en reposo
- [[databases-cheat-sheet]]

## Dato de examen

- *"Cifrado gestionado por el motor"* / *"cadena de confianza sin AWS"* (regulatorio) → **TDE**, con CloudHSM en Oracle.
- Los snapshots cifrados con TDE **no se pueden compartir** con otras cuentas ¹.

## Ver también

[[encryption-at-rest]] · [[envelope-encryption]] · [[data-encryption-key]]
