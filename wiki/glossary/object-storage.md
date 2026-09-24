---
title: Object Storage
category: glossary
tags: [s3, storage, almacenamiento]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/04 S3/04.09 S3 Lifecycle Configuration.md", "raw/doc oficial/What is Amazon S3 - Amazon Simple Storage Service.md", "raw/doc oficial/Transitioning objects using Amazon S3 Lifecycle - Amazon Simple Storage Service.md"]
updated: 2026-09-24
---

# Object Storage

> **En una línea:** guardás **objetos completos vía API**, no archivos en un disco montado.

## Definición

Modelo de almacenamiento donde la unidad es el **objeto** (key + value + version ID + metadata) dentro de un contenedor plano (**bucket**). No hay jerarquía real de carpetas: `old/koala.jpg` es una key con barras, y las "carpetas" son un [[prefix|prefijo]] que la consola dibuja.

Los tres modelos que el examen contrasta:

| Modelo | Unidad | Ejemplo AWS | Se monta |
|---|---|---|---|
| **Object** | Objeto vía API | [[S3]] | ❌ |
| **Block** | Bloques de un volumen | EBS, Instance Store | ✅ como disco |
| **File** | Archivos en un share | EFS, FSx | ✅ como filesystem de red |

## Dónde aparece

- [[S3]] — "no es file store ni block store: no se monta como disco"
- [[EC2]] — Instance Store y EBS como block storage
- [[s3-storage-classes]] — la dimensión económica del object storage
- [[storage-types]] — object vs block vs file, y por qué solo block bootea
- También en: [[delete-marker]] · [[etag]] · [[multipart-upload]] · [[prefix]]

## Dato de examen

- **"Montar S3 como filesystem compartido" → ❌**, eso es **EFS**. Es el distractor más repetido.
- Escala de cero a ilimitado, pero: objeto máx **5 TB**, single PUT máx **5 GB** (más → [[multipart-upload]]).
- Se accede por API/HTTP, así que el rendimiento se escala repartiendo por [[prefix|prefijos]], no agrandando un disco.

## Ver también

[[durability]] · [[availability]] · [[multipart-upload]] · [[prefix]]
