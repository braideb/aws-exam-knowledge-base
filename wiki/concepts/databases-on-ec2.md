---
title: Bases de datos en EC2 (self-managed)
category: concept
tags: [ec2, databases, rds, self-managed, arquitectura]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.01 Databases on EC2.md", "raw/notas curso mejorado/10 Databases (SQL)/10.02 Demostración - Splitting WordPress (app y DB).md", "raw/doc oficial/What is Amazon Relational Database Service (Amazon RDS) - Amazon Relational Database Service.md"]
updated: 2026-10-03
---

# Bases de datos en EC2 (self-managed)

## Definición

Instalar y operar vos mismo el motor de base de datos (MySQL, MariaDB, etc.) dentro de una instancia [[EC2]], en vez de usar un servicio gestionado como [[RDS]] o [[Aurora]]. Es el extremo opuesto a un [[dbaas|DBaaS]]. **Casi siempre se puede**, pero en AWS se considera **mala práctica salvo que haya una razón concreta**: te hacés cargo de todo lo que RDS te daría resuelto.

## Cómo aplica en AWS

### Las dos arquitecturas típicas

| Arquitectura | Cómo es | Consecuencia |
|---|---|---|
| **Monolito** | App + web server + DB en **una** instancia, en una AZ (el WordPress del curso) | Un solo punto de falla; todo cae junto |
| **Split** | DB en una instancia, app en otra (misma AZ o AZs distintas) | **Dependencia de red** entre ambas. Entre AZs la transferencia **se cobra**; dentro de la misma AZ por IP privada es **gratis** |

### Quién gestiona qué (doc oficial ¹)

| Tarea | EC2 | RDS |
|---|---|---|
| Hardware, energía, red física | AWS | AWS |
| SO: instalación y parches | **Vos** | AWS |
| Motor: instalación y parches | **Vos** | AWS |
| Backups, HA, escalado | **Vos** | AWS |
| Optimización de la app y queries | Vos | Vos |

## Patrones comunes

**Cuándo sí está justificado:**
- Necesitás **acceso al SO** o **root** para tuning. Conviene cuestionarlo, porque RDS expone muchos de esos parámetros sin root.
- Un **motor o versión que RDS no ofrece** (el caso más legítimo).
- Una combinación **SO + motor** o un esquema de **replicación** particular que AWS no ofrece.
- Lo exige la organización.

**Por qué no:**
- **Overhead de administración**: parchear SO y motor, mantener compatibilidad de versiones.
- **Backups y DR a mano**: EC2 y su [[EBS]] son [[az-resilient|AZ-resilient]]. Si cae la AZ, cae la base, y los snapshots y la copia a [[S3]] los tenés que hacer vos.
- **Sin features gestionadas**: Multi-AZ, read replicas, PITR, cifrado integrado, RDS Proxy.
- **Escalado**: una instancia está prendida o apagada (sin [[serverless]]) y fija un **costo base mínimo**.
- **Rendimiento**: un motor de estantería no aprovecha las optimizaciones de RDS/Aurora.

**Migración monolito → split → RDS (demos del curso):**
```bash
mysqldump -u root -p a4lwordpress > a4lwordpress.sql                 # dump local
mysql -h <IP privada MariaDB | CNAME de RDS> -u a4lwordpress -p a4lwordpress < a4lwordpress.sql
# wp-config.php: DB_HOST 'localhost' → IP privada de MariaDB → endpoint de RDS
```

## Preguntas de examen frecuentes

- *"La app necesita un motor/versión que RDS no soporta"* o *"requiere acceso al SO"* → **base en EC2**. Hay preguntas donde **esta es la respuesta correcta**.
- *"Pocos recursos de personal, HA y consultas complejas con joins"* → **RDS Multi-AZ**, no MySQL instalado en EC2 (sin HA de serie) ni DynamoDB (sin joins) ¹.
- *"Reducir el overhead de parches y backups de la base"* → migrar a **RDS**.
- *"La app en EC2 se conecta a la base nueva"* → por el **endpoint DNS** de RDS, nunca por IP.
- RDS Proxy **no** funciona con bases self-managed en EC2 ¹.

¹ Complemento de la doc oficial (clippings en `sources`), no de las notas del curso.

## Demos del curso

- [Splitting WordPress monolith into app and DB](https://learn.cantrill.io/courses/1101194/lectures/27894839)
- [Migrating EC2 DB into RDS — PART 1](https://learn.cantrill.io/courses/1101194/lectures/27894843) · [PART 2](https://learn.cantrill.io/courses/1101194/lectures/27894844)

> 📖 Lectura profunda: [[10.01 Databases on EC2]] · [[10.02 Demostración - Splitting WordPress (app y DB)]] · [[10.05 Demostración - Migrar DB de EC2 a RDS]] · [[06.16 Demostración - Instalación manual de WordPress]]
