# Log — AWS Knowledge Base

> Append-only. Formato de cabecera: `## [YYYY-MM-DD] tipo | descripción`
> Tipos: `setup` | `ingest` | `query` | `lint` | `update`

---

## [2026-06-13] setup | Inicialización de la knowledge base

- Creada estructura de carpetas: `raw/`, `wiki/services/`, `wiki/concepts/`, `wiki/domains/`, `wiki/comparisons/`, `wiki/exams/`
- Creado `CLAUDE.md` con schema, convenciones y flujos de trabajo
- Creado `index.md` (catálogo vacío)
- Creado `log.md` (este archivo)
- Examen en foco: **DOP-C02** — AWS Certified DevOps Engineer – Professional
- Próximos exámenes en cola: SAA-C03, DVA-C02
- Estado: wiki vacía, lista para primer ingest

---

## [2026-09-19] ingest | Módulo 05 VPC + 37 clippings de doc oficial de Amazon VPC

**Fuentes:**
- `raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/` — 8 secciones nuevas (05.01–05.08), segmentadas el mismo día desde `raw/notas curso/Virtual private cloud (VPC) Basics.md`.
- `raw/doc oficial/` — 37 clippings nuevos de Amazon VPC (tercer lote; total 86).
- Chequeo de modificados: ninguna fuente ya ingestada cambió (los mtime de 01.00/01.01/01.04 se movieron, pero `git diff` no muestra cambios).

**Páginas nuevas (3):** [[vpc-design]] (concept), [[security-groups-vs-nacls]] y [[nat-gateway-vs-nat-instance]] (comparisons).

**Páginas actualizadas:** [[VPC]] (reescrita y ampliada de 82 a ~270 líneas: CIDR/IPv6, subnets, DNS/DHCP, routing, IGW, bastion, SGs, NACLs, NAT GW zonal/regional, NAT64, demos), [[EC2]] (integración), [[dva-development]] (Lambda en VPC), [[dva-troubleshooting]] (fallas de conectividad).

**Glosario (+12):** [[ephemeral-port]] · [[stateful-firewall]] · [[stateless-firewall]] · [[implicit-deny]] · [[eni]] · [[elastic-ip]] · [[public-subnet]] · [[bastion-host]] · [[longest-prefix-match]] · [[ip-masquerading]] · [[egress-only-internet-gateway]] · [[dedicated-tenancy]]. Backlinks actualizados en [[cidr]], [[az-resilient]], [[region-resilient]], [[multi-az]], [[single-point-of-failure]].

**Contradicciones curso ↔ doc (resueltas a favor de la doc, marcadas `⚠️ Outdated` en [[VPC]]):**
- IPv6: el curso dice "un único `/56` por VPC"; la doc admite hasta 5 CIDRs IPv6 (el Amazon-provided sigue siendo `/56`).
- Local route: el curso dice que no se modifica; la doc permite rutas más específicas (CIDR completo de una subnet → NAT GW/ENI/GWLB endpoint) y reemplazar su target.
- NAT Gateway: 45 Gbps (curso) → 5 Gbps escalando a 100 Gbps (doc); existe el modo **regional** (nov-2025); hace **NAT64** con IPv6; el NAT GW no es el único que usa Elastic IPs.

**Otros:** [[index]] (96 páginas), [[guia-estudio]] (Bloque 3 → "Redes (VPC) y DNS", 6 páginas), [[demos]] (+5 demos de VPC, total 27).

## [2026-09-19] lint | Revisión completa post-ingest VPC

**Sin problemas:** 0 links rotos, 0 páginas huérfanas, las 96 páginas de `wiki/` están en [[index]], frontmatter completo y `category` coherente con la carpeta en todas.

**Corregido:**
- **Links de glosario faltantes (primera aparición sin link):** 30 en 17 páginas. En las comparaciones de VPC recién creadas: bastion, public subnet, AZ/region resilient, failover, ENI. En páginas viejas: [[S3]] (data plane, control plane, soberanía, deny implícito, serverless), [[KMS]] (confused deputy, trust policy), [[CloudFront]] (confused deputy, presigned URL), [[iam-policy-evaluation]], [[permissions-boundary]], [[s3-encryption]] (envelope encryption), [[s3-storage-classes]] (WORM), [[global-infrastructure]] (blast radius, Multi-AZ), [[Organizations]] (least privilege), [[EC2]], [[CloudFormation]] y [[EventBridge]] (serverless), [[multipart-upload]] (ETag), [[VPC]] (throttling), [[dva-development]] (idempotencia), [[dva-troubleshooting]] (ephemeral ports).
- **"Ver también" recíprocos:** [[least-privilege]] → [[abac]]; [[confused-deputy]] → [[cross-account]].
- **Dato desactualizado por el ingest:** [[global-infrastructure]] listaba "NAT Gateway" como AZ resilient sin matiz → ahora "NAT Gateway zonal" + mención del modo regional.

**Contradicciones:** ninguna. Se cruzaron cifras repetidas entre páginas (throughput S3 por prefijo, throttling KMS, duración STS, límites IAM, CIDR de VPC, ancho de banda NAT GW, storage classes) y coinciden.

**Pendiente (sugerencias, no aplicado):**
- Páginas viejas que no siguen la plantilla de `CLAUDE.md`: los concepts [[arn]], [[aws-account]], [[aws-cli]], [[iam-policy-evaluation]], [[observability-costs]] no tienen "Cómo aplica"/"Patrones comunes" ([[global-infrastructure]], [[ha-ft-dr]], [[shared-responsibility-model]] no tienen "Patrones comunes"); los services [[CloudTrail]], [[CloudWatch]], [[CloudWatchLogs]], [[Organizations]] no tienen "Casos de uso".
- Servicios muy mencionados sin página: **Lambda** (22 páginas), RDS, DynamoDB (12), VPC endpoints/PrivateLink, ELB (11), SNS, SQS, API Gateway, EBS. Lambda, DynamoDB, API Gateway, SQS/SNS son además huecos del DVA-C02.
- 3 entradas de "Dónde aparece" son relaciones temáticas sin mención literal en la página destino ([[object-storage]] ← [[EC2]] y [[s3-storage-classes]], [[availability]] ← [[Route53]]); se dejaron así.
- Backlog de glosario sin cambios (15 términos).

## [2026-09-19] update | Segmentación del contenido nuevo de VPC (05.09–05.13)

**Fuente:** `raw/notas curso/Virtual private cloud (VPC) Basics.md`, ampliada por el humano con 5 temas nuevos al final (líneas 680–792): VPC Endpoints, VPC Flow Logs, Lambda en una VPC, VPC Peering y el cheat sheet del módulo. Se verificó que el resto del archivo (líneas 1–679) coincide con lo ya segmentado en 05.01–05.08.

**Secciones nuevas en `raw/notas curso mejorado/05 Virtual private cloud (VPC) Basics/` (5):**
- `05.09 VPC Endpoints` — + cómo se ve la ruta con prefix list, DNS privado vs nombre propio del endpoint, nota de endpoints dualstack (`Revisar`), mención del Gateway Load Balancer Endpoint.
- `05.10 VPC Flow Logs` — + alcance y ciclo de vida (no retroactivos, inmutables, intervalo 10/1 min), tabla de campos, cómo deducir si bloqueó el SG o la NACL leyendo ACCEPT/REJECT, criterio de destino (CloudWatch Logs vs S3 vs Firehose).
- `05.11 Lambda en una VPC` — + tabla de las tres configuraciones posibles, trampa de la subnet pública, permisos `AWSLambdaVPCAccessExecutionRole`, límite de IPs libres (`ENILimitReached`).
- `05.12 VPC Peering` — + los tres pasos (aceptar, rutas en ambos lados, SG/NACL), por qué no pueden solapar los CIDRs, qué NO hace (sin edge-to-edge routing, DNS a habilitar), fórmula N×(N-1)/2.
- `05.13 Resumen para el examen (cheat sheet)` — cheat sheet con link a la sección de cada punto, tabla de números de memoria, tabla “escenario → respuesta” y los cuatro errores más repetidos.

**Actualizados:** navegación de `05.08 NAT y NAT Gateway` (ahora enlaza a 05.09), `05.00 Índice` del módulo y `00 Índice general` (módulo 05: 8 → 13 secciones). 0 links rotos.

**Pendiente:** este contenido todavía **no está ingestado en `wiki/`** — faltan VPC Endpoints, Flow Logs y Peering en [[VPC]], y los términos de glosario que introducen (privatelink, vpc-endpoint, transit-gateway, traffic-mirroring).

## [2026-09-22] update | Segmentación del módulo 06 — EC2

**Fuente:** `raw/notas curso/Elastic Compute Cloud EC2 Basics.md` (1057 líneas), módulo nuevo del curso que no tenía versión segmentada. Es la fuente más larga hasta ahora.

**Secciones nuevas en `raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/` (24):**
- **Fundamentos (06.01–06.04):** Virtualization 101 (binary translation, para-virtualization, hardware-assisted, SR-IOV/Enhanced Networking, Nitro), EC2 Architecture and Resilience (shared vs dedicated hosts, AZ-resilient, ciclo de vida stop/start vs reboot), Instance Types (5 categorías + esquema de nombres), Storage Refresh (DAS/NAS, block/file/object, IOPS × block size = throughput).
- **EBS (06.05–06.13):** basics, los tres tipos de volumen en archivo propio (gp2/gp3 con el sistema de créditos y el umbral de 1 TB · io1/io2/Block Express con el ratio IOPS/GB y el tope por instancia · st1/sc1 con bloques de 1 MB), Instance Store, la comparación Instance Store vs EBS, snapshots + lazy restore + FSR, la demo de 3 partes con sus comandos, y EBS Encryption (envelope encryption con DEK por volumen).
- **Red (06.14–06.16):** ENI y el zoológico de direcciones IP, DNS split-horizon, Elastic IP, y la demo de instalación manual de WordPress.
- **AMI y compra (06.17–06.21):** AMI lifecycle y baking; Purchase Options repartidas en On-Demand+Spot, Reserved+Savings Plans, Dedicated Hosts+Instances y Capacity Reservations.
- **Operación (06.22–06.24):** status checks y auto recovery, horizontal vs vertical scaling, Instance Metadata.

**Correcciones al original:** bloque duplicado en Instance Store (se repetía casi palabra por palabra, incluido el "Resumen mental"); párrafo de Elastic IP duplicado (callout + prosa); typos de encabezado (`Priovisioned` → `Provisioned`, `BS Snapshots` huérfano, `### ###` en la demo de WordPress); Status Checks / Scaling / Instance Metadata venían como `###` siendo temas de primer nivel.

**Agregado respecto al curso (marcado como callout separado):**
- **IMDSv2** (06.24) — la fuente solo cubre IMDSv1. Se agrega el flujo token-based (`PUT /latest/api/token` + `X-aws-ec2-metadata-token`), por qué corta el SSRF que la propia nota describe, y `HttpTokens: required`. Es material de examen y era un hueco.
- **Cobro de IPv4 públicas** (06.15) — el curso dice que se cobra la Elastic IP *ociosa*; desde feb-2024 se cobra **toda** IPv4 pública, en uso o no.
- **Auto-recovery por defecto** (06.22) — el curso lo presenta como configurable; desde 2022 viene habilitado por defecto.
- **Generaciones de instance types** (06.03) — la tabla del curso se deja intacta con una nota de que las familias siguen valiendo aunque las generaciones avanzaron.

**Solapamiento con el módulo 01 (resuelto apuntando, sin duplicar):** 06.17 AMI remite a [[01.06 Amazon Machine Image (AMI)]] para permisos/root volume/block device mapping y desarrolla solo lifecycle, baking, Marketplace y demos; 06.10 remite a la tabla de [[01.05 Elastic Compute Cloud (EC2) — Basics]] y agrega los escenarios de decisión y la escalera de IOPS.

**Actualizados:** `06.00 Índice` del módulo (nuevo) y `00 Índice general` (se agrega el bloque del módulo 06, 24 secciones). 0 links rotos, las 35 imágenes referenciadas resuelven contra `raw/assets/`.

**Pendiente:** este contenido todavía **no está ingestado en `wiki/`**. La deuda de ingest es ahora doble: VPC 05.09–05.13 (endpoints, flow logs, Lambda en VPC, peering, cheat sheet) y el módulo 06 completo — que además abre temas sin página propia: EBS, instance store, snapshots, instance types, purchase options, IMDS y scaling.

## [2026-09-22] ingest | VPC 05.09–05.13 + módulo 06 (EC2) completo

**Fuentes:** `raw/notas curso mejorado/05 …/05.09`–`05.13` (5 secciones, segmentadas el 2026-09-19) y `raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.01`–`06.24` (24 secciones, segmentadas hoy). Saldadas las dos deudas que arrastraba el log. Sin clippings nuevos en `doc oficial/`: **este ingest es solo de notas del curso**, sin contraste contra doc oficial.

**Páginas nuevas de conocimiento (14):**
- **VPC (5):** [[vpc-endpoints]] y [[gateway-vs-interface-endpoint]] (gateway vs interface, PrivateLink, endpoint policy, private DNS), [[vpc-flow-logs]] (metadatos, campos, cómo deducir si cortó el SG o la NACL), [[vpc-peering]] (no transitivo, CIDRs sin solapar, los 3 pasos, N×(N-1)/2), [[lambda-in-vpc]] (las 3 configuraciones, la trampa de la subnet pública, `ENILimitReached`).
- **EC2 (9):** [[EBS]] como servicio, [[ebs-volume-types]], [[instance-store-vs-ebs]], [[ec2-purchase-options]], [[horizontal-vs-vertical-scaling]], [[storage-types]], [[ec2-instance-types]], [[ec2-instance-metadata]], [[virtualization]].

**Páginas de examen (2), estrena `wiki/exams/`:** [[vpc-cheat-sheet]] (adaptado de 05.13, relinkeado a la wiki) y [[ec2-cheat-sheet]] (destilado propio del módulo 06 — el curso no lo trae).

**Decisiones de estructura:**
- **[[EC2]] reescrita como hub.** Pasó de 110 a ~210 líneas sumando arquitectura y hosts, ciclo de vida, ENI/IPs/DNS y split-horizon, Elastic IP, AMI lifecycle y status checks — y **delegando** los 9 subtemas grandes en vez de desarrollarlos, para no convertirla en una página de 600 líneas.
- **Lambda va como concept, no como service.** La fuente solo cubre el ángulo de red; abrir `services/Lambda.md` con eso sería un stub que miente sobre la cobertura. Sigue siendo el hueco #1 de [[dva-development]].
- **[[storage-types]] como página bisagra** entre EBS, S3 y EC2: cierra `block-storage` y `file-storage` del backlog de glosario **sin** entrada propia, aplicando la regla de `CLAUDE.md` de linkear la página existente en vez de duplicar un stub.

**Glosario (+18, total 83):** [[privatelink]] · [[prefix-list]] · [[endpoint-policy]] · [[transit-gateway]] · [[traffic-mirroring]] · [[edge-to-edge-routing]] · [[execution-role]] · [[iops]] · [[throughput]] · [[burst-credit]] · [[ephemeral-storage]] · [[lazy-restore]] · [[ebs-optimized]] · [[enhanced-networking]] · [[nitro]] · [[host-affinity]] · [[golden-ami]] · [[stateless]]. Backlinks actualizados en 18 entradas existentes ([[eni]], [[elastic-ip]], [[instance-profile]], [[envelope-encryption]], [[hypervisor]]…). Backlog: `golden-ami`, `block-storage` y `file-storage` cerrados; se suman `hyperplane-eni` y `placement-group`.

**Páginas actualizadas:** [[VPC]] (la sección "Conectividad con otras redes" pasó de 6 bullets sueltos a contenido real + 7 gotchas nuevos), [[KMS]], [[S3]], [[IAM]], [[CloudWatch]], [[CloudWatchLogs]], [[vpc-design]], [[security-groups-vs-nacls]] (tabla de cómo leer un REJECT), [[nat-gateway-vs-nat-instance]] (endpoints como tercera opción), [[global-infrastructure]], [[ha-ft-dr]] (snapshots como DR), [[observability-costs]] (costos de cómputo y red).

**Dominios:** [[dva-security]] ✅ reforzada (IMDSv2/SSRF, cifrado EBS, endpoint policies) · [[dva-troubleshooting]] ✅ reforzada (flow logs, status checks, créditos de gp2, lazy restore, `ENILimitReached`) · [[dva-deployment]] ❌→⚠️ (AMI baking vs user data; ya no está vacía) · [[dva-development]] ⚠️ (Lambda en VPC con página propia; el resto del dominio sigue siendo el hueco grande).

**Actualizaciones posteriores al curso (marcadas aparte del texto del curso):**
- **IMDSv2** — la fuente solo cubre IMDSv1. Es la mitigación del SSRF que la propia nota describe; `HttpTokens: required`.
- **Cobro de toda IPv4 pública** desde feb-2024 (el curso dice solo las Elastic IP ociosas) → [[EC2]] y [[observability-costs]].
- **Auto-recovery por defecto** desde 2022 → [[EC2]].
- **Endpoints dualstack** (el curso dice "solo TCP sobre IPv4") → [[vpc-endpoints]].
- **Scheduled Reserved Instances** discontinuadas y **gp3 como default de AWS** → [[ec2-purchase-options]], [[ebs-volume-types]].
- **Generaciones de instance types** — nota sin reescribir la tabla del curso, porque lo que se evalúa es el esquema de nombres.

**Otros:** [[index]] (96 → **130** páginas: 45 de conocimiento + 83 de glosario + 2 de examen), [[guia-estudio]] (**dos reordenamientos reales**: [[storage-types]] se adelanta del bloque de S3 al de cómputo por ser prerequisito de ambos, y el Bloque 4 deja de ser una página para pasar a ser el más largo de la guía con 10), [[demos]] (+12 demos de EC2, total 27 → **39**; 05.09–05.13 no trae ninguna).

**Lint incidental:** el chequeo de escapes encontró **8 pipes de alias sin escapar dentro de tablas** en 6 páginas viejas ([[s3-storage-classes]], [[global-infrastructure]], [[ha-ft-dr]], [[iam-policy-evaluation]], [[observability-costs]], [[S3]]) — Obsidian los interpretaba como separador de celda y rompía el render. Corregidos. El lint del 2026-09-19 no los había detectado porque buscaba links faltantes, no escapes.

**Pendiente:** no hay **ningún clipping de doc oficial de EC2 ni de EBS**, así que los límites numéricos del módulo 06 están sin cruzar contra la doc — conviene traer ese lote. Huecos de contenido que siguen: **Lambda** (página de servicio), API Gateway, DynamoDB, SQS/SNS, X-Ray, Cognito, Secrets Manager y todo el stack de Code*/SAM/Beanstalk.

## [2026-09-23] ingest | Ingest profundo: re-verificación de fuentes modificadas después del 09-22

**Detección** (según el ⚠️ del flujo de Ingest): 0 archivos nunca citados (salvo los índices de módulo, que no se ingestan), pero **31 fuentes con mtime posterior al `updated:` de las páginas que las citan**. Triage:
- **01.xx / 02.xx / 03.xx / clippings de IAM, Route 53, CloudTrail, ARN** → `git diff HEAD` vacío: **idénticos al commit del 08-13**. El mtime reciente fue un touch sin cambio. Igual se abrieron y se compararon contra [[shared-responsibility-model]], [[aws-account]], [[Route53]], [[alias-vs-cname]], [[arn]], [[aws-cli]], [[CloudTrail]] e [[IAM]]: todo cubierto. Se les actualizó `updated:` para que el detector no las vuelva a marcar.
- **05.01–05.04, 05.08–05.12 y 06.16** → editados por el humano el 2026-09-23 entre 00:39 y 01:26. Entraron a git como `A`, así que git no muestra el cambio: se leyeron completos y se compararon a mano.

**Cambios reales encontrados e ingestados:**
- **05.02** — ahora dice que el IPv6 es "5 CIDRs, ampliable a 50" (antes: "un único `/56`"). **Verificado contra [Amazon VPC quotas](https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html)**: es correcto. La misma página reveló un **error de la wiki**: los 5 CIDRs IPv4 **incluyen el primario** (4 secundarios por defecto, ampliable a 50); [[VPC]], [[cidr]] y [[vpc-design]] decían "hasta 5 secundarios". Corregido. También se corrigió en la tabla Default vs Custom "Muchas" → "5 por region (soft limit)".
- **05.03** — fórmula `usables = 2^(32−p) − 5` y por qué el mínimo es `/28` → [[vpc-design]]; caso de uso del DHCP option set (DNS de Active Directory) y "no hay DNS distinto por subnet" → [[VPC]].
- **05.04** — receta del IGW ahora en **6 pasos**; los **dos casos de override de la local route** (ruta más específica en una subnet route table vs reemplazar el target en una **gateway route table**) como tabla con ejemplos; **checklist de 5 pasos** para "la instancia en subnet pública no tiene internet"; bastion vs **Session Manager** (por qué es mala práctica + requisitos: SSM Agent y rol IAM) → [[VPC]] y [[bastion-host]].
- **05.08** — el cuerpo ya dice 100 Gbps y dejó de decir "único servicio con Elastic IPs"; la tabla todavía dice 45 Gbps → se reescribieron los callouts "Outdated" de [[VPC]] y [[nat-gateway-vs-nat-instance]] para reflejarlo. Se sumó la regla de decisión "servicios de AWS → endpoint, no NAT".
- **05.09–05.12** — sin contenido nuevo: el humano las marcó **"(Pendiente a revisar)"**. Se agregó un aviso ⚠️ en [[vpc-endpoints]], [[gateway-vs-interface-endpoint]], [[vpc-flow-logs]], [[lambda-in-vpc]] y [[vpc-peering]]: salen de esas secciones y **no están contrastadas con doc oficial**.
- **06.16** — solo arregla comandos de la demo de WordPress (faltaban `$` en las variables). Sin impacto en la wiki: [[EC2]] y [[demos]] solo linkean la demo.
- **Clippings de IAM** (sin cambios, pero con detalles que faltaban) → [[IAM]]: un user creado por CLI/API nace sin credenciales, `aws login`, "service account" sin keys en el código, server certificates → ACM, y que role chaining con `DurationSeconds` > 3600 falla.
- **01.12** — faltaba la columna *DC Hosted* en la tabla de [[shared-responsibility-model]].
- **Extra de la doc de cuotas**: 60 reglas por SG, 5 SGs por ENI (hasta 16), 20 reglas por NACL (hasta 40) → [[VPC]].

**Sin páginas nuevas** → [[index]] y [[guia-estudio]] sin cambios (no hubo que reubicar ni reordenar nada). Sin demos nuevas.

## [2026-09-23] lint | Revisión completa post-ingest

**Estructura:** 0 links rotos · 0 imágenes rotas · 0 páginas huérfanas · las 130 páginas en [[index]] · todas las páginas que no son glosario en [[guia-estudio]] · 0 pipes de alias sin escapar en tablas (el chequeo del 09-22 sigue sirviendo) · 0 contradicciones en los datos numéricos revisados en varias páginas (NAT GW, límites de S3, duración de STS, retención de CloudTrail, reglas de NACL, access keys).

**Corregido:**
- [[vpc-cheat-sheet]] tenía **un solo link entrante** (desde [[ec2-cheat-sheet]]); ni [[VPC]] ni [[EC2]] linkeaban a su cheat sheet. Ahora las dos lo hacen desde "Gotchas".
- **47 entradas de glosario** con backlinks reales que no figuraban en "Dónde aparece" (sobre todo desde los cheat sheets, las páginas de dominio y las páginas nuevas de EC2/VPC). Se agregó una línea "También en: …" a cada una, sin tocar las líneas escritas a mano.

**Pendiente (no se tocó; decide el humano):**
- **53 entradas de glosario con `sources: []`** — son las anteriores al 09-19. Completarlas implica rastrear de qué fuente sale cada término.
- **El log no tiene entradas entre el 06-13 y el 09-19**: los ingests de los módulos 01–04 nunca se registraron. Como el log es append-only, no se reconstruyen.
- **Problemas en `raw/` (el LLM no los edita):** (1) en 05.09, 05.12 y 05.13 el ancla `[[05.04 …#Prioridad de las routes]]` está rota — el encabezado real es "Route Tables: prioridad de routes"; (2) 05.02 línea 52 tiene un `**` suelto ("5 CIDR block IPv6 /56**"); (3) la tabla de 05.08 todavía dice "45 Gbps"; (4) 05.01: "para le ejemplo" y doble negación en "No se recomienda no usar direcciones que empiecen con 10.0".
- **Temas sin página propia, por cantidad de páginas que los mencionan:** STS (42, cubierto dentro de [[IAM]]), **Lambda (32)**, RDS (23), **DynamoDB (22)**, ELB/ALB (17), SNS (14), SQS (13), **API Gateway (10)**, Systems Manager/Session Manager (8), Direct Connect (8), Auto Scaling (5). Para DVA-C02 las prioridades siguen siendo Lambda, DynamoDB y API Gateway; Session Manager ya tiene material suficiente para una entrada de glosario o de concepto.
- **Sin clippings de doc oficial** para endpoints, flow logs, peering, Lambda en VPC, ni para EC2/EBS: los datos del curso quedan sin contrastar (sobre todo 05.09–05.12, que el humano marcó para revisar).
