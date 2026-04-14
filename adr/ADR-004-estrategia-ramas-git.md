# ADR-004: Estrategia de Versionamiento del Repositorio con Git Flow

## Título
Definir estrategia de ramas Git con flujo develop → qa → main para el repositorio del proyecto

## Contexto
El proyecto de base de datos requiere una estrategia de control de versiones que permita:
- Desarrollo paralelo
- Validación en ambiente de pruebas
- Promoción controlada a producción

El repositorio contiene:
- Scripts DDL
- Changelogs de Liquibase
- Configuración Docker
- Documentación técnica
- Architecture Decision Records (ADR)

## Problema
Sin una estrategia de ramas definida:

- Los cambios se mezclan sin control
- No se puede identificar qué versión está en cada ambiente
- No existen controles para evitar que código no validado llegue a producción
- Se incrementa el riesgo de errores en despliegues

## Decisión
Adoptar el siguiente flujo de ramas:

### 1. **main**
- Rama de producción
- Solo recibe merges desde **qa** mediante Pull Request aprobado
- Protegida (sin push directo)
- Cada merge genera un tag de versión (ej: v1.0.0, v1.1.0)

### 2. **qa**
- Rama de validación
- Recibe merges desde **develop**
- Usada para pruebas de integración
- Los changelogs de Liquibase se prueban contra base de datos de staging

### 3. **develop**
- Rama de integración continua
- Recibe merges de feature branches
- Punto central del desarrollo activo

### 4. **feature/HU-XXX**
- Ramas de desarrollo por historia de usuario
- Se crean desde **develop**
- Se integran nuevamente a **develop** mediante Pull Request

**Convenciones de nombres:**
- feature/HU-001-documentar-dominios
- fix/correccion-constraint-fecha

## Justificación técnica
- Basado en una adaptación de **Git Flow**, ampliamente utilizada en la industria
- Separación clara de ambientes: desarrollo, pruebas y producción
- Uso de Pull Requests permite revisión de código (code review)
- Protección de ramas evita errores humanos
- Versionamiento mediante tags permite rollback a versiones estables
- Compatible con herramientas como GitHub Projects para gestión del backlog

 Se establece la aplicación de reglas de protección de ramas (branch protection rules) para garantizar que ninguna modificación llegue a `main` sin aprobación formal mediante Pull Request.

## Consecuencias o impacto esperado
- Configuración de reglas de protección de ramas (branch protection rules)
- Definición de proceso de revisión de PRs (mínimo un aprobador)
- Validación obligatoria de changelogs de Liquibase antes de merge a develop
- Se restringe el push directo a `main` bajo cualquier circunstancia
- Inclusión de archivo `.gitignore` para excluir:
  - Archivos `.env`
  - Logs
  - Archivos temporales
- Mejora en la calidad del código y control de versiones