# Index — AWS Knowledge Base

> El LLM mantiene este archivo actualizado en cada ingest. Leer primero al responder queries.

**Total de páginas:** 137 (46 de conocimiento + 89 términos de glosario + 2 de examen)
**Fuentes ingestadas:** notas del curso **completas (96/96 secciones, módulos 01–07)** + 86 clippings de doc oficial
**Última actualización:** 2026-09-24

---

## Servicios (`wiki/services/`)

| Página | Servicio | Cubre |
|--------|----------|-------|
| [[S3]] | Simple Storage Service | Buckets/objects, seguridad, versioning, performance, replication, presigned URLs, CORS, events, object lock |
| [[IAM]] | Identity and Access Management | Users, groups, roles, STS, access keys, federación, límites |
| [[EC2]] | Elastic Compute Cloud | Hub del tema: arquitectura y hosts, ciclo de vida, estados, ENI/IPs/DNS, Elastic IP, AMI y lifecycle, status checks, key pairs |
| [[EBS]] | Elastic Block Store | Volúmenes, tipos, snapshots incrementales, lazy restore/FSR, cifrado con DEK por volumen |
| [[VPC]] | Virtual Private Cloud | Default vs custom, CIDR/IPv6, subnets, DNS/DHCP, route tables, IGW, SGs y NACLs, NAT Gateway (zonal/regional), NAT64 |
| [[KMS]] | Key Management Service | KMS keys, DEKs, envelope encryption, key policies, rotación |
| [[Organizations]] | AWS Organizations | Estructura, consolidated billing, SCPs |
| [[Route53]] | Route 53 | Hosted zones, tipos de record DNS, TTL, alias |
| [[CloudFormation]] | CloudFormation | Templates, secciones (inicial) |
| [[CloudWatch]] | CloudWatch | Arquitectura (endpoint público, agent), namespaces, metrics, dimensions, resolution y retención, statistics, alarms (M of N, high resolution) |
| [[CloudWatchLogs]] | CloudWatch Logs | Log groups/streams, metric filters, retención, export a S3, subscriptions, agregación multi-cuenta |
| [[XRay]] | AWS X-Ray | Distributed tracing: trace/segments/subsegments, service map, integración por servicio, annotations vs metadata |
| [[CloudTrail]] | CloudTrail | Event history, trails, tipos de evento, global service events |
| [[CloudFront]] | CloudFront | CDN, OAC, signed URLs (inicial) |
| [[EventBridge]] | EventBridge | Hub de eventos, integración S3 (inicial) |

---

## Conceptos (`wiki/concepts/`)

| Página | Concepto |
|--------|----------|
| [[global-infrastructure]] | Regions, AZs, edge locations, zonas de red, niveles de resiliencia |
| [[aws-account]] | AWS Account, root user, MFA, multi-account |
| [[shared-responsibility-model]] | Responsabilidad AWS vs cliente por modelo de servicio |
| [[ha-ft-dr]] | High Availability vs Fault Tolerance vs Disaster Recovery, RPO/RTO |
| [[iam-policy-evaluation]] | Anatomía de policies, Deny>Allow>Default, combinación de los 9 tipos |
| [[arn]] | Formato ARN, wildcards, bucket vs objects |
| [[aws-cli]] | CLI v2, named profiles, cadena de credenciales, acceso programático |
| [[observability-costs]] | Qué es gratis y qué se cobra en gobierno/observabilidad, los 3 motores de costo, cómo controlarlos |
| [[vpc-design]] | Sizing y estructura de una VPC: rangos a evitar, AZs × tiers, tamaños de VPC, plan por region/cuenta |
| [[vpc-endpoints]] | Acceso privado a servicios AWS: gateway vs interface, PrivateLink, endpoint policy, private DNS |
| [[vpc-flow-logs]] | Metadatos del tráfico IP, formato del record, ejemplo ICMP, cómo deducir si cortó el SG o la NACL, criterio de destino |
| [[vpc-peering]] | Unir dos VPCs: no transitivo, CIDRs sin solapar, los tres pasos, qué NO hace |
| [[lambda-in-vpc]] | Las tres configuraciones de red de una función, la trampa de la subnet pública, ENILimitReached |
| [[storage-types]] | DAS vs NAS, block/file/object, efímero vs persistente, IOPS × block size = throughput |
| [[virtualization]] | De la binary translation a SR-IOV y Nitro; por qué el cifrado de EBS no pesa |
| [[ec2-instance-types]] | Las cinco categorías y el esquema de nombres `R5dn.8xlarge` |
| [[ec2-instance-metadata]] | IMDS, `169.254.169.254`, credenciales del role, IMDSv1 vs IMDSv2 y el SSRF |

---

## Glosario (`wiki/glossary/`)

Un archivo por **término**: definición corta y atómica de la jerga que aparece suelta en el resto de la wiki. No duplica páginas existentes — si el término ya es un servicio o concepto, se linkea esa página. Convenciones y criterio de corte en `CLAUDE.md` → *Páginas de glosario*.

**Resiliencia e infraestructura**
[[globally-resilient]] · [[region-resilient]] · [[az-resilient]] · [[edge-location]] · [[blast-radius]] · [[single-point-of-failure]] · [[data-sovereignty]] · [[multi-az]] · [[failover]] · [[sla]]

**Modelos de servicio**
[[iaas]] · [[paas]] · [[saas]] · [[serverless]] · [[hypervisor]]

**Planos de operación**
[[control-plane]] · [[data-plane]]

**Identidad y acceso**
[[principal]] · [[trust-policy]] · [[permissions-boundary]] · [[least-privilege]] · [[federation]] · [[temporary-credentials]] · [[confused-deputy]] · [[instance-profile]] · [[execution-role]] · [[abac]] · [[service-linked-role]] · [[break-glass]] · [[cross-account]] · [[external-id]]

**Cifrado**
[[envelope-encryption]] · [[data-encryption-key]] · [[encryption-context]] · [[role-separation]] · [[encryption-at-rest]]

**Almacenamiento y datos**
[[object-storage]] · [[durability]] · [[availability]] · [[eventual-consistency]] · [[delete-marker]] · [[multipart-upload]] · [[prefix]] · [[worm]] · [[etag]] · [[presigned-url]] · [[ephemeral-storage]] · [[lazy-restore]]

**Cómputo y virtualización**
[[nitro]] · [[enhanced-networking]] · [[ebs-optimized]] · [[host-affinity]] · [[golden-ami]]

**Arquitectura**
[[idempotency]] · [[stateless]]

**Redes y DNS**
[[cidr]] · [[zone-apex]] · [[ttl]] · [[public-subnet]] · [[longest-prefix-match]] · [[eni]] · [[elastic-ip]] · [[ip-masquerading]] · [[egress-only-internet-gateway]] · [[bastion-host]] · [[dedicated-tenancy]] · [[privatelink]] · [[prefix-list]] · [[endpoint-policy]] · [[transit-gateway]] · [[edge-to-edge-routing]] · [[traffic-mirroring]]

**Seguridad de red**
[[stateful-firewall]] · [[stateless-firewall]] · [[ephemeral-port]] · [[implicit-deny]]

**Observabilidad y costos**
[[dimension]] · [[high-cardinality]] · [[metric-filter]] · [[custom-metric]] · [[high-resolution-metric]] · [[percentile]] · [[subscription-filter]] · [[near-real-time]] · [[distributed-tracing]]

**Límites y performance**
[[throttling]] · [[iops]] · [[throughput]] · [[burst-credit]]

### Backlog de términos pendientes

Identificados en las páginas actuales, todavía sin entrada — se van agregando en los próximos ingests:

`strong-consistency` · `rpo` · `rto` · `soft-limit` · `registrar` · `registry` · `registrant` · `fqdn` · `role-chaining` · `zone-of-trust` · `sigv4` · `session-policy` · `hyperplane-eni` · `placement-group`

> Cerrados en el ingest del 2026-09-22: `golden-ami` (con entrada propia); `block-storage` y `file-storage` **sin** entrada, porque los cubre [[storage-types]] — la regla de `CLAUDE.md` dice linkear la página existente en vez de crear un stub duplicado.

---

## Dominios del examen (`wiki/domains/`)

### DVA-C02 — AWS Certified Developer – Associate *(examen en foco)*

| Página | Dominio | Examen | Peso | Cobertura actual |
|--------|---------|--------|------|------------------|
| [[dva-development]] | Development with AWS Services | DVA-C02 | 32% | ⚠️ Parcial — Lambda solo por el lado de red ([[lambda-in-vpc]]); faltan la página de Lambda, API GW, DynamoDB, SQS/SNS |
| [[dva-security]] | Security | DVA-C02 | 26% | ✅ Fuerte — reforzada con IMDSv2 y cifrado de EBS; falta Cognito, Secrets Manager |
| [[dva-deployment]] | Deployment | DVA-C02 | 24% | ⚠️ Mínima — ya cubre AMI baking; faltan Code*, SAM, Beanstalk |
| [[dva-troubleshooting]] | Troubleshooting and Optimization | DVA-C02 | 18% | ✅ Fuerte — reforzada con flow logs, status checks, créditos de EBS y el módulo 07 (X-Ray, subscriptions, resolution); faltan Logs Insights y EMF |

### DOP-C02 — AWS Certified DevOps Engineer – Professional

| Página | Dominio | Examen | Peso |
|--------|---------|--------|------|
| *pendiente* | SDLC Automation | DOP-C02 | 22% |
| *pendiente* | Configuration Management and IaC | DOP-C02 | 17% |
| *pendiente* | Resilient Cloud Solutions | DOP-C02 | 15% |
| *pendiente* | Monitoring and Logging | DOP-C02 | 15% |
| *pendiente* | Incident and Event Response | DOP-C02 | 14% |
| *pendiente* | Security and Compliance | DOP-C02 | 17% |

---

## Comparaciones (`wiki/comparisons/`)

| Página | Compara |
|--------|---------|
| [[s3-storage-classes]] | Standard / IA / One Zone-IA / Glacier x3 / Intelligent-Tiering + Lifecycle |
| [[s3-encryption]] | Client-Side vs SSE-C vs SSE-S3 vs SSE-KMS + Bucket Keys |
| [[alias-vs-cname]] | ALIAS records vs CNAME en Route 53 |
| [[security-groups-vs-nacls]] | Stateful vs stateless, SG vs NACL: nivel, allow/deny, orden de evaluación, trampas |
| [[nat-gateway-vs-nat-instance]] | NAT Gateway vs NAT instance + NAT GW zonal vs regional |
| [[gateway-vs-interface-endpoint]] | Los dos tipos de VPC endpoint: servicios, costo, seguridad, on-premises |
| [[ebs-volume-types]] | gp2 / gp3 / io1 / io2 / io2 Block Express / st1 / sc1 |
| [[instance-store-vs-ebs]] | Efímero vs persistente + la escalera de IOPS hasta el corte de los 260.000 |
| [[ec2-purchase-options]] | On-Demand / Spot / Reserved / Savings Plans / Dedicated / Capacity Reservations |
| [[horizontal-vs-vertical-scaling]] | Downtime, techo, granularidad y el requisito de sesiones off-host |

---

## Exámenes (`wiki/exams/`)

Material de **repaso rápido**: no reemplaza a las páginas, condensa lo que más se pregunta para la última pasada.

| Página | Cubre |
|--------|-------|
| [[vpc-cheat-sheet]] | Redes: números de memoria, escenario→respuesta, los errores más repetidos |
| [[ec2-cheat-sheet]] | Cómputo y almacenamiento: la escalera de IOPS, purchase options, IMDSv2 |

---

## Fuentes en `raw/`

### raw/notas curso mejorado/ (ingestadas 2026-07-18; segmentadas por sección desde 2026-07-18)

Estructura: un archivo por sección (`NN.MM Título.md`) con navegación, + índice por módulo (`NN.00 … — Índice.md`) + [[00 Índice general]].

- `01 Fundamentos de AWS/` (15 secciones) — Global Infra, VPC, EC2, S3 basics, CFN, CloudWatch, Shared Responsibility, HA/FT/DR, Route 53, DNS
- `02 Fundamentos y cuenta AWS/` (5 secciones) — Account, root user, MFA, IAM basics, access keys, CLI
- `03 IAM ACCOUNTS y AWS Organization/` (14 secciones) — IAM en profundidad, STS, Organizations, SCPs, CloudWatch Logs, CloudTrail, **precios/costos (03.14, ingestada 2026-07-24)**
- `04 S3/` (17 secciones) — S3 en profundidad, KMS, encryption, storage classes, lifecycle, replication, presigned URLs, CORS, object lock
- `05 Virtual private cloud (VPC) Basics/` (13 secciones) — VPC sizing, custom VPC, subnets, routing e IGW, stateful vs stateless, NACLs, security groups, NAT Gateway *(05.01–05.08, ingestadas 2026-09-19)*; VPC endpoints, flow logs, Lambda en VPC, peering y cheat sheet *(05.09–05.13, ingestadas 2026-09-22)*
- `06 Elastic Compute Cloud (EC2)/` (24 secciones, segmentadas e **ingestadas 2026-09-22**) — virtualización, arquitectura y resiliencia, instance types, storage refresh, EBS y sus tipos de volumen, instance store, snapshots y FSR, cifrado, ENI/IPs/DNS, Elastic IP, AMI, purchase options, status checks, scaling, IMDS, + 2 demos con comandos
- `07 Monitoring and logging/` (8 secciones, segmentadas e **ingestadas 2026-09-24**) — arquitectura de CloudWatch, namespace/datapoint/metric/dimensions, resolution/retention/statistics, alarms, arquitectura de CloudWatch Logs, subscriptions y agregación, X-Ray, VPC Flow Logs

> ✅ **Las 96 secciones están ingestadas.** El módulo **07** se ingestó el **2026-09-24**. El ingest del **2026-09-22** saldó las dos deudas que quedaban: VPC 05.09–05.13 y el módulo 06 completo. Antes: el módulo **05 (05.01–05.08)** se ingestó el 2026-09-19; los módulos **02, 03 y 04** fueron expandidos por el humano el 2026-07-23/24 e ingestados el 2026-07-25; el módulo 01 se ingestó el 2026-07-23. El próximo material tiene que venir de módulos nuevos del curso (08+) o de clippings nuevos. `raw/definiciones/` existe pero está **vacía**.

Cada página wiki cita en `sources` los **segmentos específicos** que la alimentan.

### raw/doc oficial/ (86 archivos de doc oficial AWS, curados)

Complementan las notas con límites numéricos, permisos exactos y features no cubiertas por el curso. Citados en el frontmatter `sources` de cada página que los usa.

- 28 clippings ingestados 2026-07-18 (primer lote).
- 21 clippings ingestados 2026-07-19 (segundo lote): S3 Access Points (6), Block Public Access, condition keys de bucket policies, MFA Delete, Versioning, Batch Operations, KMS Grants y Multi-Region keys, IAM permissions boundaries / STS / Access Analyzer, SCPs y su evaluación, CloudTrail events + log file integrity, metric filters de CloudWatch Logs.
- 37 clippings ingestados 2026-09-19 (tercer lote, **Amazon VPC**): qué es y cómo funciona, connectivity options, CIDR blocks/IP addressing/subnets, DNS (Route 53 Resolver, atributos), DHCP option sets (3), route tables y prioridad de rutas, internet gateway, NAT devices/NAT gateways/regional NAT/NAT64/NAT instance, comparativa NAT, security groups (8), NACLs, infrastructure security.

> ⚠️ **No hay clippings de EC2 ni de EBS.** El ingest del 2026-09-22 se apoyó **solo en las notas del curso**, sin contraste contra doc oficial. Las páginas de EC2/EBS marcan con `⚠️ Outdated` lo que se sabe desactualizado (IMDSv2, cobro de IPv4 públicas, auto-recovery por defecto, generaciones de instance types), pero conviene traer clippings de Amazon EC2 y EBS para cruzar límites numéricos.

> ⚠️ **Tampoco hay clippings de X-Ray, ni de las subscriptions/export de CloudWatch Logs.** [[XRay]] sale solo del curso (07.07); los complementos que no vienen de la slide (daemon UDP 2000, sampling, annotations) están marcados como tales.

Los task statements oficiales citados en las 4 páginas de `wiki/domains/` vienen del [AWS Certified Developer - Associate (DVA-C02) — Exam Guide oficial](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html).
