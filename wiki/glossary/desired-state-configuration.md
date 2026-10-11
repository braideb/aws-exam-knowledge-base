---
title: Desired State Configuration
category: glossary
tags: [cloudformation, cfn-init, configuration-management, idempotency]
exam: [DVA-C02, SAA-C03, DOP-C02]
sources: ["raw/notas curso mejorado/12 CloudFormation/12.09 cfn-init (CloudFormation Init).md"]
updated: 2026-10-10
---

# Desired State Configuration

> **En una línea:** declarás **el QUÉ** (el estado final que querés) y el sistema averigua **el CÓMO** llegar.

## Definición

Enfoque de gestión de configuración opuesto al **procedural**. En vez de dictar comandos paso a paso (user data), describís el estado deseado —"el paquete httpd instalado, este archivo con este contenido, el servicio corriendo"— y la herramienta reconcilia la realidad con esa declaración. Es el modelo de Ansible, Puppet o Chef, y el de **cfn-init** dentro de CloudFormation.

Dos propiedades se desprenden del enfoque: es **[[idempotency|idempotente]]** (si ya está en el estado deseado, no hace nada) y suele ser **multiplataforma** (la misma declaración aplica en distintos SO).

## Dónde aparece

- [[cfn-init-and-signals]] — `cfn-init` aplica la metadata `AWS::CloudFormation::Init` como desired state
- [[wiki/services/CloudFormation]] — contraste con el user data procedural
- También en: [[ec2-bootstrapping]]

## Dato de examen

- Enunciado *"configuración declarativa, idempotente y multiplataforma gestionada desde CloudFormation"* → **cfn-init** (desired state), **no** user data (procedural).

## Ver también

[[idempotency]] · [[cloud-init]]
