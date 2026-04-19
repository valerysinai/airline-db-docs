# Backlog Técnico — Estabilización de la Base de Datos

**Proyecto:** Sistema de Gestión Aeronáutica  
**Herramienta sugerida:** GitHub Projects / Jira / Trello  

---

## Historias de Usuario

### HU-001 — Identificar y documentar dominios funcionales del modelo existente
- **Prioridad:** Alta  
- **Estado:** Completado  

**Descripción:**  
Como equipo técnico, necesito analizar el archivo `modelo_postgresql.sql` e identificar todos los dominios funcionales, sus entidades principales y sus relaciones, para tener una base de comprensión sólida del modelo antes de cualquier intervención.

**Criterios de aceptación:**
- Todos los dominios están identificados y documentados en `analisis_dominios.md`
- Cada dominio incluye sus tablas, relaciones relevantes y restricciones notables
- El documento está versionado en el repositorio de documentación

---

### HU-002 — Organizar la estructura base del repositorio y definir ramas develop, qa y main
- **Prioridad:** Alta  
- **Estado:** En progreso  

**Descripción:**  
Como equipo técnico, necesito crear y organizar el repositorio del proyecto con la estructura de carpetas definida y las ramas de trabajo, para garantizar un flujo de versionamiento controlado y ordenado.

**Criterios de aceptación:**
- Repositorio creado con estructura de carpetas según el modelo del instructor
- Ramas `develop`, `qa` y `main` creadas y protegidas
- `README.md` documentado con propósito del proyecto y guía de uso
- `.env.example` y `liquibase.properties.example` presentes en la raíz

---

### HU-003 — Contenerizar PostgreSQL para levantar la base de datos en entorno local
- **Prioridad:** Alta  
- **Estado:** Pendiente  

**Descripción:**  
Como desarrollador, necesito poder levantar una instancia de PostgreSQL mediante Docker, para trabajar el modelo de base de datos en entorno local sin depender de instalaciones manuales.

**Criterios de aceptación:**
- `docker-compose.yml` funcional que levanta PostgreSQL
- Variables de entorno externalizadas en `.env`
- La base de datos levanta correctamente con `docker compose up`
- El puerto y las credenciales están documentados en el `README.md` de docker

---

### HU-004 — Contenerizar Liquibase e integrarlo al proyecto
- **Prioridad:** Alta  
- **Estado:** Pendiente  

**Descripción:**  
Como desarrollador, necesito integrar Liquibase como contenedor en el proyecto, para que el versionamiento del DDL sea aplicable de forma automatizada y reproducible en cualquier entorno.

**Criterios de aceptación:**
- Liquibase está definido como servicio en `docker-compose.yml`
- `liquibase.properties.example` documenta la configuración base
- Liquibase se conecta correctamente a la instancia PostgreSQL del contenedor
- El `changelog-master.yaml` está referenciado correctamente desde Liquibase

---

### HU-005 — Separar el DDL en changelogs organizados por dominio funcional
- **Prioridad:** Alta  
- **Estado:** Pendiente  

**Descripción:**  
Como equipo técnico, necesito migrar el contenido del `modelo_postgresql.sql` a changelogs de Liquibase organizados por dominio funcional, para que cada dominio tenga su propio archivo de versionamiento controlado.

**Criterios de aceptación:**
- Cada dominio tiene su propio `changelog.yaml` dentro de `01_ddl/03_tables/`
- La extensión `pgcrypto` está en `01_ddl/00_extensions/`
- Los índices están en `01_ddl/09_indexes/`
- El `changelog-master.yaml` referencia todos los changelogs en orden correcto
- No existe un changelog monolítico con todo el DDL

---

### HU-006 — Diseñar e implementar estrategia de roles y permisos diferenciados
- **Prioridad:** Media  
- **Estado:** Pendiente  

**Descripción:**  
Como administrador del sistema, necesito definir roles de base de datos con permisos diferenciados sobre los esquemas y tablas, para garantizar el principio de mínimo privilegio y la trazabilidad de accesos.

**Criterios de aceptación:**
- Roles definidos: `role_readonly`, `role_operations`, `role_admin`
- Cada rol tiene permisos explícitos documentados en `03_dcl/`
- Los `GRANTs` están organizados en changelogs de Liquibase bajo `03_dcl/01_grants/`
- La estrategia está documentada en el `ADR-002`

---

### HU-007 — Construir plan de datos de prueba con orden de carga por dependencias
- **Prioridad:** Media  
- **Estado:** Pendiente  

**Descripción:**  
Como equipo técnico, necesito un plan estructurado para insertar datos de prueba respetando el orden de dependencias entre tablas, para poder validar el funcionamiento del modelo sin violar restricciones de integridad referencial.

**Criterios de aceptación:**
- Plan documentado en `plan_datos_prueba.md` con orden explícito de inserción
- Scripts de inserción organizados en `02_dml/00_inserts/` por dominio
- Validaciones básicas documentadas por cada grupo de tablas
- El plan está alineado con el script de migración que entregará el instructor

---

### HU-008 — Documentar seguimiento técnico y decisiones arquitectónicas
- **Prioridad:** Media  
- **Estado:** En progreso  

**Descripción:**  
Como equipo técnico, necesito mantener un registro actualizado de las decisiones arquitectónicas y el seguimiento del proceso, para garantizar trazabilidad y coherencia entre todos los artefactos del proyecto.

**Criterios de aceptación:**
- Los 5 ADR están completos y versionados en la carpeta `adr/`
- El archivo `seguimientos.md` registra avances por sesión
- Existe coherencia entre dominios identificados, ADR, HU y changelogs de Liquibase

---

## Tabla resumen del backlog

| ID Historia | Descripción                                      | Prioridad | Estado        |
|------------|--------------------------------------------------|----------|--------------|
| HU-001     | Identificar y documentar dominios funcionales    | Alta     | Completado   |
| HU-002     | Organizar repositorio y definir ramas            | Alta     | En progreso  |
| HU-003     | Contenerizar PostgreSQL                          | Alta     | Pendiente    |
| HU-004     | Contenerizar Liquibase e integrarlo              | Alta     | Pendiente    |
| HU-005     | Separar DDL en changelogs por dominio            | Alta     | Pendiente    |
| HU-006     | Estrategia de roles y permisos                   | Media    | Pendiente    |
| HU-007     | Plan de datos de prueba                          | Media    | Pendiente    |
| HU-008     | Documentar seguimiento y decisiones              | Media    | En progreso  |

---

## Dependencias entre HU

```text
HU-001
  └── HU-002
        ├── HU-003
        │     └── HU-004
        │           └── HU-005
        │                 └── HU-006
        └── HU-007 (depende también del script del instructor)

HU-008 (transversal a todo el proceso)