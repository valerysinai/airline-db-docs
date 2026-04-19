# ADR-005-contenedorizacion-docker.md

## Título
Contenedorización del entorno de base de datos con Docker Compose.

## Contexto
El proyecto requiere que PostgreSQL y Liquibase puedan ejecutarse en cualquier entorno de desarrollo sin depender de instalaciones manuales. Cada desarrollador debe poder levantar el entorno completo de forma reproducible y consistente.

## Problema
Sin contenedorización, configurar el entorno local requiere instalación manual de PostgreSQL y Liquibase, lo que genera inconsistencias entre entornos y dificulta la incorporación de nuevos integrantes al proyecto.

## Decisión
Se implementa Docker Compose como estrategia de contenedorización con dos servicios:

Servicio | Imagen | Propósito  
postgres | postgres:15 | Instancia de base de datos local  
liquibase | liquibase/liquibase | Ejecutor de changelogs sobre PostgreSQL  

Archivos clave:

- docker-compose.yml — define los servicios y dependencias  
- .env.example — variables de entorno externalizadas  
- liquibase.properties.example — configuración de conexión de Liquibase  

## Justificación técnica

- Las variables sensibles se externalizan en `.env`, nunca se versionan credenciales
- El servicio liquibase depende de postgres garantizando orden de inicio
- Cualquier entorno levanta con un solo comando: `docker compose up`
- Es coherente con ADR-003 ya que Liquibase se ejecuta como contenedor

## Consecuencias e impacto esperado

- El entorno completo es reproducible en cualquier máquina con Docker instalado
- Se elimina la dependencia de instalaciones locales manuales
- El archivo `.env.example` documenta todas las variables necesarias sin exponer credenciales reales