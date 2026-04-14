# ADR-005: Estrategia de Contenedorización con Docker Compose

## Título
Definir arquitectura de contenedores Docker para levantar el entorno de base de datos de forma reproducible

## Contexto
El proyecto requiere que cualquier desarrollador pueda levantar el entorno completo de base de datos (PostgreSQL + Liquibase) en su máquina local sin configuraciones manuales complejas.

Además, los ambientes de QA y producción deben ser:
- Reproducibles
- Consistentes
- Fácilmente configurables

## Problema
Sin contenedorización:

- La configuración del entorno es manual
- Se presentan inconsistencias entre entornos ("funciona en mi máquina")
- Diferencias de versiones (PostgreSQL, locale, etc.)
- Problemas en el orden de ejecución de scripts
- Mayor dificultad para onboarding de nuevos desarrolladores

## Decisión
Utilizar **Docker Compose** con los siguientes servicios:

### 1. **db**
- Imagen: `postgres:16-alpine`
- Volumen persistente para datos
- Variables de entorno definidas en archivo `.env`:
  - Usuario
  - Contraseña
  - Nombre de la base de datos
- Puerto 5432 expuesto solo en localhost
- Healthcheck configurado

### 2. **liquibase**
- Imagen: `liquibase/liquibase:4.x`
- Depende del servicio `db` (condition: service_healthy)
- Monta volumen de changelogs en modo lectura
- Ejecución única:
  - `command: update`
- Usa credenciales desde `.env`
- Variables de entorno:
  - LIQUIBASE_COMMAND_URL
  - LIQUIBASE_COMMAND_USERNAME
  - LIQUIBASE_COMMAND_PASSWORD
  - LIQUIBASE_COMMAND_CHANGELOG_FILE

### Configuración adicional
- El archivo `.env`:
  - No se sube al repositorio
  - Se incluye en `.gitignore`
- Se provee un archivo `.env.example` como plantilla

## Justificación técnica
- Docker Compose es el estándar para entornos multi-servicio
- La imagen `postgres:16-alpine`:
  - Reduce tamaño
  - Minimiza superficie de ataque
- El **healthcheck** evita problemas de sincronización (race conditions)
- Uso de `.env` sigue el principio de configuración externa (12-Factor App)
- La imagen oficial de Liquibase evita dependencias locales (ej: Java)

Se garantiza que la infraestructura sea completamente reproducible y que cualquier entorno pueda inicializarse sin dependencias manuales externas.

## Consecuencias o impacto esperado
- El comando `docker-compose up` levanta toda la infraestructura automáticamente
- El DDL se aplica automáticamente mediante Liquibase
- Se debe documentar el proceso en el README
- Se definen múltiples archivos:
  - `docker-compose.yml` (desarrollo)
  - `docker-compose.qa.yml` (staging)

Se asegura separación clara entre configuración de desarrollo y ambientes de pruebas, evitando inconsistencias de despliegue.

- Los datos de prueba se gestionan con contextos de Liquibase:
  - `context='seed'` (ejecución opcional)

- Mejora significativa en reproducibilidad, estabilidad y onboarding