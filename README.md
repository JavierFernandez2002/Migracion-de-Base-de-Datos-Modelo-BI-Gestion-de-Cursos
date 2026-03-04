# Migración de Base de Datos + Modelo BI - Gestión de Cursos

Proyecto académico realizado para la materia **Gestión de Datos** (UTN.BA), enfocado en el rediseño de un sistema a partir de una **tabla maestra desnormalizada**, su **migración a un modelo transaccional normalizado** y la construcción de un **modelo de Business Intelligence (BI)** para análisis e indicadores de gestión.

## Resumen

El trabajo parte de un escenario realista de migración: una única tabla con datos desorganizados y sin normalización. A partir de ese origen se diseñó una solución completa en **T-SQL / SQL Server** que incluye:

- un **modelo transaccional relacional** con entidades, claves primarias, claves foráneas y restricciones;
- la **migración de datos** hacia el nuevo esquema;
- un **modelo dimensional BI** con dimensiones y tablas de hechos;
- y un conjunto de **vistas analíticas** para responder preguntas de negocio.

El dominio del proyecto es la **gestión de cursos** de una institución: alumnos, profesores, sedes, inscripciones, cursadas, finales, pagos y encuestas.

## Objetivos del proyecto

- Normalizar datos provenientes de una tabla maestra legacy.
- Diseñar un esquema relacional consistente y mantenible.
- Resolver problemas de calidad de datos sin alterar la fuente original.
- Construir un modelo BI orientado a métricas y tableros de control.
- Implementar consultas analíticas con vistas reutilizables.

## Arquitectura de la solución

### 1) Modelo transaccional

Se diseñó un esquema relacional para reemplazar la tabla maestra, separando la información en entidades de negocio y tablas de referencia.

Principales áreas modeladas:

- **Ubicación:** provincia, localidad
- **Entidades principales:** sede, profesor, alumno
- **Académico:** curso, módulos, evaluaciones, trabajo práctico, finales
- **Inscripciones:** inscripción a curso, inscripción a final, estado de inscripción
- **Finanzas:** factura, detalle de factura, pago, medio de pago, período
- **Encuestas:** encuesta respondida, preguntas

### 2) Modelo BI

Sobre el modelo transaccional se construyó un esquema analítico con:

**Dimensiones**
- `bi_tiempo`
- `bi_sede`
- `bi_categorias_curso`
- `bi_turnos_curso`
- `bi_medio_de_pago`
- `bi_rango_edad_alumnos`
- `bi_rango_edad_profesores`
- `bi_bloque_de_satisfaccion`

**Tablas de hechos**
- `bi_inscripcion`
- `bi_cursada`
- `bi_final`
- `bi_pagos`
- `bi_encuesta`

Este modelo permite consolidar indicadores de inscripción, desempeño académico, finales, comportamiento financiero y satisfacción.

## Modelos de datos

### DER del modelo transaccional

> Si subís la imagen al repositorio con este mismo nombre, GitHub la mostrará automáticamente.

![Modelo DER](./MODELO%20DER.png)

### DER del modelo BI

![Modelo BI](./MODELO%20BI.png)

## Decisiones de diseño destacadas

Algunas decisiones importantes tomadas durante el modelado y la migración:

- Se definió `UNIQUE (mes, anio)` en `periodo` para evitar períodos duplicados.
- Se normalizó la ubicación en `provincia` y `localidad`, referenciando desde alumno, profesor y sede.
- Se unificó la información institucional en `sede` (por tratarse de una única institución dentro del dominio).
- Se utilizó el **legajo** como PK de `alumno`.
- Se reutilizaron identificadores de la tabla maestra cuando tenía sentido de negocio (por ejemplo, en curso e inscripciones).
- Se incorporaron restricciones de integridad como:
  - checks de rango (por ejemplo, meses válidos);
  - checks de consistencia temporal;
  - claves foráneas entre entidades.

## Calidad de datos y resolución de inconsistencias

Como parte del proceso de migración, se detectaron y documentaron inconsistencias típicas de sistemas legacy. En lugar de modificar la fuente original, se resolvieron en la lógica de carga.

Casos abordados:

- **Encuestas duplicadas:** deduplicación usando `ROW_NUMBER()` y conservación del primer registro válido.
- **Columnas invertidas en ubicación:** corrección lógica durante los `INSERT` de migración.
- **Registros gemelos en finales:** priorización del registro con nota válida sobre el registro nulo/ausente.

Esto refleja una práctica clave en migraciones reales: **controlar el impacto de datos defectuosos sin adulterar la fuente histórica**.

## Métricas e indicadores BI

El modelo BI fue diseñado para responder indicadores de negocio como:

1. Top 3 de categorías y turnos más solicitados por año y sede.
2. Tasa de rechazo de inscripciones por mes y sede.
3. Comparación de aprobación de cursada por sede.
4. Tiempo promedio de finalización de curso.
5. Nota promedio de finales por rango etario del alumno y categoría.
6. Tasa de ausentismo en finales.
7. Desvío de pagos fuera de término.
8. Tasa de morosidad financiera mensual.
9. Top 3 de categorías con mayores ingresos por sede y año.
10. Índice de satisfacción por rango etario de profesores y sede.

## Vistas implementadas

Para exponer los indicadores de manera simple y reutilizable, se crearon las siguientes vistas:

- `categorias_turnos_top_3`
- `tasa_rechazo_inscripciones`
- `aprobacion_cursada_sede`
- `promedio_finalizacion_curso`
- `promedio_finales_rango`
- `tasa_ausentismo_finales`
- `desvio_pagos_semestre`
- `tasa_morosidad_mensual`
- `top_ingresos_categoria_anio_sede`
- `indice_satisfaccion_rango_sede`

## Tecnologías utilizadas

- **Microsoft SQL Server**
- **T-SQL**
- Modelado relacional (DER)
- Modelado dimensional para BI
- Funciones de ventana (`ROW_NUMBER`, `RANK`)
- Agregaciones y vistas analíticas

## Estructura esperada del repositorio

```text
.
├── script_creación_inicial.sql
├── script_creacion_BI.sql
├── Estrategia.pdf
├── MODELO DER.png
├── MODELO BI.png
└── README.md
```

> Si querés evitar problemas de encoding en distintos entornos, también podés renombrar `script_creación_inicial.sql` a `script_creacion_inicial.sql`.

## Cómo ejecutar

1. Crear una base de datos en SQL Server (por ejemplo `GD2C2025`).
2. Cargar previamente el esquema base y la **tabla maestra** provista por la cátedra.
3. Ejecutar el script de creación y migración del modelo transaccional:

```sql
:r script_creación_inicial.sql
```

4. Ejecutar luego el script de creación y carga del modelo BI:

```sql
:r script_creacion_BI.sql
```

5. Consultar las vistas analíticas para validar métricas y resultados.

## Qué demuestra este proyecto

Este proyecto muestra capacidad para:

- analizar y rediseñar esquemas de bases de datos;
- migrar datos desde estructuras desnormalizadas;
- aplicar integridad referencial y reglas de negocio;
- resolver problemas de calidad de datos en procesos ETL;
- modelar data marts orientados a reporting;
- traducir requerimientos de negocio en consultas analíticas concretas.

## Contexto académico

- **Materia:** Gestión de Datos
- **Institución:** UTN.BA - Facultad Regional Buenos Aires
- **Grupo:** INSERT_PROMOCIONADOS (Grupo 39)

## Autoría

Trabajo realizado por:

- Javier Fernandez
- Melina Loza Flores
- Alexia Deza

## Nota para portfolio

Este repositorio representa un caso completo de **migración + normalización + BI** en SQL Server, mostrando tanto la parte operativa del sistema como la capa analítica para toma de decisiones. Es un muy buen ejemplo de proyecto para portfolio porque combina **modelado**, **ETL**, **calidad de datos** y **consultas de negocio** en una única solución.
