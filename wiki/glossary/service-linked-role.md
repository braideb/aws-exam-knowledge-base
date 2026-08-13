---
title: Service-Linked Role
category: glossary
tags: [iam, roles, seguridad, servicios]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: []
updated: 2026-07-25
---

# Service-Linked Role

> **En una línea:** un rol que **AWS predefine y mantiene** para un servicio, y que vos no podés editar.

## Definición

Para servicios que necesitan una lista larga y específica de permisos (Auto Scaling tiene que lanzar instancias, engancharlas a un ELB, leer métricas…), AWS provee el rol ya armado: *"yo ya sé qué necesito, no lo edites"*.

Se reconocen por el path reservado **`/aws-service-role/`** y el prefijo de nombre **`AWSServiceRoleFor…`**:

```
arn:aws:iam::123456789012:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling
```

Los usan Auto Scaling, ELB, RDS, EKS, [[Organizations]], GuardDuty, Config, Trusted Advisor.

## Dónde aparece

- [[IAM]] — tabla comparativa completa contra un service role normal
- [[permissions-boundary]] — no se le puede aplicar boundary a un service-linked role

## Dato de examen

- **No se puede borrar mientras el servicio lo esté usando** — es la diferencia que más se pregunta. Un rol normal lo borrás cuando querés (y rompés todo); acá AWS te protege de vos mismo.
- Tampoco se puede editar su permissions policy ni su trust policy. La ventaja: **AWS lo actualiza solo** cuando el servicio incorpora funciones nuevas.
- Los **SCPs no afectan a los service-linked roles** ([[Organizations]]).
- Permiso para crearlos: `iam:CreateServiceLinkedRole` con condition `iam:AWSServiceName`. ⚠️ El nombre del servicio **varía y es case sensitive** — se busca en la doc, no se adivina.

## Ver también

[[trust-policy]] · [[instance-profile]] · [[permissions-boundary]]
