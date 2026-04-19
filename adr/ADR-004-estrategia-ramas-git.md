# ADR-004-versionamiento-ramas.md

## Título
Estrategia de versionamiento del repositorio con ramas develop, qa y main.

## Contexto
El proyecto tiene dos repositorios: airline-db-docs y airline-db-project. Sin una estrategia de ramas definida, los cambios pueden aplicarse de forma desordenada, sin revisión y sin trazabilidad entre entornos.

## Problema
Sin flujo de ramas controlado, cualquier cambio puede llegar directamente a producción sin pasar por validación. Esto genera riesgo de inconsistencias entre entornos y dificulta la revisión técnica del trabajo.

## Decisión
Se define la siguiente estrategia de tres ramas en ambos repositorios:

Rama | Propósito  
develop | Rama de trabajo activo. Todo desarrollo inicia aquí  
qa | Rama de validación. Recibe merges desde develop para revisión  
main | Rama estable. Solo recibe merges validados desde qa  

Flujo de trabajo:
develop → qa → main

Convención de commits:
[HU-XXX] Descripción corta del cambio

## Justificación técnica

Separa claramente los entornos de desarrollo, validación y producción  
Ningún cambio llega a main sin pasar por revisión en qa  
Los commits referenciados a HU permiten trazabilidad directa con el backlog  
Es compatible con GitHub Projects para seguimiento del backlog técnico  

## Consecuencias e impacto esperado

Se garantiza que main siempre tiene una versión estable y funcional  
Facilita la identificación de en qué rama ocurrió un problema  
El historial de commits refleja el avance real del proyecto por historia de usuario  