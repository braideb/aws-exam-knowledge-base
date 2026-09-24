---
title: ALIAS vs CNAME
category: comparison
tags: [route53, dns, alias, cname, apex]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/01 Fundamentos de AWS/01.15 DNS Record Types.md", "raw/doc oficial/Choosing between alias and non-alias records - Amazon Route 53.md"]
updated: 2026-09-24
---

# Comparativa: ALIAS vs CNAME en Route 53

## Tabla comparativa

| | **CNAME** (estándar DNS) | **ALIAS** (exclusivo R53) |
|---|---|---|
| Mapea | host → **otro host** | host → **recurso AWS** (o record de la zone) |
| ¿En el [[zone-apex\|apex]] (`dominio.com`)? | ❌ **Prohibido** | ✅ Sí |
| ¿A una IP? | ❌ (eso es un A record) | Se comporta como A/AAAA para el cliente |
| Costo de queries | Se cobra | **Gratis** hacia recursos AWS |
| [[ttl\|TTL]] | Configurable | **No** configurable (lo maneja AWS) |
| Destinos típicos | Cualquier hostname | ELB, [[CloudFront]], [[S3]] website, API Gateway, Global Accelerator, otra hosted zone |
| Qué ve el cliente | Un CNAME → resolución extra | Un A/AAAA con la IP (AWS la actualiza si cambia) |
| Health checks / routing policies | ✅ Sí | ✅ Sí |

## Cuándo usar cada uno

- **Apuntar el apex** (`midominio.com` a secas) a un ELB/CloudFront/S3 → **ALIAS**, única opción.
- Alias de un host a otro **fuera de AWS** → CNAME clásico.
- Subdominios hacia recursos AWS → ALIAS también conviene (gratis y se actualiza solo si cambia la IP del recurso).

## Trampa típica del examen

- "CNAME en el zone apex" → siempre inválido; la respuesta correcta incluye **ALIAS record**.
- Un CNAME **no puede apuntar a una IP**.
- Detalle de la doc: un CNAME que encadena a otro record se factura como **dos** queries — el ALIAS a recurso AWS es gratis.
