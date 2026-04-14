# ADR-001: Ampliación del Dominio – Gestión de Tripulación (Crew Management)

## Título
Incorporar dominio funcional de Gestión de Tripulación (Crew Management)

## Contexto
El modelo actual gestiona de forma completa la flota de aeronaves, las operaciones de vuelo y los pasajeros, pero no registra la asignación de tripulación (pilotos, copilotos, auxiliares de vuelo) a los segmentos de vuelo.

Esta información es operacionalmente crítica para la aerolínea y es exigida por entidades regulatorias de aviación civil.

## Problema
No existe forma de registrar:
- Qué miembros de tripulación estuvieron asignados a cada vuelo
- Su rol a bordo
- Su certificación habilitante
- El cumplimiento de restricciones de horas de vuelo

Esto representa un vacío funcional y regulatorio en el modelo actual.

## Decisión
Crear el dominio **CREW** con las siguientes tablas:

- **crew_role**  
  Roles de tripulación (CAPTAIN, FIRST_OFFICER, FLIGHT_ATTENDANT, etc.)

- **crew_member**  
  Relaciona un miembro de tripulación con:
  - person (FK)
  - airline (FK)  
  Incluye campos de licencia y habilitaciones

- **crew_qualification**  
  Certificaciones por tipo de aeronave

- **flight_crew_assignment**  
  Asignación de un crew_member a un flight_segment con su rol

Este dominio se integra a:
- Identidad (person)
- Aerolínea (airline)
- Operaciones de Vuelo (flight_segment)

## Justificación técnica
- Uso de UUID como clave primaria, consistente con el modelo actual
- Separación entre catálogos (crew_role) y entidades operativas (crew_member, flight_crew_assignment)
- Uso de claves foráneas hacia tablas existentes sin modificarlas
- Extensión completamente aditiva (no rompe compatibilidad)

Además, cumple con requisitos reales de la industria aeronáutica (regulación DGAC/FAA sobre gestión de tripulación).

**Este dominio se plantea como una extensión controlada del modelo existente, sin modificar las estructuras definidas en el script base, respetando la restricción de no alterar el diseño original.**

## Consecuencias o impacto esperado
- Se agregan aproximadamente 4 nuevas tablas al modelo de datos
- Se debe crear el changelog de Liquibase para el dominio CREW
- Se deben definir permisos específicos en el sistema de seguridad (ej: OPS_MANAGER)
- No se modifican tablas existentes
- Se mejora la trazabilidad operativa y cumplimiento regulatorio