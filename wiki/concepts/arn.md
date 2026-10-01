---
title: ARN (Amazon Resource Name)
category: concept
tags: [iam, arn, identificadores, policies]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/03 IAM ACCOUNTS y AWS Organization/03.03 ARN (Amazon Resource Name).md", "raw/doc oficial/Identify AWS resources with Amazon Resource Names (ARNs) - AWS Identity and Access Management.md"]
updated: 2026-10-01
---

# ARN — Amazon Resource Name

## Definición

Identificador **único y global** de cualquier recurso AWS, propio o de otra cuenta. Es como se referencian recursos en las [[iam-policy-evaluation|policies]].

## Formato

```
arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id
```

Que existan **tres variantes** no es capricho: cada servicio eligió su separador (`/`, `:`, o acepta ambos). **No hay regla para deducirlo** — se consulta en la doc del servicio.

| Campo | Ejemplo | Nota |
|---|---|---|
| `partition` | `aws` | Otras: `aws-cn`, `aws-us-gov` |
| `service` | `s3`, `ec2`, `iam` | |
| `region` | `us-east-1` | **Vacío** en servicios globales (S3, IAM) |
| `account-id` | `123456789012` | **Vacío** en S3 (nombres ya globales) |
| `resource-id` | `catgifs` | |

Los `:::` seguidos no son un typo: son campos vacíos.

## El detalle que cae en el examen

```
arn:aws:s3:::catgifs      ← el BUCKET     (s3:ListBucket)
arn:aws:s3:::catgifs/*    ← los OBJETOS   (s3:GetObject)
```

**Uno no incluye al otro.** Para acceso completo se necesitan **los dos ARNs**. Es el error clásico de "la policy no funciona".

> Analogía: el bucket es el edificio; `/*` son los departamentos. La llave del portón no abre los departamentos.

## Wildcards — donde se cuela el bug de seguridad

- `*` (varios caracteres) y `?` (uno solo) dentro del ARN: `arn:aws:s3:::logs-*`.
- No se permite wildcard en `partition` ni en `resource-type`.

**La diferencia es una barra, y cambia todo:**

```
arn:aws:s3:::catgifs/*   → los objetos de catgifs. Correcto.
arn:aws:s3:::catgifs*    → catgifs, catgifs-backup, catgifs-produccion…
                           y cualquier bucket que alguien cree mañana con ese prefijo.
```

La segunda versión otorga acceso a **buckets que todavía no existen**. Regla práctica: al escribir un `Resource`, leerlo preguntándose **"¿qué otra cosa podría matchear esto?"**.

## Dónde aparecen los ARNs

No son solo cosa de policies:

- El campo `Resource` de identity y resource policies.
- **Referencias entre servicios**: el ARN del rol que ejecuta una Lambda, el del topic SNS al que notifica una alarma de [[CloudWatch]].
- Salidas de [[CloudFormation]] (`!GetAtt MiRecurso.Arn`).
- Comandos de la CLI que identifican un recurso de otra cuenta.

Por eso conviene leerlos de un vistazo: un ARN te dice servicio, region, cuenta dueña y tipo de recurso sin buscar nada.

## Ejemplos por servicio

```
arn:aws:ec2:us-east-1:123456789012:instance/i-0abc123
arn:aws:lambda:us-east-1:123456789012:function:miFuncion
arn:aws:iam::123456789012:role/Admin          ← IAM: sin region
arn:aws:iam::123456789012:root                ← "la cuenta entera" (trust/key policies)
```

## Preguntas de examen frecuentes

- Policy de S3 con un solo ARN que "no funciona" → falta el par bucket + `/*`.
- `arn:aws:iam::…:root` en un `Principal` = **la cuenta**, no el root user literal. En una [[trust-policy]] significa "delego en esa cuenta la decisión de qué identidades suyas pueden asumir el rol".
- `bucket*` en vez de `bucket/*` → acceso a buckets no previstos. Es el error de wildcard que más se cuela en un code review.
