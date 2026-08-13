# AWS Knowledge Base — Schema

## Propósito

Esta es una wiki personal para preparar tres certificaciones AWS:
- **DVA-C02** — AWS Certified Developer – Associate *(prioridad actual)*

El LLM mantiene y actualiza esta wiki. El humano aporta fuentes y hace preguntas.

---

## Estructura de carpetas

```
AWS/
├── CLAUDE.md              # Este archivo — reglas y convenciones
├── index.md               # Catálogo de todas las páginas wiki
├── log.md                 # Log cronológico append-only
├── guia-estudio.md        # Orden de aprendizaje del material ingestado — se reescribe en cada ingest
├── demos.md               # Índice de demos prácticas del curso, con link a la página wiki relacionada
├── raw/                   # Fuentes originales (solo lectura para el LLM)
│   ├── assets/            # Imágenes de las notas (los embeds ![[...]] de wiki/ resuelven acá)
│   ├── notas curso/       # Notas manuscritas crudas — NO procesar (ver regla abajo)
│   ├── notas curso mejorado/  # Notas del curso SEGMENTADAS: un archivo por sección (NN.MM) + índices por módulo — fuente válida de ingest
│   └── doc oficial/       # Clippings de documentación oficial AWS, curados
└── wiki/
    ├── services/          # Una página por servicio AWS (ej: CodePipeline.md)
    ├── concepts/          # Conceptos y patrones (ej: blue-green-deployment.md)
    ├── glossary/          # Un archivo por TÉRMINO — definición corta y atómica (ej: globally-resilient.md)
    ├── domains/           # Dominios del examen (ej: dva-development.md)
    ├── comparisons/       # Comparativas entre servicios/enfoques
    └── exams/             # Notas de examen, tips, preguntas de práctica
```

**Regla:** el LLM nunca modifica archivos en `raw/`. Todo lo que escribe va en `wiki/`.

**Regla:** el LLM **nunca procesa ni ingesta los archivos de `raw/notas curso/`**. Son notas manuscritas del humano, mal estructuradas y crudas. Existen solo como respaldo. Las fuentes válidas para procesar son `raw/notas curso mejorado/` (las versiones pasadas en limpio) y `raw/doc oficial/` (clippings curados), u otras que el humano indique explícitamente. Si el humano pide "ingestar" o "procesar" sin aclarar, se usa `notas curso mejorado/`, no `notas curso/`.

---

## Formato de páginas wiki

Cada página wiki sigue esta estructura mínima en YAML frontmatter + markdown:

```yaml
---
title: <Nombre de la página>
category: service | concept | domain | comparison | exam
tags: [tag1, tag2]
exam: [DOP-C02, SAA-C03, DVA-C02]   # exámenes donde aplica
sources: []                            # archivos en raw/ que alimentaron esta página
updated: YYYY-MM-DD
---
```

Luego el cuerpo en markdown con secciones según el tipo:

### Páginas de servicio (`wiki/services/`)
- **¿Qué es?** — descripción en 2-3 oraciones
- **Casos de uso** — cuándo usarlo (bullet points)
- **Características clave** — lo que debes memorizar para el examen
- **Integración con otros servicios** — links a otras páginas wiki
- **Gotchas y trampas del examen** — errores comunes, distractores

### Páginas de concepto (`wiki/concepts/`)
- **Definición** — qué es el concepto
- **Cómo aplica en AWS** — servicios involucrados con links
- **Patrones comunes** — diagramas en texto o tablas
- **Preguntas de examen frecuentes** sobre este concepto

### Páginas de glosario (`wiki/glossary/`)

Un archivo **por término**. Sirven para que cualquier pieza de jerga que aparezca en el resto de la wiki tenga un link a su definición, en vez de quedar suelta.

- **Qué entra:** jerga **atómica** que hoy vive suelta dentro de otras páginas — *globally resilient, envelope encryption, blast radius, zone apex, idempotency*…
- **Qué NO entra:** términos que ya tienen página propia. Si el término es un servicio ([[S3]]) o un concepto con página ([[arn]]), se linkea **esa** página; no se crea un stub duplicado en `glossary/`.
- **Criterio de corte con `concepts/`:** el glosario responde *"¿qué significa esta palabra?"* en una pantalla. `concepts/` desarrolla patrones, diagramas y preguntas de examen. Si un término del glosario crece más allá de una pantalla, se promueve a `concepts/` y la entrada de glosario se borra (dejando el link apuntando a la nueva página).

Estructura del cuerpo (breve — el valor está en la concisión):

- Un blockquote **`> **En una línea:**`** con la definición de una sola frase.
- **Definición** — 2-4 oraciones o una tabla chica.
- **Dónde aparece** — links a las páginas wiki que usan el término (son los backlinks "a mano").
- **Dato de examen** — el gotcha asociado, si lo hay.
- **Ver también** — links a otros términos del glosario.

**Regla de linkeo:** al ingestar o editar una página, la **primera aparición** de un término del glosario en esa página se convierte en link con alias — `[[globally-resilient|globally resilient]]` — para que el texto siga leyéndose natural. Las apariciones siguientes en la misma página se dejan en texto plano: la wiki se vuelve ilegible si cada párrafo es una sopa de links.

Dos detalles prácticos:
- **Excepción por consistencia visual:** si una fila o columna de una tabla comparativa enumera varios términos del glosario, se linkean **todos** aunque alguno ya estuviera linkeado en la prosa anterior (linkear 2 de 3 queda peor que linkear los 3).
- **Dentro de tablas** el pipe del alias se escapa: `[[iaas\|IaaS]]`, o Obsidian lo interpreta como separador de celda.

### Páginas de dominio (`wiki/domains/`)
- **Peso en el examen** — porcentaje
- **Temas clave** — lista con links a páginas de servicio/concepto
- **Servicios más importantes** para este dominio
- **Resumen de lo que el examen evalúa**

### Páginas de comparación (`wiki/comparisons/`)
- Tabla comparativa
- **Cuándo usar X vs Y**
- **Trampa típica del examen**


---

## guia-estudio.md — Guía de estudio del material

Archivo en la raíz (mismo nivel que `index.md` y `log.md`) que propone un **orden de aprendizaje** para todo el contenido ya ingestado en `wiki/`.

**Qué NO es:**
- No es una guía organizada por examen ni por peso de dominio — para eso ya están `index.md` y las páginas de `wiki/domains/`.
- No es un índice alfabético ni cronológico de ingest. El orden de aprendizaje no tiene por qué coincidir con el orden en que se fueron agregando las fuentes.

**Qué SÍ es:**
- La secuencia pedagógica recomendada: qué conviene leer primero para que lo siguiente tenga sentido, según prerequisitos **conceptuales** (no requisitos de examen).
- Organizada en bloques temáticos (ej: "Fundamentos y networking", "Identidad y acceso", "Almacenamiento"...), con las páginas de `wiki/` en el orden en que conviene recorrerlas dentro de cada bloque.
- Un documento **vivo**, no append-only: se reescribe y reordena las veces que haga falta — a diferencia de `log.md`, que nunca se reescribe.

**Regla:** el LLM actualiza `guia-estudio.md` en cada ingest (ver paso correspondiente en el flujo de Ingest). Al incorporar contenido nuevo:
1. Decide en qué bloque temático encaja la página nueva (o si hace falta crear un bloque).
2. La ubica en el lugar correcto dentro de la secuencia de ese bloque, según sus prerequisitos conceptuales — no simplemente la agrega al final.
3. Si el contenido nuevo deja en evidencia que el orden de páginas *ya existentes* estaba mal (por ejemplo, algo que se pensaba fundacional en realidad depende de un concepto recién agregado), reordena lo que haga falta.
4. Cada página referenciada usa el link estilo Obsidian (`[[NombreDePágina]]`) y, opcionalmente, una razón de una línea de por qué va en ese punto de la secuencia (útil cuando el orden no es obvio).

---

## Dominios del examen DVA-C02 *(examen en foco)*

> Fuente: [AWS Certified Developer - Associate (DVA-C02) — Exam Guide oficial](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html)

| Dominio | Nombre | Peso |
|---------|--------|------|
| 1 | Development with AWS Services | 32% |
| 2 | Security | 26% |
| 3 | Deployment | 24% |
| 4 | Troubleshooting and Optimization | 18% |

Formato del examen: 65 preguntas (50 puntuadas + 15 no puntuadas), 130 minutos, score mínimo 720/1000.


---

## Flujos de trabajo

### Ingest (fuente nueva)

> ⚠️ **Cómo detectar qué falta ingestar.** Cruzar nombres de archivo contra los `sources:` de la wiki **solo detecta archivos nunca ingestados** — es **ciego a los archivos ya citados que el humano después amplió o reescribió**, que es el caso más frecuente. Antes de concluir "no hay nada nuevo", verificar las **dos** cosas:
> 1. **Archivos nuevos**: secciones de `raw/` que no aparecen en ningún `sources:`.
> 2. **Archivos modificados**: comparar el **mtime** de cada fuente contra la fecha `updated:` de las páginas wiki que la citan. Si la fuente es más nueva que la página, hay contenido sin ingestar. (`git log`/`git diff` sirve solo si el archivo ya estaba versionado antes de la edición; si entró como `A` en su commit, git no muestra el cambio interno.)
>
> Ante la duda, **abrir la fuente y compararla con la página** — un mtime reciente no siempre es una ampliación real, pero descartarlo sin mirar sí es un error.

1. El humano agrega o **amplía** un archivo en `raw/` y dice "ingestar X"
2. El LLM lee la fuente
3. Discute con el humano los puntos clave si es necesario
4. Crea o actualiza páginas en `wiki/` afectadas
5. **Glosario:** identifica los términos nuevos que la fuente introduce sin definir, crea su entrada en `wiki/glossary/` (según los criterios de la sección "Páginas de glosario") y linkea su primera aparición en las páginas afectadas
6. Actualiza `index.md` con cualquier página nueva
7. Actualiza `guia-estudio.md`, ubicando el contenido nuevo en el orden pedagógico que corresponda (ver sección `guia-estudio.md` arriba)
8. Si la fuente trae **demos del curso**, las agrega a `demos.md` y a la sección "Demos del curso" de las páginas wiki relacionadas
9. Agrega entrada al `log.md`

### Query (pregunta)

1. El LLM lee `index.md` para encontrar páginas relevantes
2. Lee las páginas pertinentes
3. Sintetiza respuesta con citas a las páginas wiki
4. Si la respuesta es valiosa y reutilizable, la archiva como nueva página en `wiki/`

### Quiz (práctica activa)

1. El humano dice **"quiz \<tema\>"** — el tema puede ser una página wiki (`quiz S3`), un bloque de la guía (`quiz bloque 2`), un dominio (`quiz dva-security`) o `general`. Opcionalmente la cantidad (`quiz S3 10`).
2. El LLM genera preguntas **estilo examen** (default: 5): escenario + 4 opciones (una o varias correctas), inspiradas en las secciones "Gotchas", las comparaciones y los datos numéricos de las páginas del alcance. **Una pregunta por vez**; espera la respuesta antes de la siguiente. No revelar la respuesta en la pregunta.
3. Tras cada respuesta: corrige, explica **por qué** cada opción es correcta o distractor, y linkea la página wiki correspondiente.
4. Al final: puntaje y resumen. **Las preguntas falladas se archivan** en `wiki/exams/repaso-errores.md` (pregunta completa + respuesta + explicación + link + fecha) — es la lista de debilidades del humano.
5. Si se archivaron errores, agregar entrada breve en `log.md` (`tipo: quiz`).

---
### Lint (mantenimiento)

Periódicamente (o cuando el humano lo pide), el LLM:
- Busca contradicciones entre páginas
- Identifica páginas huérfanas (sin links entrantes)
- Sugiere conceptos mencionados que no tienen su propia página
- Verifica que `index.md` esté actualizado

---

## Convenciones

- Nombres de archivos: `kebab-case.md` para conceptos/comparaciones/**glosario**, `PascalCase.md` para servicios AWS
- Los términos del glosario se nombran **en inglés** (`globally-resilient.md`, `zone-apex.md`) porque así aparecen en el examen y en la doc; el **cuerpo va en español**, como el resto de la wiki
- Links internos: `[[NombreDePágina]]` estilo Obsidian; con alias cuando el texto corre en una oración: `[[globally-resilient|globally resilient]]`
- Citas a fuentes: `> Fuente: raw/nombre-del-archivo.md`
- Nunca borrar contenido sin reemplazarlo; si algo queda obsoleto, marcarlo con `> ⚠️ Outdated: <razón>`
- Mantener las páginas concisas — el examen requiere claridad, no enciclopedias

---


