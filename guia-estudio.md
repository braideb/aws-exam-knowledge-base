# Guía de Estudio — AWS Knowledge Base

> El LLM mantiene este archivo actualizado en cada ingest (ver `CLAUDE.md` → sección `guia-estudio.md`). Propone el **orden de aprendizaje** del material ya ingestado en `wiki/` — no está organizado por examen ni por peso de dominio (eso vive en `index.md`), sino por prerequisitos conceptuales: qué conviene entender antes de qué.

**Última actualización:** 2026-10-03 *(ingest del módulo 10 — Databases (SQL): **bloque nuevo, el 6 — Bases de datos relacionales**, ubicado **inmediatamente después de "Almacenamiento y cifrado"**. Necesita [[EC2]] y [[EBS]] (bloque 4: RDS es un servidor con su EBS), [[VPC]] y security groups (bloque 3), [[KMS]] (bloque 5: el cifrado de RDS) y [[SSMParameterStore]] (bloque 5: Secrets Manager se entiende por contraste). Nada de observabilidad ni de containers lo necesita, así que no hace falta llevarlo más atrás. **Cambio de orden:** [[parameter-store-vs-secrets-manager]] **se mueve del Bloque 5 al 6**, justo después de [[SecretsManager]], porque la comparación recién se entiende entera con las dos páginas leídas. Observabilidad, Containers e IaC pasan a ser los bloques **7, 8 y 9**. En el historial de abajo los números de bloque son los de **cada fecha**. Anterior, 2026-09-30: ingest del módulo 09 — Advanced EC2: **no hay bloque nuevo**, porque el módulo se reparte según sus prerequisitos. En el **Bloque 4** entran [[placement-groups]] (después de [[ec2-instance-types]], porque necesita enhanced networking y la noción de AZ) y [[ec2-bootstrapping]] (después de [[ec2-instance-metadata]], porque el user data sale del mismo endpoint y el baking supone AMI). En el **Bloque 5** entran [[SSMParameterStore]] y [[parameter-store-vs-secrets-manager]], **después de [[KMS]]**, porque SecureString se apoya en KMS. Los instance roles se leen en el Bloque 2 ([[IAM]]) y el CloudWatch Agent en el Bloque 6 ([[CloudWatchLogs]]). Anterior, 2026-09-28: ingest del módulo 08 — Containers, ECS y ECR: **bloque nuevo, el 7 — Containers**, que se ubica **después de observabilidad y antes de IaC**. Va después del 6 y no pegado a cómputo (bloque 4) porque ECS reusa casi todo lo anterior: [[virtualization]] y [[EC2]] (bloque 4), ENIs y security groups (bloque 3), roles (bloque 2), y [[CloudWatchLogs]] + el daemon de [[XRay]] como [[sidecar]] (bloque 6). IaC pasa a ser el bloque 8. Anterior, 2026-09-24: ingest del módulo 07 — Monitoring and logging: [[XRay]] entra al Bloque 6 **entre Logs y CloudTrail**, cerrando las tres patas de la observabilidad —métricas, logs, trazas— antes de pasar a auditoría; [[vpc-flow-logs]] no se mueve del Bloque 3, porque se entiende por los firewalls, pero su lectura profunda suma 07.08. Anterior, 2026-09-22: ingest de VPC 05.09–05.13 y del módulo 06 — EC2. Dos cambios de orden, no solo agregados: **[[storage-types]] se adelanta al Bloque 4**, porque es prerequisito tanto de EBS como de S3 y hasta ahora el bloque de S3 lo daba por sabido; y el **Bloque 4 deja de ser una sola página** para convertirse en el más largo de la guía, con cómputo y almacenamiento en bloque juntos. El Bloque 3 suma las cuatro páginas nuevas de VPC)*
**Páginas cubiertas:** 62 (58 de estudio + 4 mapas de dominio DVA-C02) + 117 términos de glosario (transversales) + 3 cheat sheets de repaso

---

## Cómo leer esta guía

- Cada bloque agrupa temas que se apoyan entre sí.
- Dentro de un bloque, el orden importa: lo de arriba es prerequisito de lo de abajo.
- Los bloques van de fundamentos a especialización, pero un tema puede volver a aparecer referenciado más adelante si otro bloque lo necesita como base.
- Un tema se estudia una sola vez acá, aunque aplique a varios exámenes (DVA-C02, SAA-C03, DOP-C02).
- Cada bloque cierra con **📖 Lectura profunda**: las secciones de las notas del curso (segmentadas, con navegación propia) que traen las analogías, demos y detalle completo — las páginas wiki son el condensado de examen. Las demos prácticas están indexadas en [[demos]]; el mapa completo de las notas está en [[00 Índice general]].
- El **glosario** (`wiki/glossary/`) **no forma parte de la secuencia**: es material de consulta al paso. Cuando una página use un término que no te suena, seguí el link y volvé. El catálogo completo está en [[index]]. Si preferís, al empezar el Bloque 1 leé de una sola vez los términos de *resiliencia* y *modelos de servicio* — son el vocabulario que usan todas las páginas posteriores.

---

## Bloque 1 — Fundamentos de AWS

*El mapa mental sobre el que se apoya todo lo demás: qué es una region, quién es responsable de qué, y el vocabulario de resiliencia.*

1. [[global-infrastructure]] — regions, AZs, edge locations, zonas de red, niveles de resiliencia. *Primero porque todo servicio se describe en estos términos.*
2. [[shared-responsibility-model]] — qué configura AWS y qué configurás vos. *Encuadra cada servicio que veas después.*
3. [[aws-account]] — la cuenta como contenedor, root user, MFA. *El punto de partida práctico de cualquier trabajo en AWS.*
4. [[ha-ft-dr]] — HA vs FT vs DR, RPO/RTO. *Cierra el bloque: vocabulario de resiliencia que reaparece en S3 replication, multi-AZ, etc.*

> 📖 Lectura profunda: [[01.01 Servicios públicos vs. privados]] · [[01.02 AWS Global Infrastructure]] · [[01.03 Availability Zones (AZ)]] · [[01.12 Modelo de responsabilidad compartida (Shared Responsibility Model)]] · [[01.13 HA vs. FT vs. DR]] · [[02.01 Cuenta de AWS (AWS Account)]] · [[02.02 Multi-Factor Authentication (MFA)]]

## Bloque 2 — Identidad y acceso

*Quién puede hacer qué. Prerequisito de TODOS los servicios: cada recurso que toques está detrás de una policy.*

1. [[IAM]] — users, groups, roles, STS, access keys, federación. *La entidad antes que la regla.*
2. [[iam-policy-evaluation]] — anatomía de policies, Deny>Allow>Default, cómo se combinan los tipos. *El motor de decisiones detrás de IAM.*
3. [[arn]] — el formato con que las policies nombran recursos. *Corto; leerlo pegado a policies.*
4. [[aws-cli]] — profiles, cadena de credenciales, acceso programático. *La práctica de todo lo anterior desde la terminal.*
5. [[Organizations]] — multi-cuenta, consolidated billing, SCPs. *Al final: los SCPs solo se entienden sabiendo evaluación de policies.*

> 📖 Lectura profunda: [[02.03 IAM — Conceptos básicos]] · [[02.04 IAM Access Keys]] · [[02.05 Demostración - AWS CLI y perfiles]] · [[03.01 IAM Identity Policies]] · [[03.02 IAM Users]] · [[03.03 ARN (Amazon Resource Name)]] · [[03.04 Restricciones y datos útiles de IAM]] · [[03.05 IAM Groups]] · [[03.06 IAM Roles]] · [[03.07 Cuándo usar IAM Roles - los cinco escenarios]] · [[03.08 Service-Linked Roles]] · [[03.09 Security Token Service (STS)]] · [[03.10 AWS Organizations]] · [[03.11 Service Control Policies (SCP)]] · [[09.03 EC2 Instance Roles e Instance Profiles]] *(instance roles en detalle; conviene leerla junto con el IMDS del bloque 4)*

## Bloque 3 — Redes (VPC) y DNS

*Dónde viven los recursos, cómo se mueve el tráfico, quién lo filtra y cómo se los encuentra por nombre. Prerequisito directo de EC2 (bloque 4).*

1. [[VPC]] — la red privada completa: default vs custom, CIDR, subnets, DNS/DHCP, route tables, IGW, SGs/NACLs, NAT. *La página hub del bloque; leerla entera primero, aunque los detalles de firewall y NAT se profundizan después.*
2. [[vpc-design]] — cómo elegir el rango y cortar las subnets (AZs × tiers). *Después de VPC porque usa sus reglas (`/16`–`/28`, 5 IPs reservadas); antes de firewalls porque define los tiers que ellos separan.*
3. [[security-groups-vs-nacls]] — stateful vs stateless, ephemeral ports, SG vs NACL. *Requiere entender subnets y routing; es la comparación de red que más cae en el examen.*
4. [[nat-gateway-vs-nat-instance]] — salida a internet desde private subnets, zonal vs regional, IPv6. *Al final de la parte de VPC: supone public/private subnets, route tables, Elastic IPs y security groups.*
5. [[vpc-flow-logs]] — metadatos del tráfico, ACCEPT/REJECT, destinos. *Va justo después de los firewalls: leer un REJECT y saber si cortó el SG o la NACL solo tiene sentido sabiendo que uno es stateful y el otro no.*
6. [[vpc-endpoints]] — acceso privado a servicios de AWS sin salir a internet. *Después de NAT, porque el argumento es "esto evita el rodeo por el NAT"; y después de routing, porque el gateway endpoint es una ruta.*
7. [[gateway-vs-interface-endpoint]] — la comparación de examen. *Inmediatamente después.*
8. [[vpc-peering]] — unir dos VPCs. *Cierra la parte de conectividad; reusa longest prefix match y el planeamiento de CIDRs de [[vpc-design]].*
9. [[lambda-in-vpc]] — cómo una función accede (o no) a la red. *Al final del bloque: junta subnets, SGs, NAT y endpoints en un solo escenario. Es además puro DVA.*
10. [[Route53]] — DNS, hosted zones, tipos de record, TTL. *Se apoya en el DNS de la VPC (Route 53 Resolver, private hosted zones); útil antes de hosting en S3.*
11. [[alias-vs-cname]] — la comparación que cae en el examen. *Inmediatamente después de Route53.*

> 📖 Lectura profunda: [[01.04 Default VPC (Virtual Private Cloud) — Basics]] · [[05.00 Virtual private cloud (VPC) Basics — Índice|Módulo 05 completo (05.01–05.13: sizing, custom VPC, subnets, routing e IGW, stateful vs stateless, NACLs, security groups, NAT Gateway, endpoints, flow logs, Lambda en VPC, peering, cheat sheet)]] · [[07.08 VPC Flow Logs]] · [[01.14 Route 53 (R53) — Fundamentos]] · [[01.15 DNS Record Types]]
> 🎯 Repaso: [[vpc-cheat-sheet]] al terminar el bloque.

## Bloque 4 — Cómputo y almacenamiento en bloque

*El bloque más largo de la guía (12 páginas). Arranca con el andamiaje (qué es virtualizar, qué tipos de almacenamiento existen) y recién después entra en EC2 y EBS, que se explican el uno al otro.*

1. [[virtualization]] — de la binary translation a SR-IOV y Nitro. *Corto y opcional si vas con el tiempo justo, pero explica varios "¿por qué?" que aparecen después (el cifrado de EBS sin costo de rendimiento, Enhanced Networking).*
2. [[storage-types]] — DAS vs NAS, block/file/object, efímero vs persistente, `IO block size × IOPS = throughput`. *Adelantado a propósito desde el bloque de S3: es prerequisito de EBS **y** de S3, y sin la fórmula no se entienden los tipos de volumen.*
3. [[EC2]] — el hub: arquitectura y hosts, ciclo de vida, estados, ENI/IPs/DNS, Elastic IP, AMI, status checks. *Requiere VPC y security groups (bloque 3) e IAM roles (bloque 2).*
4. [[ec2-instance-types]] — las cinco categorías y el esquema de nombres. *Después de EC2 porque es una decisión sobre una instancia que ya sabés qué es.*
5. [[placement-groups]] — cluster vs spread vs partition. *Después de instance types: el cluster exige tipos compatibles con [[enhanced-networking]], y los límites (7 por AZ) suponen saber qué es una AZ y un rack.*
6. [[EBS]] — volúmenes, snapshots, lazy restore, FSR, cifrado. *Acá se cobra lo de [[storage-types]] y lo de [[KMS]]; si no diste KMS todavía, volvé a esta sección después del bloque 5.*
7. [[ebs-volume-types]] — gp2/gp3/io1/io2/st1/sc1. *La comparación que más cae de almacenamiento.*
8. [[instance-store-vs-ebs]] — efímero vs persistente y la escalera de IOPS. *Cierra el almacenamiento: solo tiene sentido con los dos anteriores leídos.*
9. [[ec2-instance-metadata]] — IMDS, credenciales del role, IMDSv1 vs IMDSv2. *Necesita [[IAM]] (bloque 2) para entender qué se está robando en el ataque.*
10. [[ec2-bootstrapping]] — user data, boot time to service time, bootstrapping vs AMI baking. *Justo después del IMDS: el user data sale del mismo endpoint, y el baking supone la AMI de [[EC2]] y el [[golden-ami]].*
11. [[ec2-purchase-options]] — On-Demand, Spot, Reserved, Savings Plans, Dedicated, Capacity Reservations. *Al final: decidir cómo pagar supone saber qué estás pagando.*
12. [[horizontal-vs-vertical-scaling]] — el trade-off y las sesiones off-host. *Cierra el bloque y es la bisagra hacia arquitectura de aplicaciones.*

> 📖 Lectura profunda: [[01.05 Elastic Compute Cloud (EC2) — Basics]] · [[01.06 Amazon Machine Image (AMI)]] · [[01.07 Conectarse a EC2]] · [[06.00 Elastic Compute Cloud (EC2) — Índice|Módulo 06 completo (06.01–06.24: virtualización, arquitectura, instance types, storage, EBS y sus tipos, instance store, snapshots, cifrado, ENI/IPs/DNS, Elastic IP, AMI, purchase options, status checks, scaling, IMDS)]] · [[09.01 Bootstrapping EC2 con User Data]] · [[09.02 Boot Time to Service Time y AMI Baking]] · [[09.08 Placement Groups — Overview]] · [[09.09 Cluster Placement Groups]] · [[09.10 Spread Placement Groups]] · [[09.11 Partition Placement Groups]] · [[09.12 Enhanced Networking (SR-IOV, ENA, EFA)]] · [[09.13 EBS Optimized]]
> 🎯 Repaso: [[ec2-cheat-sheet]] al terminar el bloque.

## Bloque 5 — Almacenamiento y cifrado

*S3 primero como servicio, después sus dimensiones económicas y de seguridad. Da por sabido [[storage-types]], que se lee en el bloque 4.*

1. [[S3]] — el servicio completo: seguridad, versioning, performance, replication, presigned URLs, CORS, events, object lock. *Requiere policies (bloque 2).*
2. [[s3-storage-classes]] — clases de almacenamiento + lifecycle. *La dimensión económica de S3.*
3. [[KMS]] — claves, DEKs, envelope encryption, key policies. *Antes del cifrado de S3, porque SSE-KMS se apoya en esto.*
4. [[s3-encryption]] — client-side vs SSE-C/S3/KMS + bucket keys. *La síntesis: une S3 con KMS.*
5. [[SSMParameterStore]] — configuración y secretos: String / StringList / SecureString, jerarquías, tiers. *Después de KMS, porque SecureString es KMS + doble permiso. Abre el hilo de "dónde van los secretos" que empieza en [[ec2-bootstrapping]]; lo cierra [[SecretsManager]] en el bloque 6.*
6. [[CloudFront]] — CDN, OAC, contenido privado. *Cierra el bloque: la capa de entrega delante de S3.*

> 📖 Lectura profunda: [[01.08 S3 Buckets — Basics]] · [[01.09 S3 — Patterns y Anti-Patterns]] · [[04.00 S3 — Índice|Módulo 04 completo (04.01–04.17: security, versioning, performance, KMS, encryption, storage classes, replication, presigned URLs, CORS, object lock)]] · [[09.04 SSM Parameter Store]] · [[09.05 Demostración - Parameter Store]]

## Bloque 6 — Bases de datos relacionales

*De "instalo MySQL en la instancia" a una base gestionada, con HA, réplicas, backups, cifrado y credenciales que rotan solas. Se apoya en EC2/EBS (bloque 4), VPC y security groups (bloque 3), y KMS y Parameter Store (bloque 5).*

1. [[databases-on-ec2]] — la base self-managed: cuándo se justifica y por qué casi nunca. *Primero, porque es el punto de partida del curso y explica qué te ahorra RDS.*
2. [[RDS]] — el hub: DB instance en la VPC, DB subnet group, Multi-AZ, read replicas, backups, cifrado, IAM DB auth, RDS Proxy. *Usa [[EBS]] (cada instancia tiene el suyo) y [[KMS]].*
3. [[rds-automated-backups-vs-snapshots]] — retención, PITR vs punto fijo, restore = endpoint nuevo. *Inmediatamente después de RDS: es la parte de backups ampliada y la base del razonamiento RPO/RTO.*
4. [[Aurora]] — cluster con storage compartido, 15 replicas, endpoints, backtrack, clone, Serverless y Global Database. *Después de RDS, porque se explica por contraste con él.*
5. [[rds-vs-aurora]] — la comparación de arquitectura. *Inmediatamente después de Aurora.*
6. [[rds-ha-options]] — Multi-AZ instance vs cluster vs read replicas vs Aurora vs Global Database. *Recién acá, porque cruza todo lo anterior. Es la comparación que más cae del bloque.*
7. [[SecretsManager]] — secretos con rotación (Lambda o *managed*) e integración con RDS. *Al final: la rotación tiene sentido una vez que sabés qué credencial de RDS se rota.*
8. [[parameter-store-vs-secrets-manager]] — la comparación de examen (rotación). *Movida desde el bloque 5: con las dos páginas leídas se entiende entera.*

> 📖 Lectura profunda: [[10.00 Databases (SQL) — Índice|Módulo 10 completo (10.01–10.17: bases en EC2, RDS architecture y costos, Multi-AZ instance y cluster, backups y restore, read replicas, seguridad y TDE, IAM DB auth, Aurora, Serverless, Global Database, Secrets Manager, + 3 demos)]]
> 🎯 Repaso: [[databases-cheat-sheet]] al terminar el bloque. Para el razonamiento RPO/RTO de las bases, volvé a [[ha-ft-dr]] (bloque 1).

## Bloque 7 — Observabilidad y auditoría

*Ver qué pasa (métricas, logs y trazas), auditar quién hizo qué y reaccionar (eventos).*

1. [[CloudWatch]] — arquitectura, namespaces, metrics, dimensions, resolution/retención, alarms. *La base conceptual del bloque.*
2. [[CloudWatchLogs]] — log groups/streams, CloudWatch Agent en EC2, metric filters, export a S3, subscriptions. *Extiende CloudWatch a logs; las subscriptions reusan Lambda/Kinesis como destinos, alcanza con saber que existen.*
3. [[XRay]] — distributed tracing, segments, service map, annotations. *Tercera pata de la observabilidad: tiene sentido después de métricas y logs, porque resuelve lo que ellos no pueden (seguir una request individual sin explotar la cardinalidad). Muy DVA.*
4. [[CloudTrail]] — auditoría de API, trails, global service events. *Después de Logs porque sus eventos suelen mandarse ahí.*
5. [[EventBridge]] — reaccionar a eventos casi en tiempo real. *Cierra el ciclo observar → reaccionar; contrasta con el delay de CloudTrail.*
6. [[observability-costs]] — qué es gratis, qué se cobra y los 3 motores de gasto. *Va último del bloque: solo tiene sentido sabiendo qué son métricas custom, data events y dimensions. Cierra con la regla que ordena todo: el gobierno (IAM/Organizations/SCPs) es gratis, lo que se paga es observar.*

> 📖 Lectura profunda: [[01.11 CloudWatch — Basics]] · [[03.12 CloudWatch Logs]] · [[07.00 Monitoring and logging — Índice|Módulo 07 completo (07.01–07.08: arquitectura de CloudWatch, datos, resolution/retention, alarms, arquitectura de Logs, subscriptions y agregación, X-Ray, flow logs)]] · [[09.06 Logging en EC2 con CloudWatch Agent]] · [[09.07 Demostración - Logging y métricas con CloudWatch Agent]] · [[03.13 CloudTrail]] · [[03.14 Precios]]

## Bloque 8 — Containers

*Del "una VM por app" al "un proceso aislado por app": qué es un container, cómo lo orquesta AWS y dónde se guardan las images. Pesa en DVA por los roles de ECS y las estrategias de deployment.*

1. [[containers]] — VM vs container, images como layers read-only + R/W layer, Dockerfile, registry. *Primero, porque todo lo que sigue da por sabido qué es una image. Se apoya en [[virtualization]] (bloque 4).*
2. [[ECS]] — cluster, task definition, service, los tres roles, scaling en dos capas, rolling vs blue/green. *El corazón del bloque; necesita IAM roles (bloque 2), ENIs y SGs (bloque 3) y CloudWatch Logs / X-Ray (bloque 7).*
3. [[ecs-ec2-vs-fargate]] — los dos modos de cluster, networking (`awsvpc` vs `bridge`) y costos. *Inmediatamente después de ECS: es la decisión de examen del bloque, y usa [[ec2-purchase-options]].*
4. [[ECR]] — el registry: repositories, tags, scanning, repository policies, push. *Después de ECS porque su gotcha principal (quién hace el pull) solo se entiende sabiendo qué es el task execution role.*
5. [[EKS]] — Kubernetes 101 y Kubernetes managed. *Al final: se entiende por contraste con ECS (IRSA ↔ task role, Fargate profiles ↔ Fargate). Para DVA pesa menos que ECS.*

> 📖 Lectura profunda: [[08.00 Containers, ECS y ECR — Índice|Módulo 08 completo (08.01–08.10: containers, images y registry, demo de Docker en EC2, ECS concepts, cluster types, EC2 vs ECS vs Fargate, demo de Fargate, ECR, Kubernetes 101, EKS)]]

## Bloque 9 — Infraestructura como código

1. [[CloudFormation]] — templates y secciones. *Va último por ahora: declara recursos de todos los bloques anteriores. Página inicial, se ampliará con el curso.*

> 📖 Lectura profunda: [[01.10 CloudFormation — Basics]]

---

## Sugerencia de ruta

- **Si arrancás de cero**: bloques en orden, 1 → 9.
- **Si ya viste el curso** (este material sale de tus notas): usá los bloques como checklist de repaso y saltá directo a las páginas de comparación ([[s3-storage-classes]], [[s3-encryption]], [[alias-vs-cname]], [[security-groups-vs-nacls]], [[nat-gateway-vs-nat-instance]], [[gateway-vs-interface-endpoint]], [[ebs-volume-types]], [[instance-store-vs-ebs]], [[ec2-purchase-options]], [[horizontal-vs-vertical-scaling]], [[placement-groups]], [[parameter-store-vs-secrets-manager]], [[ecs-ec2-vs-fargate]], [[rds-automated-backups-vs-snapshots]], [[rds-vs-aurora]], [[rds-ha-options]]) + las secciones "Gotchas" de cada página, que es donde vive el jugo de examen.
- **Última pasada antes del examen**: los tres cheat sheets, [[vpc-cheat-sheet]], [[ec2-cheat-sheet]] y [[databases-cheat-sheet]]. **No están en la secuencia** — no enseñan nada nuevo, condensan lo ya leído en tablas de "escenario → respuesta" y listas de números. Leerlos antes de estudiar el tema no sirve.
- **Práctica activa**: al terminar un bloque, pedí un **quiz** ("quiz bloque 2") para repasar en activo lo que acabás de leer.

## Mapas de dominio (transversales, no secuenciales)

Las páginas de dominio de DVA-C02 no forman parte de la secuencia de estudio: son **mapas** que cruzan estos bloques con el temario oficial del examen y marcan los huecos pendientes de ingest. Consultalas para saber *cuánto del examen* cubre lo que ya estudiaste:

- [[dva-development]] (32%) · [[dva-security]] (26%) · [[dva-deployment]] (24%) · [[dva-troubleshooting]] (18%)
