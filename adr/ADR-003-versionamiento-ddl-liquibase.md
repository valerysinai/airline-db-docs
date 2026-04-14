# ADR-003: Implementación de Liquibase para Versionamiento del DDL

## Título
Adoptar Liquibase como herramienta de gestión y versionamiento del DDL del modelo

## Contexto
El archivo `modelo_postgresql.sql` es actualmente un script DDL monolítico.

A medida que el proyecto evolucione (nuevas tablas, modificaciones, índices, datos de referencia), mantener un único archivo SQL se vuelve difícil de gestionar:

- No hay trazabilidad de cambios
- No se sabe cuándo ni quién aplicó cada modificación
- No hay control por ambiente (local, QA, producción)
- No existe mecanismo de rollback

## Problema
La ausencia de versionamiento del esquema genera múltiples riesgos:

- Desincronización entre ambientes
- Cambios manuales propensos a error
- Falta de visibilidad del estado del esquema
- Imposibilidad de revertir cambios incorrectos
- Riesgo de inconsistencias en la base de datos

## Decisión
Implementar Liquibase con la siguiente estrategia:

1. **Changelog maestro**
   - `changelog-master.xml`
   - Incluye los changelogs de cada dominio mediante `include` o `includeAll`
   - Ordenado por dependencias

2. **Changelog por dominio**
   - Ejemplos:
     - changelog-geography.xml
     - changelog-airline.xml
     - changelog-identity.xml
   - Organización modular del esquema

3. **Changesets**
   - Cada changeset debe tener:
     - `id` único
     - `author`
     - `context` (ej: `ddl`, `seed`)

4. **Ejecución con Docker**
   - Uso de imagen oficial: `liquibase/liquibase`
   - Integración con entorno de desarrollo y despliegue

 El sistema de versionamiento garantiza que todos los cambios se realicen de forma incremental sin modificar directamente el script base del modelo.

## Justificación técnica
- Liquibase es una herramienta estándar para versionamiento de bases de datos
- Utiliza tablas internas:
  - DATABASECHANGELOG
  - DATABASECHANGELOGLOCK
- Permite:
  - Control de cambios incremental
  - Rollback de cambios
  - Uso de etiquetas (tags)
  - Contextos y precondiciones
- Compatible con PostgreSQL
- Integración sencilla con Docker y pipelines CI/CD
- Disponible en versión community (gratuita)

El archivo `modelo_postgresql.sql` se conserva como referencia base y no se modifica directamente; todos los cambios futuros se gestionan exclusivamente mediante changesets de Liquibase.

## Consecuencias o impacto esperado
- El archivo `modelo_postgresql.sql` pasa a ser solo referencia histórica
- Todos los cambios futuros se gestionan mediante changesets
- Se requiere inicialización del proyecto Liquibase:
  - `liquibase.properties` o `liquibase.yml`
- Se debe configurar Docker para montar los changelogs como volumen
- Se simplifica el onboarding de desarrolladores:
  - ejecución de `docker-compose up`
- Mejora significativa en control, trazabilidad y estabilidad del esquema