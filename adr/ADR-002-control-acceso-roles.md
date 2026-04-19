# ADR-002 — Estrategia de roles y permisos diferenciados (RBAC)

## Título
Estrategia de roles y permisos diferenciados sobre el modelo de seguridad existente.

## Contexto
El modelo ya cuenta con las tablas `security_role`, `security_permission`, `user_role` y `role_permission` que conforman una estructura RBAC completa a nivel de datos. Sin embargo, no está definida la estrategia concreta de qué roles existen, qué permisos tiene cada uno ni cómo se aplican a nivel de base de datos mediante DCL.

## Problema
Sin roles de base de datos definidos, cualquier usuario conectado puede ejecutar operaciones sobre cualquier tabla. Esto viola el principio de mínimo privilegio y deja sin trazabilidad los accesos al sistema.

## Decisión
Se definen tres roles de base de datos gestionados mediante changelogs en `03_dcl/`:

| Rol               | Permisos                                      |
|------------------|----------------------------------------------|
| role_readonly     | SELECT sobre todas las tablas                |
| role_operations   | SELECT, INSERT, UPDATE sobre tablas operativas |
| role_admin        | SELECT, INSERT, UPDATE, DELETE sobre todas las tablas |

## Justificación técnica

- Los roles se crean en `03_dcl/00_roles/` y los grants en `03_dcl/01_grants/`
- Liquibase gestiona y versiona los cambios DCL igual que el DDL
- Se alinea con las tablas `security_role` y `role_permission` ya existentes en el modelo
- Permite auditar quién tiene acceso a qué sin intervenir el modelo base

## Consecuencias e impacto esperado

- Se garantiza el principio de mínimo privilegio en todos los entornos
- Los changelogs DCL son independientes del DDL y pueden evolucionar por separado
- Cualquier cambio de permisos queda versionado y es reversible mediante rollback