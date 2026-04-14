# Análisis de Dominios Funcionales

Se realizó un análisis del modelo relacional contenido en el archivo `modelo_postgresql.sql`, con el objetivo de identificar dominios funcionales que permitan organizar el sistema de manera modular.

## Dominios identificados

- Geografía
- Aerolínea
- Identidad
- Seguridad
- Clientes y lealtad
- Aeropuerto
- Aeronaves
- Operaciones de vuelo
- Ventas y reservas
- Abordaje
- Pagos
- Facturación

## Relaciones relevantes

- Cliente → Reserva → Vuelo → Pago
- Aeronave → Vuelo → Tripulación
- Usuario → Rol → Permisos

## Conclusión

La identificación de dominios permite:
- Organizar el versionamiento con Liquibase
- Mejorar mantenibilidad
- Separar responsabilidades