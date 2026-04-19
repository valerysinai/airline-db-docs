# Seguimiento Técnico del Proyecto

**Proyecto:** Sistema de Gestión Aeronáutica  
**Repositorio documentación:** `airline-db-docs`  
**Repositorio proyecto:** `airline-db-project`  

---

## Formato de registro

Cada sesión de trabajo debe registrarse con la siguiente estructura:

- **Fecha:** fecha de la sesión  
- **Duración:** tiempo trabajado  
- **Actividades realizadas:** qué se hizo concretamente  
- **Decisiones tomadas:** cualquier decisión técnica relevante  
- **Problemas encontrados:** bloqueos o dificultades  
- **Próximos pasos:** qué sigue después de esta sesión  

---

## SESIÓN 01 — Prueba supervisada

- **Fecha:** 18/04/2026  
- **Duración:** 5 horas  
- **Tipo:** Prueba supervisada  

### Actividades realizadas

- Lectura y análisis completo del archivo `modelo_postgresql.sql`
- Identificación de los 12 dominios funcionales del modelo
- Documentación del análisis en `analisis_dominios.md`
- Creación del repositorio de documentación `airline-db-docs`
- Creación del repositorio del proyecto `airline-db-project`
- Definición de ramas `develop`, `qa` y `main` en ambos repositorios
- Elaboración del `backlog_tecnico.md` (HU-001 a HU-008)
- Elaboración del `plan_datos_prueba.md`
- Elaboración del `plan_trabajo_inicial.md`
- Documentación de los 5 ADR requeridos
- Inicio de la estructura base de changelogs en el repositorio del proyecto

### Decisiones tomadas

- Se adoptó la estructura del repositorio ejemplo del instructor (`shopping-cart-db`) como base
- El archivo `modelo_postgresql.sql` se ubica en la raíz del repositorio de documentación
- Los changelogs DDL se organizan por dominio en `01_ddl/03_tables/`
- Se definieron 12 dominios funcionales (separando Aeronaves y Operaciones de Vuelo)
- La rama activa de desarrollo es `develop`

### Problemas encontrados

- Ninguno crítico durante la prueba supervisada

### Próximos pasos

- Crear `docker-compose.yml` con PostgreSQL y Liquibase (HU-003, HU-004)
- Iniciar migración del DDL a Liquibase por dominio (HU-005)
- Esperar script de datos del instructor (HU-007)

---

## SESIÓN 02 — Trabajo desescolarizado

- **Fecha:** (pendiente)  
- **Duración:** (pendiente)  
- **Tipo:** Trabajo desescolarizado  

### Actividades realizadas
_(Registrar durante la sesión)_

### Decisiones tomadas
_(Registrar durante la sesión)_

### Problemas encontrados
_(Registrar durante la sesión)_

### Próximos pasos
_(Registrar durante la sesión)_

---

## SESIÓN 03 — Trabajo desescolarizado

- **Fecha:** (pendiente)  
- **Duración:** (pendiente)  
- **Tipo:** Trabajo desescolarizado  

### Actividades realizadas
_(Registrar durante la sesión)_

### Decisiones tomadas
_(Registrar durante la sesión)_

### Problemas encontrados
_(Registrar durante la sesión)_

### Próximos pasos
_(Registrar durante la sesión)_

---

## SESIÓN 04 — Trabajo desescolarizado

- **Fecha:** (pendiente)  
- **Duración:** (pendiente)  
- **Tipo:** Trabajo desescolarizado  

### Actividades realizadas
_(Registrar durante la sesión)_

### Decisiones tomadas
_(Registrar durante la sesión)_

### Problemas encontrados
_(Registrar durante la sesión)_

### Próximos pasos
_(Registrar durante la sesión)_

---

## Registro de decisiones arquitectónicas (ADR)

| ID      | Título                                                        | Estado     | Fecha       |
|--------|---------------------------------------------------------------|-----------|------------|
| ADR-001 | Incorporación del dominio de Notificaciones y Comunicaciones | Propuesto | 14/04/2026 |
| ADR-002 | Estrategia de roles y permisos (RBAC)                        | Propuesto | 14/04/2026 |
| ADR-003 | Implementación de Liquibase para DDL                         | Propuesto | 14/04/2026 |
| ADR-004 | Estrategia de versionamiento (develop, qa, main)             | Propuesto | 14/04/2026 |
| ADR-005 | Contenedorización con Docker Compose                         | Propuesto | 14/04/2026 |

---

## Registro de incidencias técnicas

| Fecha | Descripción | Impacto | Resolución |
|------|------------|--------|-----------|
| _(pendiente)_ | | | |

---

## Estado general del proyecto

| Entregable                              | Estado         |
|----------------------------------------|--------------|
| `analisis_dominios.md`                 | ✅ Completado |
| `backlog_tecnico.md`                   | ✅ Completado |
| `plan_datos_prueba.md`                 | ✅ Completado |
| `plan_trabajo_inicial.md`              | ✅ Completado |
| `seguimientos.md`                      | ✅ Completado |
| ADR-001 a ADR-005                      | ✅ Completado |
| Estructura repositorio proyecto        | 🔄 En progreso |
| `docker-compose.yml`                   | ⏳ Pendiente |
| Changelogs DDL por dominio             | ⏳ Pendiente |
| Changelogs DCL roles y permisos        | ⏳ Pendiente |
| Scripts DML datos de prueba            | ⏳ Pendiente |

---