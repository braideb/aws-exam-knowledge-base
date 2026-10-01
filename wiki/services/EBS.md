---
title: EBS (Elastic Block Store)
category: service
tags: [ebs, storage, ec2, snapshots, kms, cifrado, iops]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.05 Elastic Block Store (EBS) — Basics.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.11 EBS Snapshots, Restore y Fast Snapshot Restore (FSR).md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.13 EBS Encryption.md", "raw/notas curso mejorado/06 Elastic Compute Cloud (EC2)/06.12 Demostración - EBS Volumes e Instance Store.md", "raw/notas curso mejorado/09 Advanced EC2/09.13 EBS Optimized.md"]
updated: 2026-09-30
---

# EBS — Elastic Block Store

## ¿Qué es?

Servicio de **block storage por red** para [[EC2]]: toma discos físicos en crudo y te presenta **volúmenes** sobre los que el sistema operativo crea su file system. Es el almacenamiento **persistente** por defecto de una instancia — el disco donde vive el SO y los datos que importan.

A diferencia del instance store, el volumen **no está atado al ciclo de vida de la instancia**: se puede separar (`detach`) y volver a conectar (`attach`) a otra, y los datos siguen ahí.

## Casos de uso

- **Boot volume** de casi cualquier instancia EC2.
- Bases de datos autogestionadas, donde importa la latencia consistente.
- Datos que tienen que sobrevivir a un stop/start o a la terminación de la instancia.
- Backups a nivel de disco vía snapshots, y migración de datos entre AZs o regiones.

## Características clave

- **[[az-resilient|AZ-resilient]]**, no region-resilient: el volumen se aprovisiona **dentro de una AZ**. Si se cae la AZ, el volumen se ve afectado. La protección contra eso son los **snapshots**, que van a [[S3]].
- Un volumen se adjunta a **una** instancia (caso normal) y **no cruza de AZ**: no podés conectar un volumen de AZ-A a una instancia de AZ-B.
- Se factura por **GB-mes** aprovisionado, corra o no la instancia. Un volumen adjunto a una instancia detenida **se sigue cobrando**.
- Opcionalmente **cifrado** con [[KMS]].

### Tipos de volumen

Seis tipos vigentes en tres familias. La tabla completa y los criterios están en **[[ebs-volume-types]]**:

| Familia | Tipos | En una línea |
|---|---|---|
| SSD propósito general | `gp2`, `gp3` | El default; gp3 da 3.000 [[iops\|IOPS]] fijas y es más barato |
| SSD IOPS aprovisionadas | `io1`, `io2`, `io2 Block Express` | IOPS independientes del tamaño, latencia consistente |
| HDD | `st1`, `sc1` | Secuencial y barato; **no botean** |

### Snapshots

Copias de seguridad de un volumen **hacia S3**, lo que las vuelve **resilientes a nivel de región**. Son el mecanismo para sobrevivir a la caída de una AZ y para mover datos entre AZs.

Son **incrementales**:

- La **primera** snapshot es una copia completa de los **datos usados** del volumen.
- Cada snapshot siguiente captura **solo los bloques que cambiaron**.

Desde una snapshot podés restaurar en la misma AZ, hacer un **cross-AZ restore**, o copiarla a otra región y restaurar allá.

**Facturación:** por **GB-mes**, y sobre **datos usados, no asignados**. Un volumen de 40 GB con 10 GB escritos produce un primer snapshot de ~10 GB — EBS **no cobra el espacio vacío**.

### Restore y el bajón de rendimiento

- Un volumen creado **en blanco** rinde al máximo **de inmediato**.
- Un volumen creado **desde un snapshot** sufre [[lazy-restore|lazy restore]]: está disponible al instante pero los bloques se traen de S3 en segundo plano. Leer un bloque que todavía no llegó funciona, pero **con latencia mucho más alta**.

Dos formas de evitarlo:

1. **Forzar la lectura de todos los bloques** antes de poner el volumen en producción (`dd` en Linux). Gratis, pero lleva tiempo.
2. **Fast Snapshot Restore (FSR)**: se activa **sobre un snapshot** y hace la restauración instantánea. Cuesta extra.

> **Dato de examen (FSR):** cada combinación **snapshot + AZ** cuenta como un FSR, y el límite es de **50 por región**. Un snapshot habilitado en 4 AZs consume **4** de esos 50.

### Cifrado

EBS **no cifra nada por defecto**: lo que escribe el OS queda en texto plano en el disco físico. El cifrado de EBS da [[encryption-at-rest|encryption at rest]] para **volúmenes y snapshots**, usando [[envelope-encryption|envelope encryption]] con [[KMS]]:

1. Al crear el volumen, EBS le pide a KMS una **[[data-encryption-key|DEK]] única para ese volumen** (`GenerateDataKeyWithoutPlaintext`: KMS devuelve **solo la versión cifrada**). Esa DEK cifrada se guarda junto al volumen.
2. Al usarlo, EBS pide a KMS que descifre la DEK. La clave en texto plano se carga **en la memoria del EC2 host** — nunca en disco — y el host cifra y descifra el tráfico entre la instancia y el almacenamiento.
3. Al mover la instancia de host, la DEK en texto plano **se descarta** y hay que volver a pedirla.

Consecuencias que caen en el examen:

- Un **snapshot de un volumen cifrado usa la misma DEK**, así que el snapshot queda cifrado, y todos los volúmenes creados desde él también. Un volumen **nuevo desde cero** recibe una DEK propia.
- **No se puede "descifrar" un volumen**, ni cifrar directamente uno existente. El camino es: **snapshot → copiar el snapshot con encryption activado → crear un volumen desde esa copia**.
- **El OS no se entera**: todo ocurre en el host, entre el host y EBS → **sin pérdida de rendimiento**. El algoritmo es **AES-256**.
- Se puede activar el **cifrado por defecto a nivel de cuenta**, eligiendo la KMS key.

## Integración con otros servicios

- [[EC2]] — el volumen se adjunta a una instancia; el ancho de banda hacia EBS depende del tipo de instancia ([[ebs-optimized]]). EBS optimized le da a EBS **capacidad de red dedicada**, separada del tráfico de datos. Hoy viene habilitado por defecto y sin costo, y es necesario para sacarle las IOPS prometidas a gp2/io1.
- [[KMS]] — la KMS key que protege la DEK de cada volumen.
- [[S3]] — donde viven los snapshots (y de ahí su resiliencia de región).
- [[CloudWatch]] — métricas de IOPS, [[throughput]] y **balance de [[burst-credit|créditos de burst]]** de gp2.

## Gotchas y trampas del examen

- **EBS es AZ-resilient.** "Proteger los datos contra la caída de una AZ" → **snapshots** (que van a S3), no "EBS es resiliente".
- Un volumen y una instancia **no cruzan de AZ**. Para mover un volumen de AZ: snapshot → crear volumen en la otra AZ. Para cambiar de región, además hay que **copiar el snapshot**.
- Se cobra por **GB aprovisionado**, aunque la instancia esté **detenida**. Un stop no frena el costo del disco.
- Los snapshots se cobran por **datos usados**, no por el tamaño del volumen.
- "El volumen restaurado anda lento al principio" → **lazy restore**, no un problema del tipo de volumen.
- No se puede quitar el cifrado de un volumen cifrado, ni cifrar uno existente en el lugar.
- Borrar una **AMI no borra sus snapshots** — te los siguen cobrando ([[EC2]]).

## Demos del curso

- [EBS Volumes — Parte 1](https://learn.cantrill.io/courses/1101194/lectures/27806446)
- [EBS Volumes — Parte 2](https://learn.cantrill.io/courses/1101194/lectures/28705524)
- [EBS Volumes — Parte 3](https://learn.cantrill.io/courses/1101194/lectures/28705526)

## Ver también

[[ebs-volume-types]] · [[instance-store-vs-ebs]] · [[storage-types]] · [[EC2]] · [[KMS]]
