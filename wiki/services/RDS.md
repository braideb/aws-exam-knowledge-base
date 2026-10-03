---
title: Amazon RDS (Relational Database Service)
category: service
tags: [rds, databases, sql, multi-az, read-replicas, backups, rds-proxy, iam-auth, kms]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/10 Databases (SQL)/10.03 RDS — Architecture.md", "raw/notas curso mejorado/10 Databases (SQL)/10.04 RDS — Costos.md", "raw/notas curso mejorado/10 Databases (SQL)/10.05 Demostración - Migrar DB de EC2 a RDS.md", "raw/notas curso mejorado/10 Databases (SQL)/10.06 RDS Multi-AZ Instance.md", "raw/notas curso mejorado/10 Databases (SQL)/10.07 RDS Multi-AZ Cluster.md", "raw/notas curso mejorado/10 Databases (SQL)/10.08 RDS Backups y Restore.md", "raw/notas curso mejorado/10 Databases (SQL)/10.09 RDS Read Replicas.md", "raw/notas curso mejorado/10 Databases (SQL)/10.10 Demostración - Multi-AZ y snapshot restore con RDS.md", "raw/notas curso mejorado/10 Databases (SQL)/10.11 RDS Security (Encryption y TDE).md", "raw/notas curso mejorado/10 Databases (SQL)/10.12 RDS IAM Authentication.md", "raw/doc oficial/What is Amazon Relational Database Service (Amazon RDS) - Amazon Relational Database Service.md", "raw/doc oficial/Configuring and managing a Multi-AZ deployment for Amazon RDS - Amazon Relational Database Service.md", "raw/doc oficial/RDS Multi-AZ Deployment – Failover, Sync Replication & HA Guide.md", "raw/doc oficial/New Amazon RDS for MySQL & PostgreSQL Multi-AZ Deployment Option Improved Write Performance & Faster Failover.md", "raw/doc oficial/Creating a read replica in a different AWS Region - Amazon Relational Database Service.md", "raw/doc oficial/RDS Cross-Region Read Replicas – DR & Scaling.md", "raw/doc oficial/Restoring a DB instance to a specified time for Amazon RDS - Amazon Relational Database Service.md", "raw/doc oficial/Encrypting Amazon RDS resources - Amazon Relational Database Service.md", "raw/doc oficial/Amazon Relational Database Service - AWS Prescriptive Guidance.md", "raw/doc oficial/Sharing encrypted snapshots for Amazon RDS - Amazon Relational Database Service.md", "raw/doc oficial/IAM database authentication for MariaDB, MySQL, and PostgreSQL - Amazon Relational Database Service.md", "raw/doc oficial/Enabling and disabling IAM database authentication - Amazon Relational Database Service.md", "raw/doc oficial/Connect to an RDS PostgreSQL instance using IAM authentication.md", "raw/doc oficial/Connect to an RDS PostgreSQL instance using IAM authentication 1.md", "raw/doc oficial/Troubleshooting for IAM DB authentication - Amazon Relational Database Service.md", "raw/doc oficial/Amazon RDS Proxy - Amazon Relational Database Service.md", "raw/doc oficial/Using Amazon RDS Proxy with AWS Lambda.md", "raw/doc oficial/Automatically connecting a Lambda function and a DB instance - Amazon Relational Database Service.md", "raw/doc oficial/Amazon RDS and Aurora High Availability Guide - Multi-AZ, Read Replicas, RDS Proxy, and Global Database Failover.md", "raw/doc oficial/Password management with Amazon RDS and AWS Secrets Manager - Amazon Relational Database Service.md"]
updated: 2026-10-03
---

# Amazon RDS (Relational Database Service)

## ¿Qué es?

Base de datos relacional **gestionada**. Más que un [[dbaas|DBaaS]] es un **"database server as a service"**: te da una **DB instance** (un servidor de base de datos) que puede contener **varias bases de datos**. AWS gestiona hardware, SO, instalación, parcheo, backups y [[failover]]; vos no tenés acceso al SO ni SSH. Corre **dentro de una [[VPC]]**, no es un servicio público.

Motores: **MySQL, MariaDB, PostgreSQL, Oracle, SQL Server** y **Db2** ¹. **[[Aurora]] es un producto aparte**, aunque se administre desde la misma consola.

¹ Db2 no figura en las notas del curso; viene de la doc oficial.

## Casos de uso

- Cualquier carga **relacional / SQL** (joins, transacciones) sin administrar servidores. La doc lo recomienda como **opción por defecto** para bases relacionales.
- Escalar lecturas con **read replicas**; HA con **Multi-AZ**; DR cross-region con réplicas o backups copiados.
- Cuando el motor es **Oracle, SQL Server o Db2**, que Aurora no soporta.
- Lo que **no** cubre (acceso al SO, motor o versión no soportada) → base en EC2 ([[databases-on-ec2]]).

## Características clave

### Arquitectura

```
VPC (us-east-1)
├── DB subnet group "privado"  → subnets en AZ-A, AZ-B, AZ-C
│     ├── primary  (AZ-A)  ── EBS propio
│     └── standby  (AZ-B)  ── EBS propio      ← Multi-AZ: replicación síncrona
└── Backups / snapshots → S3 gestionado por AWS (no se ve en tu cuenta)
```

- **[[db-subnet-group|DB subnet group]]**: la lista de subnets donde RDS puede ubicar instancias. Con Multi-AZ, primary y standby quedan en **AZs distintas**. Best practice del curso: uno por implementación. En subnets públicas la base **puede** ser pública (mala práctica, pero existe).
- Acceso de red con un **security group** adjunto a la instancia. Puertos por defecto: **MySQL/MariaDB/Aurora MySQL 3306 · PostgreSQL 5432 · SQL Server 1433 · Oracle 1521**.
- **Endpoint = nombre DNS (CNAME)**, nunca una IP: las apps siempre se conectan al endpoint.
- **Storage**: cada instancia tiene **su propio [[EBS]]** (gp2/gp3 o **Provisioned IOPS**). El magnético está deprecado ¹.

### Costos

Se cobra por **asignación**, no por uso: tamaño/tipo de instancia (por hora, facturado por segundo) · Multi-AZ (más instancias) · storage por GB-mes (PIOPS más caro) · transferencia de datos · backups más allá del tamaño gratis (= al storage provisionado) · licencias de motores comerciales. La replicación primary → standby de Multi-AZ **no cobra transferencia** ¹.

### Alta disponibilidad: Multi-AZ

| | **Multi-AZ DB instance** | **Multi-AZ DB cluster** |
|---|---|---|
| Topología | 1 primary + **1 standby** | 1 writer + **2 readers** en 3 AZs |
| ¿La réplica sirve lecturas? | ❌ **No** (solo espera el failover) | ✅ Sí, por el **reader endpoint** |
| Replicación | **[[synchronous-replication\|Síncrona]]**, a nivel storage | Por **transaction logs**: commit cuando **≥1 reader** confirma (semisíncrona ¹) |
| Failover | Cambio de **DNS**: **60–120 s** | **~35 s** (+ aplicar logs pendientes) |
| Motores | Todos | Solo **MySQL y PostgreSQL** ¹ |
| Hardware | EBS | **Graviton + NVMe local** → EBS |

- Failover: lo dispara la caída de la AZ, la falla de la primary, un **cambio de tipo de instancia**, el **parcheo** (se hace primero en la standby) o un `reboot --force-failover`. **No** lo dispara la corrupción de datos ni la caída de la región.
- Los **backups se toman desde la standby** → sin pausa de I/O en la primary.
- Clientes que **cachean DNS** (la JVM) siguen apuntando a la IP vieja: TTL de cache ≤ 60 s ¹.
- Multi-AZ **no está en el free tier**. Multi-AZ instance es **solo dentro de una región**.

> Comparación completa con read replicas, Aurora y Global Database: [[rds-ha-options]].

### Read replicas

- Copias **de solo lectura**, con **[[asynchronous-replication|replicación asíncrona]]** (puede haber [[replication-lag]]). Misma región o **cross-region** (tráfico cifrado en tránsito).
- **Tienen su propio endpoint**: la app tiene que saber usarlas. **No hay failover automático**: se **promueven** a mano (`promote-read-replica`) y pasan a ser una instancia independiente.
- Pueden tener **sus propias réplicas** (cascada, más lag). Una read replica puede ser a su vez **Multi-AZ** ¹.
- **[[rpo|RPO]] casi nulo y [[rto|RTO]] bajo** para recuperarse de una **falla**. **No sirven ante corrupción**: la corrupción también se replica.

> ⚠️ Outdated: el curso dice **5 read replicas directas** por instancia. La doc actual dice **hasta 15** (in-region + cross-region combinadas) para MySQL, MariaDB, PostgreSQL, Oracle, SQL Server y Db2; lo que RDS **no garantiza** es más de **5 cross-region**.

Detalles cross-region de la doc ¹: si se borra la instancia de origen, la réplica cross-region **se promueve sola** en MySQL, MariaDB, Oracle y Db2, pero en **PostgreSQL** queda `terminated` y hay que promoverla a mano. Una réplica cifrada en otra región necesita una **KMS key de esa región**.

### Backups y restore

| | **Automated backups** | **Snapshots manuales** |
|---|---|---|
| Frecuencia | 1 snapshot/día en la **backup window** + **transaction logs cada 5 min** | Cuando los pedís |
| Retención | **0–35 días** (0 = deshabilitado); se borran solos | **Indefinida**: sobreviven a la instancia |
| Restore | **[[point-in-time-recovery\|Cualquier punto]]** de la ventana (RPO ≈ 5 min) | Al momento exacto del snapshot |

- Todos van a **S3 gestionado por AWS** → [[region-resilient|resilientes de región]]. **No los ves en la consola de S3.**
- Primer snapshot completo, después **incrementales**, y de **toda la instancia** (no de una base).
- **Restaurar crea una instancia nueva con un endpoint nuevo**: hay que reconfigurar la app. Es **lento** (mal [[rto|RTO]]). Además, la instancia restaurada carga los bloques desde S3 en segundo plano ([[lazy-restore]]) ¹.
- Al borrar la instancia, los automated backups retenidos **igual caducan**: para conservar más de 35 días, **snapshot final**.
- **Backups cross-region**: snapshots + transaction logs a otra región. **No es el default**; se configura.
- Snapshots: se **comparten** con otras cuentas y se **copian** a otras regiones. → [[rds-automated-backups-vs-snapshots]]

### Seguridad

- **En tránsito**: SSL/TLS en todos los motores; se puede **forzar** (por usuario, o con `rds.force_ssl=1` en el parameter group ¹).
- **[[encryption-at-rest|En reposo]]**: [[KMS]] + EBS, cifrado por el **host** (AES-256). Cifra storage, logs, snapshots, backups y réplicas con la **misma key**. [[transparent-data-encryption|TDE]] (cifrado dentro del motor) en **Oracle y SQL Server**; Oracle + **CloudHSM** para que AWS no vea las claves.
- Reglas de cifrado (curso + doc ¹):
  - Se elige **al crear**; no se activa ni se quita después, y la **KMS key no se cambia**.
  - **Read replica = mismo estado de cifrado** que la primary (misma key en la misma región).
  - Para cifrar una base existente: **snapshot → copiar con cifrado → restaurar**.
  - Un snapshot cifrado con la **AWS managed key no se puede compartir**: hay que recopiarlo con una **customer managed key** y compartir esa key. Los snapshots cifrados nunca son públicos, y los cifrados con TDE no se comparten.
  - Si RDS pierde acceso a la KMS key, la instancia pasa a un estado recuperable por **7 días**; después solo se recupera desde un backup ¹.

### IAM database authentication

- Usuario local en la DB mapeado a una identidad de IAM. La identidad llama a **`generate-db-auth-token`** (firmado con [[sigv4|SigV4]]) y obtiene un **token de 15 minutos** que reemplaza a la contraseña.
- Permiso: **`rds-db:connect`** sobre `arn:aws:rds-db:<region>:<account>:dbuser:<resource-id>/<usuario>`. En PostgreSQL además `GRANT rds_iam TO <usuario>` ¹.
- **Es autenticación, no autorización**: los permisos dentro de la base siguen siendo del usuario local.
- Solo **MariaDB, MySQL y PostgreSQL** ¹ (más Aurora MySQL/PostgreSQL). Exige SSL/TLS. Deshabilitado por defecto.
- Ideal con [[temporary-credentials|credenciales temporales]] de un instance role o una Lambda. Contra: límite de conexiones nuevas por segundo y consumo de memoria extra ¹ → para mucha concurrencia, **RDS Proxy**.

### RDS Proxy

- **[[connection-pooling|Pool de conexiones]] gestionado** delante de RDS/Aurora. Para **Lambda** y apps con muchas conexiones cortas que agotan `max_connections`.
- Hace el **failover más rápido**: retiene las conexiones de los clientes y las lleva a la nueva primary **sin esperar al DNS** (hasta −66 % ¹).
- Credenciales: los clientes pueden autenticarse con **IAM** y el proxy se conecta a la base con un secreto de **[[SecretsManager]]**. El código nunca ve la contraseña.
- Restricciones ¹: misma VPC que la base, **nunca público**, 20 proxies por cuenta, y en RDS solo se asocia al **writer** (no a una read replica).

### Credenciales del master en Secrets Manager

Con `--manage-master-user-password`, RDS **genera** la contraseña del master, la guarda en [[SecretsManager]] y la **rota cada 7 días** por defecto (*managed rotation*: sin Lambda propia). Si se borra la instancia, se borra el secreto ¹.

## Integración con otros servicios

- [[VPC]]: subnets (DB subnet group) y [[security-groups-vs-nacls|security groups]]; las apps llegan por IP privada.
- [[EBS]]: storage de cada instancia; los snapshots funcionan igual que los de EBS.
- [[S3]]: destino gestionado de backups y snapshots.
- [[KMS]]: cifrado en reposo; key por región para réplicas y copias.
- [[IAM]]: IAM DB auth, roles para la app, [[service-linked-role]] de RDS.
- [[SecretsManager]]: credenciales y rotación (integración nativa).
- [[lambda-in-vpc]]: una Lambda que lee una RDS privada tiene que estar en la VPC; con muchas invocaciones, RDS Proxy.
- [[CloudWatch]] y eventos de RDS (vía SNS) para enterarse de un failover.

## Gotchas y trampas del examen

- **Multi-AZ instance NO escala lecturas**: la standby no se lee. Escalar lecturas → **read replicas**, **Multi-AZ cluster** o [[Aurora]].
- **Síncrono = Multi-AZ · asíncrono = read replicas** (en RDS "clásico").
- El failover cambia el **CNAME** del endpoint, no la IP. Si la app no se reconecta → **cache de DNS**; la cura es RDS Proxy o un TTL corto. Durante el failover, los reintentos van con backoff, y las escrituras tienen que ser [[idempotency|idempotentes]] ¹.
- *"Recuperarse de un borrado accidental de hace 2 horas"* → **PITR** desde automated backups (instancia nueva). Una read replica **no** sirve: replicó el borrado.
- *"Conservar backups más de 35 días"* → **snapshots manuales** (o snapshot final al borrar).
- *"Cifrar una RDS que se creó sin cifrar"* → snapshot → **copia cifrada** → restore. No hay un "enable encryption".
- *"Compartir un snapshot cifrado con otra cuenta"* → tiene que estar cifrado con una **customer managed key** compartida, no con la AWS managed.
- *"Lambda agota las conexiones de la base"* → **RDS Proxy**.
- *"Que la app no guarde la contraseña de la base"* → IAM DB auth (token de 15 min) o Secrets Manager.
- *"Rotar la contraseña de RDS automáticamente"* → **Secrets Manager**, no Parameter Store ([[parameter-store-vs-secrets-manager]]).
- *"Necesito acceso al SO / un motor que RDS no tiene"* → base en EC2 ([[databases-on-ec2]]).
- Restaurar = **endpoint nuevo**; un failover Multi-AZ = **mismo endpoint**.

¹ Complemento de la doc oficial (clippings en `sources`), no de las notas del curso.

## Demos del curso

- [Splitting WordPress monolith into app and DB](https://learn.cantrill.io/courses/1101194/lectures/27894839): `mysqldump` + restore en una instancia MariaDB aparte. Ver [[databases-on-ec2]].
- [Migrating EC2 DB into RDS — PART 1](https://learn.cantrill.io/courses/1101194/lectures/27894843) · [PART 2](https://learn.cantrill.io/courses/1101194/lectures/27894844): dump desde MariaDB, restore contra el **endpoint (CNAME)** de RDS y cambio de `DB_HOST` en `wp-config.php`.
- [Multi-AZ & snapshot restore with RDS — PART 1](https://learn.cantrill.io/courses/1101194/lectures/27894848) · [PART 2](https://learn.cantrill.io/courses/1101194/lectures/27894849).

> 📖 Lectura profunda: [[10.03 RDS — Architecture]] · [[10.04 RDS — Costos]] · [[10.05 Demostración - Migrar DB de EC2 a RDS]] · [[10.06 RDS Multi-AZ Instance]] · [[10.07 RDS Multi-AZ Cluster]] · [[10.08 RDS Backups y Restore]] · [[10.09 RDS Read Replicas]] · [[10.10 Demostración - Multi-AZ y snapshot restore con RDS]] · [[10.11 RDS Security (Encryption y TDE)]] · [[10.12 RDS IAM Authentication]]
