# Plan de Trabajo Inicial — Estabilización de la Base de Datos

**Proyecto:** Sistema de Gestión Aeronáutica  
**Fase:** Ejecución  
**Duración prueba supervisada:** 5 horas  
**Fecha de elaboración:** 14/04/2026  

---

## Objetivo general

Dejar estructurada la base técnica y documental del proyecto a partir del análisis del modelo `modelo_postgresql.sql`, de forma que el trabajo desescolarizado pueda ejecutarse de manera ordenada, trazable y coherente con los entregables definidos en el instrumento de evaluación.

---

## Actividades ejecutadas en la prueba supervisada

| # | Actividad                                                                 | Estado        |
|--|--------------------------------------------------------------------------|--------------|
| 1 | Lectura y análisis del archivo `modelo_postgresql.sql`                  | Completado   |
| 2 | Identificación y documentación de los 12 dominios funcionales           | Completado   |
| 3 | Creación del repositorio de documentación (`airline-db-docs`)           | Completado   |
| 4 | Creación del repositorio del proyecto (`airline-db-project`)            | Completado   |
| 5 | Definición de ramas `develop`, `qa` y `main` en ambos repositorios      | Completado   |
| 6 | Elaboración del `backlog_tecnico.md` con HU-001 a HU-008                | Completado   |
| 7 | Elaboración del `plan_datos_prueba.md`                                  | Completado   |
| 8 | Documentación de los 5 ADR requeridos                                   | Completado   |
| 9 | Inicio de la estructura base del proyecto con Liquibase                 | En progreso  |
| 10| Elaboración del presente plan de trabajo                                | Completado   |

---

## Plan de trabajo desescolarizado

### SEMANA 1 — Infraestructura y estructura base

| Tarea                                                             | HU asociada | Prioridad |
|------------------------------------------------------------------|------------|----------|
| Crear `docker-compose.yml` con servicio PostgreSQL               | HU-003     | Alta     |
| Crear `.env.example` con variables de entorno                    | HU-003     | Alta     |
| Verificar que PostgreSQL levanta correctamente (`docker compose up`) | HU-003 | Alta     |
| Agregar servicio Liquibase al `docker-compose.yml`               | HU-004     | Alta     |
| Crear `liquibase.properties.example`                             | HU-004     | Alta     |
| Verificar conexión de Liquibase con PostgreSQL                   | HU-004     | Alta     |

---

### SEMANA 2 — Migración del DDL a Liquibase por dominio

| Tarea                                                                 | HU asociada | Prioridad |
|----------------------------------------------------------------------|------------|----------|
| Crear `changelog-master.yaml` en la raíz                             | HU-005     | Alta     |
| Crear changelog para extensiones (`00_extensions/`)                  | HU-005     | Alta     |
| Crear changelogs por dominio en `01_ddl/03_tables/` (12 dominios)    | HU-005     | Alta     |
| Crear changelog para índices (`09_indexes/`)                         | HU-005     | Alta     |
| Verificar ejecución completa de Liquibase sin errores                | HU-005     | Alta     |
| Validar que la estructura coincide con el modelo original            | HU-005     | Alta     |

---

### SEMANA 3 — Seguridad y roles

| Tarea                                              | HU asociada | Prioridad |
|---------------------------------------------------|------------|----------|
| Definir roles: `role_readonly`, `role_operations`, `role_admin` | HU-006 | Media |
| Crear changelogs de roles en `03_dcl/00_roles/`    | HU-006     | Media    |
| Crear changelogs de grants en `03_dcl/01_grants/`  | HU-006     | Media    |
| Verificar correcta aplicación de permisos          | HU-006     | Media    |

---

### SEMANA 4 — Datos de prueba e integración final

| Tarea                                                                 | HU asociada | Prioridad |
|----------------------------------------------------------------------|------------|----------|
| Recibir script de migración de datos del instructor                  | HU-007     | Media    |
| Organizar scripts en `02_dml/00_inserts/`                            | HU-007     | Media    |
| Ejecutar inserciones según el plan de datos de prueba                | HU-007     | Media    |
| Validar integridad referencial por fase                             | HU-007     | Media    |
| Actualizar `seguimientos.md` por sesión                             | HU-008     | Media    |
| Revisar coherencia global (ADR, HU, changelogs, docs)               | HU-008     | Media    |

---

## Estructura de ramas y flujo de trabajo

```text
main
 └── qa
      └── develop  ← rama de trabajo activo