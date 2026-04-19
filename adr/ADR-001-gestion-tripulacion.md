# ADR-001 — Incorporación del dominio de Notificaciones y Comunicaciones

## Título
Incorporación del dominio funcional de Notificaciones y Comunicaciones.

## Contexto
El modelo cubre robustamente los flujos operativos del sistema aeronáutico. Sin embargo, no existe ninguna entidad que registre las comunicaciones enviadas a pasajeros ante eventos como cambios de vuelo, confirmaciones de reserva o alertas de check-in.

## Problema
El sistema carece de trazabilidad sobre las comunicaciones enviadas. No es posible auditar si una notificación fue enviada, por qué canal, en qué momento ni si fue entregada correctamente.

## Decisión
Se incorpora el dominio `notification` con las entidades:

- `notification_channel` — canales disponibles (EMAIL, SMS, PUSH)
- `notification_template` — plantillas por tipo de evento
- `notification_event` — registro de cada notificación disparada, con `person_id`, canal, plantilla, estado y timestamp

## Justificación técnica

- Sigue el estándar del modelo: UUID como PK, created_at/updated_at, restricciones CHECK
- Referencia a `person` del dominio identidad, manteniendo coherencia relacional
- Es un dominio aditivo — no modifica ninguna tabla existente
- Se agrega un changelog en `01_ddl/03_tables/13_notification/` sin afectar los demás

## Consecuencias e impacto esperado

- Mejora la trazabilidad y auditoría de comunicaciones con pasajeros
- Requiere un nuevo changelog en Liquibase y entradas en el plan de datos de prueba
- No genera cambios destructivos sobre el modelo base