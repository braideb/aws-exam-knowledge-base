# Guía de Estudio — AWS Knowledge Base

> El LLM mantiene este archivo actualizado en cada ingest (ver `CLAUDE.md` → sección `guia-estudio.md`). Propone el **orden de aprendizaje** del material ya ingestado en `wiki/` — no está organizado por examen ni por peso de dominio (eso vive en `index.md`), sino por prerequisitos conceptuales: qué conviene entender antes de qué.

**Última actualización:** 2026-09-22 *(ingest de VPC 05.09–05.13 y del módulo 06 — EC2. Dos cambios de orden, no solo agregados: **[[storage-types]] se adelanta al Bloque 4**, porque es prerequisito tanto de EBS como de S3 y hasta ahora el bloque de S3 lo daba por sabido; y el **Bloque 4 deja de ser una sola página** para convertirse en el más largo de la guía, con cómputo y almacenamiento en bloque juntos. El Bloque 3 suma las cuatro páginas nuevas de VPC)*
**Páginas cubiertas:** 45 (41 de estudio + 4 mapas de dominio DVA-C02) + 83 términos de glosario (transversales) + 2 cheat sheets de repaso

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

> 📖 Lectura profunda: [[02.03 IAM — Conceptos básicos]] · [[02.04 IAM Access Keys]] · [[02.05 Demostración - AWS CLI y perfiles]] · [[03.01 IAM Identity Policies]] · [[03.02 IAM Users]] · [[03.03 ARN (Amazon Resource Name)]] · [[03.04 Restricciones y datos útiles de IAM]] · [[03.05 IAM Groups]] · [[03.06 IAM Roles]] · [[03.07 Cuándo usar IAM Roles - los cinco escenarios]] · [[03.08 Service-Linked Roles]] · [[03.09 Security Token Service (STS)]] · [[03.10 AWS Organizations]] · [[03.11 Service Control Policies (SCP)]]

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

> 📖 Lectura profunda: [[01.04 Default VPC (Virtual Private Cloud) — Basics]] · [[05.00 Virtual private cloud (VPC) Basics — Índice|Módulo 05 completo (05.01–05.13: sizing, custom VPC, subnets, routing e IGW, stateful vs stateless, NACLs, security groups, NAT Gateway, endpoints, flow logs, Lambda en VPC, peering, cheat sheet)]] · [[01.14 Route 53 (R53) — Fundamentos]] · [[01.15 DNS Record Types]]
> 🎯 Repaso: [[vpc-cheat-sheet]] al terminar el bloque.

## Bloque 4 — Cómputo y almacenamiento en bloque

*El bloque más largo de la guía. Arranca con el andamiaje (qué es virtualizar, qué tipos de almacenamiento existen) y recién después entra en EC2 y EBS, que se explican el uno al otro.*

1. [[virtualization]] — de la binary translation a SR-IOV y Nitro. *Corto y opcional si vas con el tiempo justo, pero explica varios "¿por qué?" que aparecen después (el cifrado de EBS sin costo de rendimiento, Enhanced Networking).*
2. [[storage-types]] — DAS vs NAS, block/file/object, efímero vs persistente, `IO block size × IOPS = throughput`. *Adelantado a propósito desde el bloque de S3: es prerequisito de EBS **y** de S3, y sin la fórmula no se entienden los tipos de volumen.*
3. [[EC2]] — el hub: arquitectura y hosts, ciclo de vida, estados, ENI/IPs/DNS, Elastic IP, AMI, status checks. *Requiere VPC y security groups (bloque 3) e IAM roles (bloque 2).*
4. [[ec2-instance-types]] — las cinco categorías y el esquema de nombres. *Después de EC2 porque es una decisión sobre una instancia que ya sabés qué es.*
5. [[EBS]] — volúmenes, snapshots, lazy restore, FSR, cifrado. *Acá se cobra lo de [[storage-types]] y lo de [[KMS]]; si no diste KMS todavía, volvé a esta sección después del bloque 5.*
6. [[ebs-volume-types]] — gp2/gp3/io1/io2/st1/sc1. *La comparación que más cae de almacenamiento.*
7. [[instance-store-vs-ebs]] — efímero vs persistente y la escalera de IOPS. *Cierra el almacenamiento: solo tiene sentido con los dos anteriores leídos.*
8. [[ec2-instance-metadata]] — IMDS, credenciales del role, IMDSv1 vs IMDSv2. *Necesita [[IAM]] (bloque 2) para entender qué se está robando en el ataque.*
9. [[ec2-purchase-options]] — On-Demand, Spot, Reserved, Savings Plans, Dedicated, Capacity Reservations. *Al final: decidir cómo pagar supone saber qué estás pagando.*
10. [[horizontal-vs-vertical-scaling]] — el trade-off y las sesiones off-host. *Cierra el bloque y es la bisagra hacia arquitectura de aplicaciones.*

> 📖 Lectura profunda: [[01.05 Elastic Compute Cloud (EC2) — Basics]] · [[01.06 Amazon Machine Image (AMI)]] · [[01.07 Conectarse a EC2]] · [[06.00 Elastic Compute Cloud (EC2) — Índice|Módulo 06 completo (06.01–06.24: virtualización, arquitectura, instance types, storage, EBS y sus tipos, instance store, snapshots, cifrado, ENI/IPs/DNS, Elastic IP, AMI, purchase options, status checks, scaling, IMDS)]]
> 🎯 Repaso: [[ec2-cheat-sheet]] al terminar el bloque.

## Bloque 5 — Almacenamiento y cifrado

*S3 primero como servicio, después sus dimensiones económicas y de seguridad. Da por sabido [[storage-types]], que se lee en el bloque 4.*

1. [[S3]] — el servicio completo: seguridad, versioning, performance, replication, presigned URLs, CORS, events, object lock. *Requiere policies (bloque 2).*
2. [[s3-storage-classes]] — clases de almacenamiento + lifecycle. *La dimensión económica de S3.*
3. [[KMS]] — claves, DEKs, envelope encryption, key policies. *Antes del cifrado de S3, porque SSE-KMS se apoya en esto.*
4. [[s3-encryption]] — client-side vs SSE-C/S3/KMS + bucket keys. *La síntesis: une S3 con KMS.*
5. [[CloudFront]] — CDN, OAC, contenido privado. *Cierra el bloque: la capa de entrega delante de S3.*

> 📖 Lectura profunda: [[01.08 S3 Buckets — Basics]] · [[01.09 S3 — Patterns y Anti-Patterns]] · [[04.00 S3 — Índice|Módulo 04 completo (04.01–04.17: security, versioning, performance, KMS, encryption, storage classes, replication, presigned URLs, CORS, object lock)]]

## Bloque 6 — Observabilidad y auditoría

*Ver qué pasa (métricas/logs), reaccionar (eventos) y auditar quién hizo qué.*

1. [[CloudWatch]] — namespaces, metrics, dimensions, alarms. *La base conceptual del bloque.*
2. [[CloudWatchLogs]] — log groups/streams, metric filters. *Extiende CloudWatch a logs.*
3. [[CloudTrail]] — auditoría de API, trails, global service events. *Después de Logs porque sus eventos suelen mandarse ahí.*
4. [[EventBridge]] — reaccionar a eventos casi en tiempo real. *Cierra el ciclo observar → reaccionar; contrasta con el delay de CloudTrail.*
5. [[observability-costs]] — qué es gratis, qué se cobra y los 3 motores de gasto. *Va último del bloque: solo tiene sentido sabiendo qué son métricas custom, data events y dimensions. Cierra con la regla que ordena todo: el gobierno (IAM/Organizations/SCPs) es gratis, lo que se paga es observar.*

> 📖 Lectura profunda: [[01.11 CloudWatch — Basics]] · [[03.12 CloudWatch Logs]] · [[03.13 CloudTrail]] · [[03.14 Precios]]

## Bloque 7 — Infraestructura como código

1. [[CloudFormation]] — templates y secciones. *Va último por ahora: declara recursos de todos los bloques anteriores. Página inicial, se ampliará con el curso.*

> 📖 Lectura profunda: [[01.10 CloudFormation — Basics]]

---

## Sugerencia de ruta

- **Si arrancás de cero**: bloques en orden, 1 → 7.
- **Si ya viste el curso** (este material sale de tus notas): usá los bloques como checklist de repaso y saltá directo a las páginas de comparación ([[s3-storage-classes]], [[s3-encryption]], [[alias-vs-cname]], [[security-groups-vs-nacls]], [[nat-gateway-vs-nat-instance]], [[gateway-vs-interface-endpoint]], [[ebs-volume-types]], [[instance-store-vs-ebs]], [[ec2-purchase-options]], [[horizontal-vs-vertical-scaling]]) + las secciones "Gotchas" de cada página, que es donde vive el jugo de examen.
- **Última pasada antes del examen**: los dos cheat sheets, [[vpc-cheat-sheet]] y [[ec2-cheat-sheet]]. **No están en la secuencia** — no enseñan nada nuevo, condensan lo ya leído en tablas de "escenario → respuesta" y listas de números. Leerlos antes de estudiar el tema no sirve.
- **Práctica activa**: al terminar un bloque, pedí un **quiz** ("quiz bloque 2") para repasar en activo lo que acabás de leer.

## Mapas de dominio (transversales, no secuenciales)

Las páginas de dominio de DVA-C02 no forman parte de la secuencia de estudio: son **mapas** que cruzan estos bloques con el temario oficial del examen y marcan los huecos pendientes de ingest. Consultalas para saber *cuánto del examen* cubre lo que ya estudiaste:

- [[dva-development]] (32%) · [[dva-security]] (26%) · [[dva-deployment]] (24%) · [[dva-troubleshooting]] (18%)
