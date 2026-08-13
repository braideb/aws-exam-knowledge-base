---
title: AWS Global Infrastructure
category: concept
tags: [fundamentos, regions, availability-zones, edge-locations, resiliencia, networking]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.01 Servicios públicos vs. privados.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.02 AWS Global Infrastructure.md", "raw/notas curso mejorado/01 Fundamentos de AWS/01.03 Availability Zones (AZ).md"]
updated: 2026-07-23
---

# AWS Global Infrastructure

## Definición

La infraestructura global de AWS se compone de **Regions** (áreas geográficas con infraestructura completa), **Availability Zones** (entornos aislados dentro de una region) y **[[edge-location|Edge Locations]]** (puntos de presencia pequeños con CDN, mucho más numerosos). Sobre esta base se define la resiliencia de todo servicio AWS.

## Cómo aplica en AWS

### Servicios públicos vs. privados (perspectiva de RED, no de permisos)

- **Servicio público**: accesible desde un public endpoint (ej: [[S3]]). Que sea "público" por red no significa que los datos lo sean — alcanzabilidad y autorización son capas independientes.
- **Servicio privado**: corre dentro de una [[VPC]] (ej: [[EC2]]). Solo lo que está dentro o conectado a esa VPC accede.

Las tres zonas de red:

| Zona | Qué contiene |
|---|---|
| **Public Internet Zone** | Internet: clientes, sitios web |
| **AWS Public Zone** | Servicios con endpoint público (S3). Internet es el tránsito |
| **AWS Private Zone** | Las VPCs, aisladas salvo configuración explícita |

![[Pasted image 20260614003358.png]]

### Regions

- Cuando usás un servicio, lo usás **en una region específica**.
- Separación geográfica (desastres), **[[data-sovereignty|soberanía de datos]]** (los datos no salen de la region salvo config explícita), control de latencia/compliance.
- **Region code** (`ap-southeast-2`, para CLI/SDK/ARNs/CloudFormation) vs. **region name** (Asia Pacific (Sydney), para consola/docs). Formato del code: `{geografía}-{dirección}-{número}` (`us`, `eu`, `ap`=Asia Pacific, `sa`=South America, `ca`, `me`, `af`).

> **`us-east-1` (N. Virginia) es especial — cae en el examen:**
> - Los **eventos de servicios globales** (IAM, STS, CloudFront, Route 53) se registran acá en [[CloudTrail]].
> - Operaciones globales van contra `us-east-1`: crear buckets sin region, algunas APIs de billing y —el clásico— los **certificados de ACM para CloudFront deben emitirse en `us-east-1`** aunque tu infra esté en otra region.
> - Suele ser la **más barata**, donde los servicios nuevos **aparecen primero**, y la de **más historial de outages** (por ser la más usada).

![[Pasted image 20260614005731.png]]

### Availability Zones

- Múltiples por region (2, 3, 4… hasta 6; la mayoría **3**), aisladas entre sí a nivel de energía, red e instalaciones. Conectadas con enlaces high-speed low-latency (**~1 ms** entre AZs vs. ~100 ms entre regiones → permite replicación **síncrona**).
- Una AZ **no es un datacenter**: es **uno o más DCs** cercanos con dominio de falla independiente.
- Patrón de arquitectura: **repartir componentes entre AZs** (6 VMs / 3 AZs = 2 por AZ).

**Dimensionamiento N+1 a nivel de AZ (trampa clásica):** repartir no es sobrevivir. Si distribuís 6 VMs en 3 AZs (2 c/u) y se cae una AZ, te quedan **4** — insuficiente si necesitabas 6. Hay que dimensionar para que las AZs restantes absorban el 100%:
- **2 AZs** → cada una soporta el **100%** (total desplegado 200%).
- **3 AZs** → cada una el **50%** (total 150%) → **3 AZs sale más barato que 2** para la misma resiliencia (argumento de preguntas de costo).

**Costo cross-AZ:** tráfico por **IP privada dentro de la misma AZ = gratis**; entre AZs distintas AWS cobra por GB **en ambas direcciones** (envío y recepción). Tensión de diseño: más AZs = más resiliencia pero más costo/latencia cross-AZ.

![[Pasted image 20260614005711.png]]

![[Pasted image 20260614005941.png]]

### Edge Locations

Puntos de presencia pequeños (**400+ PoPs**, muchos más que regiones) cerca de los usuarios. Corren: CDN ([[CloudFront]]), **Lambda@Edge / CloudFront Functions** (cómputo en el borde), **S3 Transfer Acceleration**, **[[Route53]]** y **Global Accelerator** (anycast). Regla mental: **Region = infraestructura completa; Edge = cache + poco cómputo cerca del usuario**. El tráfico de salida por CloudFront además es **más barato** que el directo de S3/EC2.

## Niveles de resiliencia (clave para el examen)

| Nivel | Significa | Ejemplos |
|---|---|---|
| **[[globally-resilient|Globally Resilient]]** | La caída de una region no lo afecta; datos replicados entre regiones | [[IAM]], [[Route53]], CloudFront |
| **[[region-resilient|Region Resilient]]** | Replica entre AZs; cae la region → cae el servicio | [[S3]], DynamoDB, [[VPC]], ELB, RDS [[multi-az\|Multi-AZ]] |
| **[[az-resilient|AZ Resilient]]** | Vive en una sola AZ | instancia [[EC2]], volumen EBS, subnet, RDS single-AZ, NAT Gateway |

## Preguntas de examen frecuentes

- "¿Sobrevive a la caída de una AZ?" → distribuir en múltiples AZs (Multi-AZ).
- "¿Sobrevive a la caída de una region?" → replicación cross-region (CRR de S3, backups en otra region).
- Los servicios AZ-resilient (EC2, EBS) son siempre el punto débil a redundar ([[single-point-of-failure]]).
- No confundir "servicio público" (red) con "datos públicos" (permisos).
