# ADR-003 — Implementación de Liquibase para versionamiento del DDL

## Título
Implementación de Liquibase para administrar el versionamiento del DDL.

## Contexto
El modelo existe como un único archivo `modelo_postgresql.sql`. No hay mecanismo de control sobre los cambios que se aplican a la base de datos, lo que impide rastrear qué cambios se hicieron, cuándo y en qué entorno.

## Problema
Sin versionamiento del DDL, aplicar cambios en diferentes entornos (local, qa, producción) es manual y propenso a errores. No existe forma de revertir un cambio ni garantizar que todos los entornos tienen la misma estructura.

## Decisión
Se implementa Liquibase como herramienta de versionamiento del DDL, organizado así:

- Un `changelog-master.yaml` en la raíz referencia todos los changelogs
- Cada dominio funcional tiene su propio changelog dentro de `01_ddl/03_tables/`
- El DDL se organiza además por tipo: extensiones, esquemas, tablas, índices
- Liquibase se ejecuta mediante contenedor Docker integrado al proyecto

## Justificación técnica

- Liquibase registra cada cambio aplicado en la tabla `DATABASECHANGELOG`
- Permite rollback controlado por changeset
- La organización por dominio evita changelogs masivos y facilita el mantenimiento
- Es compatible con PostgreSQL y se integra naturalmente con Docker Compose

## Consecuencias e impacto esperado

- Todo cambio al DDL pasa obligatoriamente por un changeset versionado
- Los entornos `develop`, `qa` y `main` tienen garantía de estructura idéntica
- El archivo `modelo_postgresql.sql` original queda solo como referencia de análisis