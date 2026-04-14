# ADR-002: Manejo de Roles con Permisos Diferenciados

## Título
Implementar estrategia de roles PostgreSQL a nivel de base de datos complementando el RBAC del modelo de aplicación

## Contexto
El modelo ya incluye un dominio de Seguridad con tablas:
- security_role
- security_permission
- user_account
- user_role
- role_permission

Estas implementan RBAC a nivel de aplicación. Sin embargo, a nivel de base de datos no existe separación de permisos entre roles funcionales.

Actualmente, cualquier usuario con acceso a la base de datos puede leer y modificar cualquier tabla.

## Problema
La ausencia de roles diferenciados a nivel de base de datos genera riesgos de seguridad:

- Posible acceso a datos sensibles (passwords, documentos, información financiera)
- Falta de control ante accesos directos a la base de datos
- Riesgo de escalamiento de privilegios ante fallos en la aplicación
- No existe trazabilidad de operaciones a nivel del motor de base de datos

## Decisión
Crear los siguientes roles en PostgreSQL:

- **app_readonly**
  - Permisos: SELECT en todas las tablas
  - Uso: consultas, reportes y analítica

- **app_readwrite**
  - Permisos: SELECT, INSERT, UPDATE en tablas operativas
  - Restricción: sin acceso a `user_account.password_hash`

- **app_admin**
  - Permisos: acceso completo
  - Uso: tareas administrativas y migraciones

- **app_security**
  - Permisos exclusivos sobre tablas del dominio Seguridad:
    - user_account
    - user_role
    - role_permission
    - security_role
    - security_permission

Cada rol será asignado a usuarios de conexión específicos según el contexto de ejecución.

## Justificación técnica
- Se aplica el principio de **mínimo privilegio (least privilege)**
- Se garantiza la separación entre la seguridad a nivel de aplicación (RBAC) y la seguridad a nivel de base de datos (roles PostgreSQL), asegurando múltiples capas de protección.
- Reduce riesgos ante fallos en la capa de aplicación
- PostgreSQL permite control granular mediante:
  - GRANT
  - REVOKE
  - permisos por tabla, columna y esquema
- Complementa el RBAC existente a nivel de aplicación (no lo reemplaza)
- Mejora la seguridad en entornos productivos

## Consecuencias o impacto esperado
- Se deben incluir scripts DDL de creación de roles en Liquibase
- Se debe actualizar el `docker-compose.yml` para manejar credenciales por rol
- Se deben inicializar roles en ambientes de prueba
- Impacto bajo en código de aplicación:
  - ajuste de connection strings según contexto
- Mejora significativa en seguridad y control de accesos