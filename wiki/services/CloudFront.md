---
title: CloudFront
category: service
tags: [cloudfront, cdn, edge, oac, signed-urls, https]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/doc oficial/Restrict access to an Amazon S3 origin.md"]
updated: 2026-09-24
---

# CloudFront

## ¿Qué es?

El **CDN** de AWS: cachea y sirve contenido desde las **[[edge-location|edge locations]]** ([[global-infrastructure]]) cerca de los usuarios. Servicio **global** (sus eventos de API se registran en `us-east-1` — ver [[CloudTrail]]).

## Casos de uso

- Acelerar la entrega de contenido estático/dinámico a usuarios globales.
- Dar **HTTPS y dominio propio** a un sitio estático en [[S3]] (los website endpoints de S3 son solo HTTP).
- Distribuir **contenido privado** a escala (signed URLs/cookies).

## Características clave

### Origin Access Control (OAC)

Para que el bucket S3 de origen **no sea público** pero CloudFront sí pueda leerlo:

1. El bucket queda privado (Block Public Access on).
2. Se crea un **OAC** en la distribución.
3. La bucket policy permite al [[principal]] de CloudFront (`cloudfront.amazonaws.com`) con condición `AWS:SourceArn` = el ARN de la distribución (evita el [[confused-deputy|confused deputy]]).

> **OAC** es el mecanismo actual; **OAI** es el legacy que aparece en material viejo. Diferencias que caen en el examen: solo OAC soporta **PUT/DELETE dinámicos**, **SSE-KMS** y todas las regiones (incluidas opt-in). Si el origen usa SSE-KMS, la **key policy** debe permitir a `cloudfront.amazonaws.com` (`kms:Decrypt`, `kms:Encrypt`, `kms:GenerateDataKey*`) con condición `AWS:SourceArn` = ARN de la distribución (ver [[s3-encryption]]).
>
> Gotcha: un **S3 website endpoint no puede usar OAC ni OAI** — CloudFront lo trata como custom origin (el website endpoint es HTTP puro y anónimo). OAC solo aplica al REST endpoint del bucket.

### Contenido privado: Signed URLs y Signed Cookies

| | S3 [[presigned-url\|Presigned URL]] | CloudFront Signed URL / Cookie |
|---|---|---|
| Alcance | **Un** objeto de S3 | Contenido detrás del CDN (cualquier origen) |
| Muchos archivos | No | **Signed Cookie** cubre varios |
| Cache/escala | No cachea | Cachea en edge |

Regla: acceso puntual a un objeto → [[presigned-url|presigned URL]] de [[S3]]; contenido premium distribuido por CDN → CloudFront signed URLs/cookies.

## Integración con otros servicios

- [[S3]] — origen típico; OAC para mantenerlo privado.
- [[Route53]] — alias records hacia la distribución (funciona en el apex).
- ACM — certificados TLS para dominio propio.

## Gotchas y trampas del examen

- "Sitio S3 con HTTPS y dominio custom" → **S3 + CloudFront + ACM + Route53 alias** (S3 website solo hace HTTP).
- "El bucket no debe ser público pero el sitio sí" → **OAC** + bucket policy.
- No confundir CloudFront (CDN, cachea) con S3 Transfer Acceleration (acelera **subidas**, no cachea).

> ⚠️ Página inicial basada en clippings — se ampliará cuando el curso cubra CloudFront (behaviors, TTLs, invalidations, Lambda@Edge).
