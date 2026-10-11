---
title: PassRole (iam:PassRole)
category: glossary
tags: [iam, cloudformation, permissions, security]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/12 CloudFormation/12.16 Stack Roles.md"]
updated: 2026-10-10
---

# PassRole (iam:PassRole)

> **En una línea:** permiso que deja a una identidad **entregarle un [[IAM]] role a un servicio** para que el servicio actúe con los permisos de ese role, sin que la identidad pueda usarlos ella misma.

## Definición

`iam:PassRole` es un permiso de control: autoriza a *pasar* un role a un servicio de AWS (CloudFormation, EC2, Lambda…) al crear o configurar un recurso. El servicio **asume** el role y de ahí saca sus permisos. La identidad que lo pasa **no** obtiene esos permisos para sí: solo puede entregárselos al servicio.

Es la pieza que habilita la **separación de responsabilidades**: quien opera un recurso necesita permisos limitados + `PassRole`, en vez de todos los permisos de los recursos que el servicio va a crear.

## Dónde aparece

- [[wiki/services/CloudFormation]] — **stack roles**: el operador pasa un role y CFN lo asume para crear/actualizar/eliminar
- [[IAM]] — patrón general de delegación a servicios
- También en: [[cfn-init-and-signals]] (instance roles vía [[instance-profile]])

## Dato de examen

- Escenario *"un operador debe desplegar una stack sin tener permiso para crear sus recursos por fuera de CloudFormation"* → darle permisos sobre la stack **+ `iam:PassRole`** de un role que preparó un admin. El operador no puede editar ni asumir ese role directamente.

## Ver también

[[trust-policy]] · [[least-privilege]] · [[confused-deputy]]
