---
title: "DVA-C02 · Dominio 3: Deployment"
category: domain
tags: [dva-c02, deployment, cicd, iac, cloudformation]
exam: [DVA-C02]
sources: ["https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html"]
updated: 2026-10-03
---

# DVA-C02 · Dominio 3 — Deployment

## Peso en el examen

**24%**

## Resumen de lo que el examen evalúa

Preparar artefactos y desplegarlos: entornos de dev/test, despliegues automatizados con CI/CD (CodePipeline/CodeBuild/CodeDeploy), IaC (CloudFormation/SAM), y estrategias de deployment (blue/green, canary, rolling).

## Task statements oficiales (exam guide)

**Task 1 — Preparar artefactos**: dependencias del paquete (env vars, config files, imágenes de contenedor), estructura de directorios, repos de código, requisitos de recursos (memoria, cores), configs por entorno (**AWS AppConfig**).

**Task 2 — Testear en entornos de desarrollo**: probar código desplegado con herramientas AWS, integration tests y mocks de APIs externas, endpoints de desarrollo (**stages de API Gateway**), desplegar actualizaciones de stack (SAM a staging), testear apps event-driven.

**Task 3 — Automatizar el testing de deployment**: test events (payloads JSON para Lambda/API GW/SAM), desplegar APIs a varios entornos, entornos con versiones aprobadas (**Lambda aliases**, tags de imágenes, Amplify branches), implementar y desplegar **IaC** (SAM, CloudFormation), gestión de entornos por servicio, Amazon Q Developer para generar tests.

**Task 4 — Desplegar con CI/CD**: opciones de packaging de Lambda, stages y custom domains de API Gateway, actualizar templates IaC, **estrategias de deployment (blue/green, canary, rolling)**, commit → build/test/deploy, workflows orquestados, **rollbacks**, labels/branches para versionado, configs dinámicas (staging variables de API GW en Lambda).

## Temas clave — cobertura actual

| Tema | Página | Estado |
|---|---|---|
| **IaC con CloudFormation** (Tasks 3/4): templates, stacks, lifecycle, StackSets, DeletionPolicy, stack roles, change sets, custom resources | [[wiki/services/CloudFormation]] | ✅ solo curso |
| Templates **portables**: parameters, pseudo parameters, Mappings, Conditions, Outputs, DependsOn | [[cloudformation-template]] | ✅ solo curso |
| Funciones intrínsecas (`!Ref`, `!GetAtt`, `!Sub`, `!ImportValue`…) | [[cloudformation-intrinsic-functions]] | ✅ solo curso |
| Configurar/validar el deploy en EC2: cfn-init (desired state), cfn-signal, cfn-hup, CreationPolicy | [[cfn-init-and-signals]] | ✅ solo curso |
| Reutilizar template vs reutilizar stack; **change sets** como preview antes de aplicar | [[nested-stacks-vs-cross-stack-references]], [[wiki/services/CloudFormation]] | ✅ solo curso |
| Hosting estático como target de deploy | [[S3]] (static hosting), [[CloudFront]] | ✅ parcial |
| **Artefactos inmutables: AMI baking y el AMI lifecycle** | [[golden-ami]], [[EC2]] | ✅ |
| **Bootstrapping con user data** vs baking, boot time to service time | [[ec2-bootstrapping]] | ✅ |
| Configuración por entorno fuera del código (configs dinámicas) | [[SSMParameterStore]] | ✅ solo curso |
| Dimensionar el entorno (memoria, cores) del Task 1 | [[ec2-instance-types]] | ✅ |
| Escalar el entorno desplegado: horizontal vs vertical | [[horizontal-vs-vertical-scaling]] | ✅ |
| Modelo de costos de los entornos (dev/test vs prod) | [[ec2-purchase-options]] | ✅ |
| **Imágenes de contenedor como artefacto** (Task 1): Dockerfile, layers, tags, registry | [[containers]], [[ECR]] | ✅ solo curso |
| Build + push de la image en CI/CD (CodeBuild con privileged mode), tag immutability | [[ECR]] | ✅ solo curso |
| **Estrategias de deployment** (Task 4): [[rolling-deployment\|rolling]] (`minimumHealthyPercent` / `maximumPercent`) y [[blue-green-deployment\|blue/green]] con CodeDeploy (canary / linear) | [[ECS]] | ✅ para ECS; falta Lambda/Beanstalk |
| Dónde corre el artefacto: EC2 mode vs Fargate | [[ecs-ec2-vs-fargate]] | ✅ |
| Entornos de test con datos reales sin copiar todo (Aurora **fast clone**) y bases de dev/test que se pausan solas (Serverless v2) | [[Aurora]] | ✅ |
| Migrar la base de un entorno (EC2 → RDS: dump/restore y cambio de endpoint) | [[databases-on-ec2]], [[RDS]] | ✅ solo curso |

## Servicios más importantes para este dominio

Presente en la wiki: [[wiki/services/CloudFormation]] (ahora en profundidad — ver [[cloudformation-template]], [[cloudformation-intrinsic-functions]], [[cfn-init-and-signals]], [[nested-stacks-vs-cross-stack-references]]), [[EC2]] y [[EBS]] como destino de despliegue, [[ECS]] y [[ECR]] para containers.

> **Lo que aporta el módulo 08 (containers):** el primer contacto real con las estrategias de deployment del Task 4. En [[ECS]] un deploy es **registrar una nueva revisión de la task definition y actualizar el service**. El tipo por defecto es [[rolling-deployment|rolling]], controlado por `minimumHealthyPercent` y `maximumPercent`. La alternativa es [[blue-green-deployment|blue/green]] con **CodeDeploy**, que mueve el tráfico del ALB todo de una vez, en **canary** o en **linear**. El artefacto es una image versionada por **tag** en [[ECR]].

> **El patrón que sí cubre la wiki hoy:** despliegue **inmutable** vía [[golden-ami|AMI baking]] — lanzar, configurar, hornear la AMI y relanzar desde ella. Su contraste de examen es configurar en boot con **user data** ([[ec2-bootstrapping]]): baking da arranque rápido y determinista (clave para autoscaling) pero exige rehornear ante cualquier cambio; user data es más flexible y más lento. Una AMI **no se edita** y es **regional**. Lo óptimo es **combinar**: hornear lo lento y dejar el user data para la configuración final, que reduce el **boot time to service time** sin perder flexibilidad.

> **Lo que aporta el módulo 12 (CloudFormation):** el dominio gana su pata de **IaC** completa. CFN deja de ser una página inicial y pasa a cubrir templates portables, funciones intrínsecas, la configuración de EC2 con cfn-init (desired state) vs user data (procedural), señales honestas del bootstrapping (cfn-signal/CreationPolicy), arquitecturas multi-stack (nested vs cross-stack), StackSets, DeletionPolicy, stack roles y **change sets** (preview de un update antes de aplicarlo, clave para evitar replacements destructivos).

> ⚠️ **Sigue siendo el dominio con huecos**, pero menos: resuelto CloudFormation, el curso **todavía no llegó** a **CodePipeline, CodeBuild, CodeDeploy** como servicios (hoy solo aparecen por su rol en ECS/ECR), **AWS SAM** (extensión de CFN para serverless), **Elastic Beanstalk**, **CDK**, estrategias de deployment fuera de ECS (all-at-once, immutable, traffic shifting de Lambda), versionado/aliases de Lambda. Prioridad alta cuando el curso avance — 24% del examen depende de esto.
