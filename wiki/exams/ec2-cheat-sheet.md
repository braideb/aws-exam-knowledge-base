---
title: EC2 y almacenamiento — Cheat sheet de examen
category: exam
tags: [ec2, ebs, storage, repaso, cheat-sheet, dva-c02]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.00 Elastic Compute Cloud (EC2) — Índice.md", "raw/notas curso mejorado/09 Advanced EC2/09.00 Advanced EC2 — Índice.md"]
updated: 2026-09-30
---

# EC2 y almacenamiento — Cheat sheet de examen

Destilado de los módulos 06 (EC2) y 09 (Advanced EC2). A diferencia del de VPC, el curso no trae este resumen: está armado con los mismos criterios. Cada punto linkea a su página.

## Los puntos, uno por línea

- **EC2:** [[iaas|IaaS]], **[[az-resilient|AZ-resilient]]**. La instancia vive en una subnet → una AZ. La resiliencia la diseñás vos. → [[EC2]]
- **Hosts:** *shared* (pagás por instancia) vs *dedicated* (pagás el host). → [[ec2-purchase-options]]
- **Ciclo de vida:** la instancia cambia de host si **falla el host** o si hacés **stop + start**. Un **reboot no**. Nunca cruza de AZ. → [[EC2]]
- **Instance type:** `R5dn.8xlarge` = familia · generación · capacidades · tamaño. 5 categorías. → [[ec2-instance-types]]
- **Storage:** block (bootea) / file / object. DAS = instance store, NAS = EBS. → [[storage-types]]
- **EBS:** block por red, **persistente**, **AZ-resilient**, por **GB-mes** aunque la instancia esté detenida. → [[EBS]]
- **Instance store:** local, **[[ephemeral-storage|efímero]]**, el más rápido, **solo se adjunta al lanzar**. → [[instance-store-vs-ebs]]
- **Snapshots:** a [[S3]] → **resilientes de región**. **Incrementales**; se cobran por **datos usados**. → [[EBS]]
- **Cifrado EBS:** [[envelope-encryption|envelope encryption]], **DEK única por volumen**, AES-256, **sin impacto de rendimiento**. No se puede descifrar. → [[EBS]]
- **ENI:** lleva IPs, DNS y **security groups**. La IPv4 privada es fija; la pública **cambia con stop+start**. → [[EC2]]
- **AMI:** **regional**, **no se edita**, privada por defecto. Borrarla **no borra sus snapshots**. → [[golden-ami]]
- **IMDS:** `169.254.169.254`. Entrega las credenciales del role. **IMDSv2** es la defensa contra [[ssrf|SSRF]]. → [[ec2-instance-metadata]]
- **User data:** script que corre **como root, solo en el primer launch**, máximo 16 KB, **no es seguro**. Si falla, la instancia queda `running` igual. → [[ec2-bootstrapping]]
- **Instance role:** se adjunta el **instance profile**; las credenciales llegan por el IMDS y STS las renueva. Las keys en disco **pisan** al rol. → [[IAM]]
- **Parameter Store:** String / StringList / **SecureString** (KMS). Jerarquías, versionado, **sin rotación**. → [[SSMParameterStore]]
- **CloudWatch Agent:** el interior de la instancia es opaco. Memoria, disco y logs del SO solo llegan con el agente + un rol. → [[CloudWatchLogs]]
- **Placement groups:** cluster (rendimiento, 1 AZ) · spread (7 por AZ) · partition (7 particiones por AZ). → [[placement-groups]]
- **Red:** [[enhanced-networking]] (SR-IOV, ENA 100 Gbps, EFA para HPC) · [[ebs-optimized]] (red dedicada para EBS, por defecto).
- **Status checks:** *System* (el host) vs *Instance* (adentro). Auto-recovery mueve a otro host → **se pierde el instance store**. → [[EC2]]
- **Escalado:** vertical = downtime + techo; horizontal = necesita app [[stateless]] + load balancer. → [[horizontal-vs-vertical-scaling]]

## Los números de memoria

| Dato | Valor |
|---|---|
| gp2 / gp3 — [[iops\|IOPS]] máx. | **16.000** |
| io1 / io2 — IOPS máx. | **64.000** |
| io2 Block Express — IOPS máx. | **256.000** |
| Tope por instancia (RAID 0 + EBS) | **~260.000** |
| gp2 — baseline | **3 IOPS/GB** (mín. 100), burst **3.000** |
| gp2 — balde de créditos | **5,4 millones** (30 min a full burst) |
| gp2 — dónde deja de importar el crédito | **1 TB** |
| gp2 vs gp3 — [[throughput\|throughput]] máx. | **250** vs **1.000 MB/s** |
| gp3 — base fija | **3.000 IOPS + 125 MB/s** |
| Tamaño de bloque | **16 KB** (SSD) · **1 MB** (HDD) |
| st1 / sc1 — máximo | 500 IOPS (500 MB/s) · 250 IOPS (250 MB/s) |
| Ratio IOPS/GB | io1 **50** · io2 **500** · Block Express **1.000** |
| FSR por región | **50** (cada par snapshot + AZ cuenta uno) |
| Reserved — plazos | **1 o 3 años** |
| Savings Plans — ahorro | Compute **66%** · EC2 **72%** |
| IP del IMDS | **169.254.169.254** |
| User data — tamaño máximo | **16 KB** |
| IMDSv2 — TTL máx. del token / hop limit | **6 h** (21.600 s) / **1** |
| Parameter Store Standard | **10.000** parámetros, **4 KB** (gratis) |
| Parameter Store Advanced | **8 KB** + parameter policies |
| Spread placement group | **7 instancias por AZ** |
| Partition placement group | **7 particiones por AZ** |
| Cluster PG — single-stream | **10 Gbps** (5 Gbps fuera) |
| ENA / Intel 82599 VF | **100** / 10 Gbps |

## Escenario → respuesta

| El escenario dice… | La respuesta es |
|---|---|
| Los datos deben sobrevivir a un stop/start | **EBS** |
| Caché o scratch ultrarrápido, datos reconstruibles | **Instance Store** |
| Más de 260.000 IOPS | **Instance Store** (EBS no llega) |
| Volumen **chico** con muchísimas IOPS | **io1 / io2** |
| Mismo rendimiento que gp2, más barato | **gp3** |
| El volumen "se puso lento" después de un rato | gp2 con [[burst-credit\|créditos]] agotados |
| Secuencial, logs, big data, barato | **st1** |
| Datos fríos, lo más barato de EBS | **sc1** |
| Bootear | **Cualquier SSD**; nunca st1/sc1 |
| Proteger un volumen contra la caída de su AZ | **Snapshot** (va a S3) |
| Mover un volumen a otra AZ | Snapshot → crear volumen allá |
| Cifrar un volumen que **ya existe** | Snapshot → **copiar con encryption** → volumen nuevo |
| Volumen restaurado que rinde poco al principio | **[[lazy-restore]]** → FSR o forzar lectura con `dd` |
| Aprovisioné IOPS pero no las alcanzo | Tope de la **instancia** ([[ebs-optimized]]) |
| Batch tolerante a interrupciones, mínimo costo | **Spot** |
| Uso constante 24/7 por años | **Reserved** (All Upfront 3 años = máximo descuento) |
| Descuento que cubra EC2 **y Lambda/Fargate** | **Compute Savings Plan** |
| **Garantizar** que voy a poder lanzar | Reserva **zonal** u **On-Demand Capacity Reservation** |
| Licencias por socket o core físico | **Dedicated Host** |
| Compliance: no compartir hardware | **Dedicated Instances** |
| Credenciales para el SDK en la instancia | **IMDS** + [[instance-profile\|instance profile]] |
| Proteger esas credenciales de un SSRF | **IMDSv2** (`HttpTokens: required`) |
| Usuarios que se deslogean al escalar | Sesiones **off-host** ([[stateless]]) |
| Acceso admin sin abrir puertos ni llaves | **SSM Session Manager** |
| Instancia lista para servir en el menor tiempo | **AMI baking** (+ user data para lo variable) |
| Cambié el user data y reinicié: no pasó nada | Solo corre en el **primer launch** |
| `running`, 2/2 checks, pero la app no responde | **User data fallido**: EC2 no lo valida |
| Contraseña de la DB para la instancia | **Parameter Store SecureString** / Secrets Manager + instance role, **no** user data |
| Rotación automática de credenciales de RDS | **Secrets Manager** ([[parameter-store-vs-secrets-manager]]) |
| `AccessDenied` con `--with-decryption` | Falta **`kms:Decrypt`** sobre la key |
| Métrica de **memoria** o **disco** de la instancia | **CloudWatch Agent** + rol `CloudWatchAgentServerPolicy` |
| Mínima latencia entre nodos ([[hpc\|HPC]]) | **Cluster PG** + enhanced networking / **EFA** |
| Pocas instancias críticas que no caigan juntas | **Spread PG** |
| HDFS / HBase / Cassandra con cientos de nodos | **Partition PG** |

## Los errores que más se repiten

1. Confundir **reboot** con **stop + start**: el reboot conserva el instance store y la IP pública; el stop + start no.
2. Creer que **EBS es resiliente de región**: es de **AZ**. Lo de región son los snapshots, porque viven en S3.
3. Suponer que **detener la instancia deja de costar**: el volumen EBS se sigue facturando, y desde 2024 también las IPv4 públicas.
4. Ofrecer **st1/sc1 como boot volume**: los HDD no botean.
5. Pensar que se puede **cifrar un volumen existente** en el lugar: hay que pasar por snapshot + copia cifrada.
6. Creer que un **Savings Plan o una reserva regional garantizan capacidad**: solo dan descuento.
7. Tomar el **precio máximo de Spot** como lo que se paga: es un techo.
8. Querer **agregar instance store** a una instancia ya lanzada: solo se adjunta al lanzar.
9. Pensar que el **user data se re-ejecuta** en cada reboot: corre solo en el primer launch.
10. Elegir **Parameter Store** cuando el enunciado pide **rotación automática**: eso es Secrets Manager.
11. Poner un **cluster PG en varias AZs**, o un **spread PG con más de 7 instancias por AZ**: ninguno de los dos se puede.

## Ver también

[[EC2]] · [[EBS]] · [[ebs-volume-types]] · [[instance-store-vs-ebs]] · [[ec2-purchase-options]] · [[ec2-bootstrapping]] · [[placement-groups]] · [[SSMParameterStore]] · [[vpc-cheat-sheet]]
