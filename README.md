# airline-db-project

Repositorio técnico del proyecto **Sistema de Gestión Aeronáutica**.  
Contiene los changelogs de Liquibase (DDL y DML), la configuración de contenedores Docker y la estructura de versionamiento del modelo de base de datos PostgreSQL.


## Repositorio principal del proyecto evaluativo 
https://github.com/valerysinai/airline-db-project


> La documentación del proyecto (análisis de dominios, ADR, backlog, plan de datos de prueba y seguimientos) se encuentra en el repositorio complementario **airline-db-docs**.

---

## Estructura del repositorio

```
airline-db-project/
├── changelog-master.yaml          # Punto de entrada de Liquibase — referencia todos los changelogs
├── docker-compose.yml             # Servicios PostgreSQL y Liquibase
├── liquibase.properties           # Configuración de conexión de Liquibase (no versionar credenciales reales)
├── .env.example                   # Variables de entorno requeridas
├── 01_ddl/                        # Definición de estructura (DDL)
│   ├── 00_extensions/             # Extensiones PostgreSQL (pgcrypto)
│   ├── 03_tables/                 # Tablas organizadas por dominio funcional
│   │   ├── 01_geography/
│   │   ├── 02_airline/
│   │   ├── 03_identity/
│   │   ├── 04_security/
│   │   ├── 05_customer_loyalty/
│   │   ├── 06_airport/
│   │   ├── 07_aircraft/
│   │   ├── 08_flight_operations/
│   │   ├── 09_sales_reservations/
│   │   ├── 10_boarding/
│   │   ├── 11_payment/
│   │   └── 12_billing/
│   └── 09_indexes/                # Índices de apoyo sobre claves foráneas
├── 02_dml/
│   └── 00_inserts/                # Scripts de inserción de datos de prueba por dominio
│       ├── 01_geography/
│       ├── 02_airline/
│       ├── 03_identity/
│       ├── 04_security/
│       ├── 05_customer_loyalty/
│       ├── 06_airport/
│       ├── 07_aircraft/
│       ├── 08_flight_operations/
│       ├── 09_sales_reservations/
│       ├── 10_boarding/
│       ├── 11_payment/
│       └── 12_billing/
├── 03_dcl/
│   ├── 00_roles/                  # Definición de roles de base de datos
│   └── 01_grants/                 # Grants por rol
├── 04_tcl/                        # Control de transacciones (reservado)
└── 05_rollbacks/                  # Scripts de rollback (reservado)
```

---

## Requisitos previos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado y en ejecución
- Git

---

## Configuración inicial

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd airline-db-project
```

### 2. Crear el archivo `.env`

Copiar el archivo de ejemplo y ajustar las variables:

```bash
cp .env.example .env
```

Contenido de `.env`:

```env
POSTGRES_DB=airline_db
POSTGRES_USER=airline_user
POSTGRES_PASSWORD=airline_pass
POSTGRES_PORT=5432
```

---

## Levantar el entorno

```bash
docker compose up -d
```

Esto levanta dos servicios:

| Servicio    | Imagen                    | Puerto | Descripción                        |
|-------------|---------------------------|--------|------------------------------------|
| `postgres`  | `postgres:15`             | 5432   | Instancia de base de datos         |
| `liquibase` | `liquibase/liquibase:4.27`| —      | Ejecutor de changelogs             |

PostgreSQL incluye un healthcheck — Liquibase espera a que la base de datos esté lista antes de iniciar.

---

## Ejecutar los changelogs con Liquibase

Una vez los contenedores estén corriendo, ejecutar el siguiente comando para aplicar todos los changesets (DDL + DML):

```bash
docker exec -w /liquibase/changelog airline_liquibase liquibase \
  --url=jdbc:postgresql://postgres:5432/airline_db \
  --username=airline_user \
  --password=airline_pass \
  --changeLogFile=changelog-master.yaml \
  update
```

Una ejecución exitosa muestra:

```
Liquibase: Update has been successful.
```

---

## Estrategia de versionamiento (ramas)

```
main       ← versión estable, solo recibe merges validados desde qa
 └── qa    ← validación e integración
      └── develop  ← rama de trabajo activo
```


## Dominios funcionales del modelo

El modelo de base de datos cubre 12 dominios funcionales:

| # | Dominio                  | Tablas principales                                      |
|---|--------------------------|----------------------------------------------------------|
| 1 | Geografía y referencia   | continent, country, city, address, currency              |
| 2 | Aerolínea                | airline                                                  |
| 3 | Identidad                | person, person_document, person_contact                  |
| 4 | Seguridad                | user_account, security_role, role_permission             |
| 5 | Clientes y fidelización  | customer, loyalty_account, miles_transaction             |
| 6 | Aeropuerto               | airport, terminal, boarding_gate, runway                 |
| 7 | Aeronaves                | aircraft, aircraft_seat, maintenance_event               |
| 8 | Operaciones de vuelo     | flight, flight_segment, fare, flight_delay               |
| 9 | Ventas y reservas        | reservation, ticket, ticket_segment, seat_assignment     |
|10 | Abordaje                 | check_in, boarding_pass, boarding_validation             |
|11 | Pagos                    | payment, payment_transaction, refund                     |
|12 | Facturación              | invoice, invoice_line, tax, exchange_rate                |

---

## Roles de base de datos

| Rol               | Permisos                                              |
|------------------|-------------------------------------------------------|
| `role_readonly`   | SELECT sobre todas las tablas                         |
| `role_operations` | SELECT, INSERT, UPDATE sobre tablas operativas        |
| `role_admin`      | SELECT, INSERT, UPDATE, DELETE sobre todas las tablas |

Los roles y grants se gestionan mediante changelogs en `03_dcl/`.

---

## Detener el entorno

```bash
docker compose down
```

Para eliminar también los volúmenes (datos persistidos):

```bash
docker compose down -v
```

---

## Programa de formación

**SENA — Análisis y Desarrollo de Software**  
Proyecto formativo: Desarrollo de software orientado a servicios  
Competencia: 220501096 — Construir la base de datos para el software a partir del modelo de datos
