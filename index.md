# Index — AWS Knowledge Base

> El LLM mantiene este archivo actualizado en cada ingest. Leer primero al responder queries.

**Total de páginas:** 81 (28 de conocimiento + 53 términos de glosario)
**Fuentes ingestadas:** notas del curso **completas (51/51 secciones), incluidas las ampliaciones de los módulos 02/03/04** + 49 clippings de doc oficial
**Última actualización:** 2026-08-12

---

## Servicios (`wiki/services/`)

| Página | Servicio | Cubre |
|--------|----------|-------|
| [[S3]] | Simple Storage Service | Buckets/objects, seguridad, versioning, performance, replication, presigned URLs, CORS, events, object lock |
| [[IAM]] | Identity and Access Management | Users, groups, roles, STS, access keys, federación, límites |
| [[EC2]] | Elastic Compute Cloud | Estados, AMI, storage, key pairs, conexión |
| [[VPC]] | Virtual Private Cloud | Default vs custom VPC, subnets, IGW |
| [[KMS]] | Key Management Service | KMS keys, DEKs, envelope encryption, key policies, rotación |
| [[Organizations]] | AWS Organizations | Estructura, consolidated billing, SCPs |
| [[Route53]] | Route 53 | Hosted zones, tipos de record DNS, TTL, alias |
| [[CloudFormation]] | CloudFormation | Templates, secciones (inicial) |
| [[CloudWatch]] | CloudWatch | Namespaces, metrics, dimensions, alarms |
| [[CloudWatchLogs]] | CloudWatch Logs | Log groups/streams, metric filters, retención |
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
[[principal]] · [[trust-policy]] · [[permissions-boundary]] · [[least-privilege]] · [[federation]] · [[temporary-credentials]] · [[confused-deputy]] · [[instance-profile]] · [[abac]] · [[service-linked-role]] · [[break-glass]] · [[cross-account]] · [[external-id]]

**Cifrado**
[[envelope-encryption]] · [[data-encryption-key]] · [[encryption-context]] · [[role-separation]] · [[encryption-at-rest]]

**Almacenamiento y datos**
[[object-storage]] · [[durability]] · [[availability]] · [[eventual-consistency]] · [[delete-marker]] · [[multipart-upload]] · [[prefix]] · [[worm]] · [[etag]] · [[presigned-url]]

**Arquitectura**
[[idempotency]]

**Redes y DNS**
[[cidr]] · [[zone-apex]] · [[ttl]]

**Observabilidad y costos**
[[dimension]] · [[high-cardinality]] · [[metric-filter]]

**Límites y performance**
[[throttling]]

### Backlog de términos pendientes

Identificados en las páginas actuales, todavía sin entrada — se van agregando en los próximos ingests:

`block-storage` · `file-storage` · `strong-consistency` · `rpo` · `rto` · `soft-limit` · `registrar` · `registry` · `registrant` · `fqdn` · `role-chaining` · `zone-of-trust` · `golden-ami` · `sigv4` · `session-policy`

---

## Dominios del examen (`wiki/domains/`)

### DVA-C02 — AWS Certified Developer – Associate *(examen en foco)*

| Página | Dominio | Examen | Peso | Cobertura actual |
|--------|---------|--------|------|------------------|
| [[dva-development]] | Development with AWS Services | DVA-C02 | 32% | ⚠️ Parcial — faltan Lambda, API GW, DynamoDB, SQS/SNS |
| [[dva-security]] | Security | DVA-C02 | 26% | ✅ Fuerte — falta Cognito, Secrets Manager |
| [[dva-deployment]] | Deployment | DVA-C02 | 24% | ❌ Mínima — faltan Code*, SAM, Beanstalk |
| [[dva-troubleshooting]] | Troubleshooting and Optimization | DVA-C02 | 18% | ✅ Buena — falta X-Ray |

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

---

## Fuentes en `raw/`

### raw/notas curso mejorado/ (ingestadas 2026-07-18; segmentadas por sección desde 2026-07-18)

Estructura: un archivo por sección (`NN.MM Título.md`) con navegación, + índice por módulo (`NN.00 … — Índice.md`) + [[00 Índice general]].

- `01 Fundamentos de AWS/` (15 secciones) — Global Infra, VPC, EC2, S3 basics, CFN, CloudWatch, Shared Responsibility, HA/FT/DR, Route 53, DNS
- `02 Fundamentos y cuenta AWS/` (5 secciones) — Account, root user, MFA, IAM basics, access keys, CLI
- `03 IAM ACCOUNTS y AWS Organization/` (14 secciones) — IAM en profundidad, STS, Organizations, SCPs, CloudWatch Logs, CloudTrail, **precios/costos (03.14, ingestada 2026-07-24)**
- `04 S3/` (17 secciones) — S3 en profundidad, KMS, encryption, storage classes, lifecycle, replication, presigned URLs, CORS, object lock

> ✅ **Las 51 secciones están ingestadas, en su versión ampliada.** Los módulos **02, 03 y 04** fueron expandidos por el humano el 2026-07-23/24 e ingestados el 2026-07-25 (el módulo 01 ya se había ingestado el 2026-07-23). El próximo material tiene que venir de módulos nuevos del curso (05+) o de clippings nuevos. `raw/definiciones/` existe pero está **vacía**.

Cada página wiki cita en `sources` los **segmentos específicos** que la alimentan.

### raw/doc oficial/ (49 archivos de doc oficial AWS, curados)

Complementan las notas con límites numéricos, permisos exactos y features no cubiertas por el curso. Citados en el frontmatter `sources` de cada página que los usa.

- 28 clippings ingestados 2026-07-18 (primer lote).
- 21 clippings ingestados 2026-07-19 (segundo lote): S3 Access Points (6), Block Public Access, condition keys de bucket policies, MFA Delete, Versioning, Batch Operations, KMS Grants y Multi-Region keys, IAM permissions boundaries / STS / Access Analyzer, SCPs y su evaluación, CloudTrail events + log file integrity, metric filters de CloudWatch Logs.

Los task statements oficiales citados en las 4 páginas de `wiki/domains/` vienen del [AWS Certified Developer - Associate (DVA-C02) — Exam Guide oficial](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html).
