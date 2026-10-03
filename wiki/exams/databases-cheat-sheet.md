---
title: Bases de datos (SQL) — Cheat sheet de examen
category: exam
tags: [rds, aurora, secrets-manager, repaso, cheat-sheet, dva-c02]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.00 Databases (SQL) — Índice.md"]
updated: 2026-10-03
---

# Bases de datos (SQL) — Cheat sheet de examen

Destilado del módulo 10 (Databases SQL), con los números ya corregidos contra la doc oficial. No enseña nada nuevo: es para la última pasada. Cada punto linkea a su página.

## Los puntos, uno por línea

- **Base en EC2:** casi siempre mala práctica; se justifica con **motor/versión no soportada** o **acceso al SO**. → [[databases-on-ec2]]
- **RDS:** "database server as a service". Una **DB instance** con varias bases, **en una VPC**, **EBS propio**, endpoint **DNS**. → [[RDS]]
- **[[db-subnet-group|DB subnet group]]:** las subnets donde RDS ubica primary y standby (en AZs distintas). → [[RDS]]
- **Multi-AZ instance:** 1 standby **síncrona** que **no se lee**. Failover por **CNAME**, **60–120 s**. Solo misma región. → [[rds-ha-options]]
- **Multi-AZ cluster:** 1 writer + **2 readers legibles**, solo MySQL/PostgreSQL, commit con ≥1 reader, failover **~35 s**. **No es Aurora.** → [[rds-ha-options]]
- **Read replicas:** **asíncronas**, endpoint propio, **promote manual**, cross-region posible. **Sirven ante fallas, no ante corrupción.** → [[rds-ha-options]]
- **Automated backups:** diario + **transaction logs cada 5 min** → **PITR**, retención **0–35 días**. → [[rds-automated-backups-vs-snapshots]]
- **Snapshots manuales:** **no caducan**, sobreviven a la instancia, se comparten y se copian. → [[rds-automated-backups-vs-snapshots]]
- **Restore:** siempre **instancia nueva = endpoint nuevo**, y lento (mal RTO). → [[RDS]]
- **Cifrado RDS:** KMS + EBS, **solo al crear**, no se quita, la key no cambia. Réplica = mismo estado de cifrado. Sin cifrar → **snapshot → copia cifrada → restore**. → [[RDS]]
- **TDE:** cifrado **dentro del motor**, Oracle y SQL Server. Oracle + **CloudHSM** = AWS no ve las claves. → [[transparent-data-encryption]]
- **IAM DB auth:** token de **15 min** con `generate-db-auth-token`, permiso `rds-db:connect`. **Autenticación, no autorización.** MySQL/MariaDB/PostgreSQL. → [[RDS]]
- **RDS Proxy:** pool de conexiones para **Lambda**/mucha concurrencia; **failover más rápido sin esperar al DNS**; credenciales en Secrets Manager. → [[RDS]]
- **Aurora:** cluster con **storage compartido** (6 copias, 3 AZs), hasta **15 replicas legibles** que son target de failover (~30 s). → [[Aurora]]
- **Endpoints de Aurora:** cluster (escribe) · reader (balancea lecturas) · custom (subconjunto) · instance (diagnóstico). → [[Aurora]]
- **Backtrack:** rebobina el **mismo cluster in-place**, solo **Aurora MySQL**, ventana máx. **72 h**. **Fast clone:** [[copy-on-write]]. → [[Aurora]]
- **Aurora Serverless v2:** [[acu|ACUs]] entre mín. y máx., pasos de **0,5**, **auto-pause** con mínimo 0. → [[Aurora]]
- **Global Database:** 1 región primaria + **10 secundarias** read-only, lag **< 1 s**. **Switchover** (sin pérdida) vs **failover** (pierde segundos). → [[Aurora]]
- **Data API:** SQL por **HTTPS** sin conexión persistente ni Lambda en VPC; credenciales desde Secrets Manager. → [[Aurora]]
- **Secrets Manager:** secretos cifrados con KMS + **rotación** (Lambda o gestionada) + integración nativa con RDS. De pago. → [[SecretsManager]]

## Los números de memoria

| Dato | Valor |
|---|---|
| Failover Multi-AZ instance | **60–120 s** |
| Failover Multi-AZ cluster | **~35 s** |
| Failover Aurora (con replica) | **~30 s**; sin replica, recrea la primary (~10 min) |
| Retención de automated backups | **0–35 días** |
| Transaction logs a S3 | **cada 5 min** (RPO ≈ 5 min) |
| Read replicas RDS por instancia | **15** (el curso dice 5); cross-region garantizadas: **5** |
| Readers en Multi-AZ cluster | **2** |
| Aurora replicas | **15** (16 en una región secundaria de Global Database) |
| Copias del storage de Aurora | **6**, en **3 AZs** |
| Tamaño máx. de Aurora | **256 TiB** (128 en el curso / versiones viejas) |
| Ventana de backtrack | **72 h** |
| Clones copy-on-write por origen | **15** |
| ACU | ~**2 GiB**; v2 de **0,5** a **256**; mínimo **0** con auto-pause |
| Auto-pause | idle de **300 s a 86.400 s**; resume ~**15 s** |
| Global Database | **1 + 10** regiones, lag **< 1 s** |
| Token de IAM DB auth | **15 min** |
| Rotación del master gestionada por RDS | cada **7 días** (default) |
| Failover tiers de Aurora | **0–15** (gana el menor; si empatan, la más grande) |
| Puertos | MySQL/MariaDB **3306** · PostgreSQL **5432** · SQL Server **1433** · Oracle **1521** |

## Escenario → respuesta

| Escenario | Respuesta |
|---|---|
| HA ante la caída de una AZ, motor Oracle | **RDS Multi-AZ** (instance) |
| La primary está saturada de lecturas | **Read replicas** / Multi-AZ cluster / Aurora + reader endpoint |
| Lambda abre miles de conexiones y la base se cae | **RDS Proxy** |
| La app tarda minutos en volver tras un failover | Cache de **DNS** → TTL corto o **RDS Proxy** |
| Recuperar el estado de hace 2 horas tras un borrado | **PITR** (automated backups) |
| Guardar una copia por 7 años | **Snapshot manual** |
| Cifrar una base creada sin cifrar | Snapshot → **copia cifrada** → restore |
| Compartir un snapshot cifrado con otra cuenta | Recifrarlo con una **customer managed key** y compartir la key |
| Que el código no tenga la contraseña de la DB | **IAM DB auth** o **Secrets Manager** |
| Rotar la contraseña de la DB automáticamente | **Secrets Manager** |
| Configuración barata y sin rotación | **Parameter Store** |
| DR multi-región, RPO de segundos, failover gestionado | **Aurora Global Database** |
| Mover la región primaria sin perder datos | Global Database **switchover** |
| Deshacer un `DELETE` sin `WHERE` sin cambiar endpoints | **Backtrack** (Aurora MySQL) |
| Copia de prod para test en minutos | **Aurora fast clone** |
| Carga SQL impredecible, con horas ociosas | **Aurora Serverless v2** |
| Motor/versión que RDS no tiene, o acceso al SO | **Base en EC2** |
| Lambda → Aurora sin VPC ni pool de conexiones | **RDS Data API** |

## Los errores que más se repiten

1. Creer que la **standby de Multi-AZ instance** sirve lecturas.
2. Confundir el **Multi-AZ cluster** de RDS con **Aurora**.
3. Usar una **read replica** para recuperarse de **corrupción** (la replica también está corrupta).
4. Esperar que un **restore** conserve el endpoint (es una instancia nueva).
5. Creer que los **automated backups** retenidos duran para siempre (caducan a los 35 días como máximo).
6. Pensar que **IAM DB auth** da permisos dentro de la base (solo autentica).
7. Elegir **Parameter Store** cuando el enunciado pide **rotación**.
8. Esperar **failover automático cross-region** con read replicas de RDS (es promote manual; el gestionado es Aurora Global Database).
9. Buscar el bucket de S3 de los backups de RDS (es de AWS, no se ve).
10. Repetir el **5** de read replicas del curso: la doc actual dice **15**.

## Ver también

[[RDS]] · [[Aurora]] · [[SecretsManager]] · [[rds-ha-options]] · [[rds-vs-aurora]] · [[rds-automated-backups-vs-snapshots]] · [[databases-on-ec2]] · [[parameter-store-vs-secrets-manager]] · [[ha-ft-dr]]
