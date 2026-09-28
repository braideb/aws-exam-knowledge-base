---
title: Vendor lock-in
category: glossary
tags: [arquitectura, kubernetes, eks, multicloud]
exam: [SAA-C03, DVA-C02]
sources: ["raw/notas curso mejorado/08 Containers, ECS y ECR/08.10 Elastic Kubernetes Service (EKS) 101.md"]
updated: 2026-09-28
---

# Vendor lock-in

> **En una línea:** quedar **atado a un proveedor** porque la solución usa tecnología propia suya y salir cuesta caro.

## Definición

Lo contrario es una solución **cloud agnostic**, que corre igual en cualquier proveedor. Kubernetes es el ejemplo del curso: un cluster de [[EKS]] usa el mismo Kubernetes que Azure, GCP u on-premises. [[ECS]], en cambio, es propio de AWS.

## Dónde aparece

- [[EKS]] — el motivo principal para elegirlo frente a ECS

## Dato de examen

- Containers + "evitar vendor lock-in", "multicloud" o "cloud agnostic" → **EKS**.

## Ver también

[[containers]]
