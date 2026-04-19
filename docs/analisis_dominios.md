# Análisis de Dominios Funcionales — Modelo PostgreSQL

**Proyecto:** Sistema de Gestión Aeronáutica  
**Insumo base:** `modelo_postgresql.sql`  
**Total de dominios identificados:** 12  
**Total de tablas:** ~75  

---

## Características técnicas generales del modelo

- Todas las claves primarias usan `UUID` generado con `gen_random_uuid()` de la extensión `pgcrypto`
- Todos los registros tienen `created_at` y `updated_at` con `timestamptz`
- Se aplican restricciones `CHECK` para validación de datos en origen
- Se definen índices de apoyo sobre todas las claves foráneas relevantes
- El modelo mantiene 3FN — no hay totales derivados persistidos
- Los comentarios técnicos documentan decisiones de normalización importantes

---

## DOMINIO 1 — GEOGRAFÍA Y DATOS DE REFERENCIA

**Propósito:** Proveer la base territorial y monetaria que usan todos los demás dominios. Es el dominio más fundamental del modelo.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `time_zone` | Zonas horarias con offset UTC en minutos |
| `continent` | Continentes con código único de 3 caracteres |
| `country` | Países con códigos ISO alpha-2 e alpha-3, referencia a continente |
| `state_province` | Provincias/departamentos, referenciados a país |
| `city` | Ciudades con referencia a provincia y zona horaria |
| `district` | Barrios o distritos dentro de una ciudad |
| `address` | Dirección física completa con coordenadas geográficas (lat/long) |
| `currency` | Monedas con código ISO, símbolo y unidades decimales |

**Relaciones relevantes:**
- `continent` → `country` → `state_province` → `city` → `district` → `address` (cadena jerárquica territorial)
- `city` depende también de `time_zone` para la localización horaria
- `address` es consumida por `airport`, `maintenance_provider` y otros dominios

---

## DOMINIO 2 — AEROLÍNEA

**Propósito:** Representar la entidad operadora central del sistema. Casi todos los dominios operacionales referencian a esta entidad.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `airline` | Aerolínea con códigos IATA (2 chars) e ICAO (3 chars), país de origen |

**Restricciones notables:**
- `ck_airline_iata_len` y `ck_airline_icao_len` validan la longitud exacta de los códigos
- Unicidad garantizada sobre `airline_code`, `iata_code` e `icao_code`

**Relaciones relevantes:**
- Referencia a `country` para el país de origen
- Es referenciada por `aircraft`, `flight`, `customer`, `loyalty_program` y `fare`

---

## DOMINIO 3 — IDENTIDAD

**Propósito:** Modelar a toda persona natural que interactúa con el sistema, ya sea como cliente, pasajero o usuario operativo.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `person_type` | Tipos de persona (cliente, empleado, etc.) |
| `document_type` | Tipos de documento de identidad |
| `contact_type` | Tipos de contacto (email, teléfono, etc.) |
| `person` | Entidad central de persona con nombre, género, fecha de nacimiento |
| `person_document` | Documentos de identidad de una persona con vigencia |
| `person_contact` | Datos de contacto de una persona |

**Restricciones notables:**
- `ck_person_gender` valida que el género sea `'F'`, `'M'` o `'X'`
- `ck_person_document_dates` valida que fecha de expiración sea posterior a emisión
- `uq_person_document_natural` garantiza unicidad por tipo + país + número

**Relaciones relevantes:**
- `person` es la base de `user_account` (seguridad) y `customer` (fidelización)
- `person_document` y `person_contact` extienden la identidad sin romper normalización

---

## DOMINIO 4 — SEGURIDAD

**Propósito:** Controlar el acceso al sistema mediante cuentas de usuario, roles y permisos diferenciados.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `user_status` | Estados posibles de una cuenta (activo, suspendido, etc.) |
| `security_role` | Roles del sistema con código y descripción |
| `security_permission` | Permisos granulares con código único |
| `user_account` | Cuenta de usuario vinculada a una persona, con hash de contraseña |
| `user_role` | Asignación de roles a usuarios con trazabilidad de quién asignó |
| `role_permission` | Permisos asignados a cada rol |

**Restricciones notables:**
- `user_account.password_hash` almacena solo el hash, nunca la contraseña plana
- `user_role.assigned_by_user_id` permite trazabilidad completa de asignaciones
- `uq_user_role` y `uq_role_permission` evitan duplicados en asignaciones

**Relaciones relevantes:**
- `user_account` referencia a `person` (1 a 1)
- `user_account` es referenciada por `check_in` y `boarding_validation` para trazabilidad de operadores
- Modelo RBAC completo: `user` → `role` → `permission`

---

## DOMINIO 5 — CLIENTES Y FIDELIZACIÓN

**Propósito:** Gestionar la relación comercial con los clientes y sus programas de millas y beneficios.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `customer_category` | Categorías de cliente (frecuente, corporativo, etc.) |
| `benefit_type` | Tipos de beneficios ofrecidos |
| `loyalty_program` | Programa de fidelización de una aerolínea |
| `loyalty_tier` | Niveles dentro del programa (bronce, plata, oro) |
| `customer` | Cliente registrado ante una aerolínea específica |
| `loyalty_account` | Cuenta de millas del cliente en un programa |
| `loyalty_account_tier` | Historial de niveles asignados a la cuenta |
| `miles_transaction` | Movimientos de millas (acumulación, redención, ajuste) |
| `customer_benefit` | Beneficios asignados a un cliente con vigencia |

**Restricciones notables:**
- `ck_miles_transaction_type` valida: `'EARN'`, `'REDEEM'`, `'ADJUST'`
- `ck_miles_delta_non_zero` impide registrar transacciones con delta cero
- `loyalty_account_tier` es un historial — decisión de normalización documentada en COMMENTS

**Relaciones relevantes:**
- `customer` referencia tanto a `person` como a `airline`
- `loyalty_account` conecta `customer` con `loyalty_program`
- `miles_transaction` referencia `loyalty_account` para el movimiento de puntos

---

## DOMINIO 6 — AEROPUERTO

**Propósito:** Representar la infraestructura física aeroportuaria donde operan los vuelos.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `airport` | Aeropuerto con códigos IATA (3 chars) e ICAO (4 chars) |
| `terminal` | Terminales del aeropuerto |
| `boarding_gate` | Puertas de embarque dentro de cada terminal |
| `runway` | Pistas con longitud en metros y tipo de superficie |
| `airport_regulation` | Regulaciones vigentes del aeropuerto con autoridad emisora |

**Restricciones notables:**
- `ck_airport_iata_len` y `ck_airport_icao_len` validan longitud de códigos IATA/ICAO
- `ck_runway_length` asegura que la longitud sea mayor a cero
- `ck_airport_regulation_dates` valida vigencia de la regulación

**Relaciones relevantes:**
- `airport` referencia a `address` (dominio de geografía)
- `terminal` → `boarding_gate` (jerarquía interna del aeropuerto)
- `airport` es referenciada por `flight_segment` como origen y destino, y por `fare`

---

## DOMINIO 7 — AERONAVES

**Propósito:** Modelar la flota de aeronaves, su configuración de cabinas, asientos y el historial de mantenimiento.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `aircraft_manufacturer` | Fabricantes de aeronaves (Boeing, Airbus, etc.) |
| `aircraft_model` | Modelos de aeronave con alcance máximo en km |
| `cabin_class` | Clases de cabina (económica, ejecutiva, primera) |
| `aircraft` | Aeronave concreta con número de registro y serie |
| `aircraft_cabin` | Cabinas configuradas en una aeronave específica |
| `aircraft_seat` | Asientos individuales con fila, columna y características |
| `maintenance_provider` | Proveedores de mantenimiento |
| `maintenance_type` | Tipos de mantenimiento |
| `maintenance_event` | Eventos de mantenimiento con fechas y proveedor |

**Restricciones notables:**
- `ck_aircraft_service_dates` valida que la fecha de retiro sea posterior al servicio
- `uq_aircraft_seat_position` garantiza que no existan dos asientos iguales en la misma cabina
- `ck_aircraft_cabin_deck` valida que el número de cubierta sea positivo

**Relaciones relevantes:**
- `aircraft` referencia a `airline` y a `aircraft_model`
- `aircraft_cabin` conecta `aircraft` con `cabin_class`
- `aircraft_seat` es referenciada por `seat_assignment` en ventas/reservas
- `aircraft` es referenciada por `flight`

---

## DOMINIO 8 — OPERACIONES DE VUELO

**Propósito:** Gestionar los vuelos programados, sus segmentos, tarifas y los registros de demoras.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `flight_status` | Estados del vuelo (programado, en vuelo, aterrizado, cancelado) |
| `delay_reason_type` | Categorías de razones de demora |
| `fare_class` | Clases tarifarias vinculadas a cabina |
| `fare` | Tarifas con vigencia, penalizaciones y allowance de equipaje |
| `flight` | Vuelo como instancia única por aerolínea + número + fecha |
| `flight_segment` | Tramo individual del vuelo con aeropuertos y tiempos reales/programados |
| `flight_delay` | Registro de demoras con minutos y razón |

**Restricciones notables:**
- `ck_flight_segment_airports` impide que origen y destino sean el mismo aeropuerto
- `ck_flight_segment_schedule` valida que llegada programada sea posterior a salida
- `ck_flight_delay_minutes` asegura que la demora registrada sea mayor a cero
- `uq_flight_instance` garantiza unicidad por aerolínea + número + fecha de servicio

**Relaciones relevantes:**
- `flight` referencia a `airline`, `aircraft` y `flight_status`
- `flight_segment` referencia a `flight` y a `airport` (x2: origen y destino)
- `fare` referencia a `airline`, `airport` (x2), `fare_class` y `currency`

---

## DOMINIO 9 — VENTAS Y RESERVAS

**Propósito:** Gestionar el flujo comercial completo: desde la reserva hasta la emisión de tiquetes y la asignación de asientos.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `reservation_status` | Estados de reserva (confirmada, pendiente, cancelada) |
| `sale_channel` | Canal de venta (web, app, mostrador, agencia) |
| `reservation` | Reserva raíz del flujo comercial con código PNR |
| `reservation_passenger` | Pasajeros incluidos en una reserva |
| `sale` | Venta concreta derivada de una reserva |
| `ticket_status` | Estados del tiquete |
| `ticket` | Tiquete emitido por pasajero y venta |
| `ticket_segment` | Tabla puente entre tiquete y segmentos de vuelo (soporta escalas) |
| `seat_assignment` | Asignación de asiento por segmento con fuente (auto/manual/cliente) |
| `baggage` | Equipaje registrado por segmento de tiquete |

**Restricciones notables:**
- `ck_reservation_passenger_type` valida: `'ADULT'`, `'CHILD'`, `'INFANT'`
- `ck_seat_assignment_source` valida: `'AUTO'`, `'MANUAL'`, `'CUSTOMER'`
- `ck_baggage_type` valida: `'CHECKED'`, `'CARRY_ON'`, `'SPECIAL'`
- `ck_baggage_status` valida: `'REGISTERED'`, `'LOADED'`, `'CLAIMED'`, `'LOST'`
- `seat_assignment` usa FK compuesta hacia `ticket_segment` para garantizar coherencia de segmento

**Relaciones relevantes:**
- `reservation` es la entidad raíz documentada explícitamente en COMMENTS
- `ticket_segment` es tabla puente documentada — soporta itinerarios con escalas
- `seat_assignment` referencia a `aircraft_seat` cerrando el ciclo operativo

---

## DOMINIO 10 — ABORDAJE

**Propósito:** Gestionar el proceso de check-in, emisión de pases de abordar y validación en puerta.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `boarding_group` | Grupos de abordaje con secuencia de prioridad |
| `check_in_status` | Estados del check-in |
| `check_in` | Registro del check-in por segmento de tiquete |
| `boarding_pass` | Pase de abordar con código de barras único |
| `boarding_validation` | Validación del pase en puerta con resultado |

**Restricciones notables:**
- `uq_check_in_ticket_segment` garantiza un único check-in por segmento
- `uq_boarding_pass_check_in` garantiza un único pase por check-in
- `ck_boarding_validation_result` valida: `'APPROVED'`, `'REJECTED'`, `'MANUAL_REVIEW'`

**Relaciones relevantes:**
- `check_in` referencia a `ticket_segment` y a `user_account` (quién realizó el check-in)
- `boarding_validation` referencia a `boarding_gate` y a `user_account` (operador)
- Flujo secuencial: `check_in` → `boarding_pass` → `boarding_validation`

---

## DOMINIO 11 — PAGOS

**Propósito:** Registrar y trazar el ciclo completo de transacciones financieras de una venta, incluyendo reembolsos.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `payment_status` | Estados del pago (pendiente, aprobado, rechazado) |
| `payment_method` | Métodos de pago (tarjeta, PSE, efectivo, etc.) |
| `payment` | Pago asociado a una venta con monto y referencia |
| `payment_transaction` | Transacciones individuales de un pago (auth, capture, void, refund) |
| `refund` | Reembolsos solicitados con fecha de procesamiento |

**Restricciones notables:**
- `ck_payment_transaction_type` valida: `'AUTH'`, `'CAPTURE'`, `'VOID'`, `'REFUND'`, `'REVERSAL'`
- `ck_payment_amount` y `ck_refund_amount` garantizan montos positivos
- `ck_refund_dates` valida que la fecha de procesamiento sea posterior o igual a la solicitud

**Relaciones relevantes:**
- `payment` referencia a `sale` (ventas) y a `currency`
- `payment_transaction` extiende `payment` con el detalle de cada movimiento financiero
- `refund` referencia a `payment` para trazabilidad del reembolso

---

## DOMINIO 12 — FACTURACIÓN

**Propósito:** Emitir facturas formales asociadas a ventas, aplicando impuestos y tasas de cambio.

**Tablas:**

| Tabla | Descripción |
|---|---|
| `tax` | Impuestos con porcentaje y vigencia |
| `exchange_rate` | Tasas de cambio entre monedas por fecha |
| `invoice_status` | Estados de factura (emitida, anulada, pagada) |
| `invoice` | Factura vinculada a una venta con fecha de vencimiento |
| `invoice_line` | Líneas de detalle de la factura sin totales persistidos |

**Restricciones notables:**
- `ck_tax_rate` garantiza porcentaje no negativo
- `ck_exchange_rate_pair` impide que origen y destino de moneda sean iguales
- `ck_invoice_line_unit_price` permite precio unitario cero (descuentos totales)
- `invoice_line` documentada en COMMENTS como decisión de 3FN (sin totales derivados)

**Relaciones relevantes:**
- `invoice` referencia a `sale`, `invoice_status` y `currency`
- `invoice_line` referencia a `invoice` y opcionalmente a `tax`
- `exchange_rate` conecta dos instancias de `currency` con una fecha efectiva

---

## MAPA DE DEPENDENCIAS ENTRE DOMINIOS

```
GEOGRAFÍA ──────────────────────────────────────────┐
    │                                               │
    ▼                                               ▼
AEROLÍNEA          IDENTIDAD ──────► SEGURIDAD
    │                  │
    │                  ▼
    ├─────────► CLIENTES & FIDELIZACIÓN
    │
    ▼
AEROPUERTO ◄──── OPERACIONES DE VUELO ◄──── AERONAVES
                        │
                        ▼
                VENTAS & RESERVAS ──────► ABORDAJE
                        │
                        ▼
                    PAGOS ──────► FACTURACIÓN
```

---

## Observaciones técnicas del modelo

- El modelo evidencia separación clara de responsabilidades entre dominios funcionales.
- Se evitan dependencias transitivas — los totales calculables no se persisten.
- El uso de `timestamptz` en todos los campos temporales garantiza consistencia ante cambios de zona horaria.
- Los índices están definidos sistemáticamente sobre todas las FK, lo que indica diseño orientado a rendimiento de consultas.
- Las restricciones `CHECK` cubren validaciones de negocio en capa de base de datos, no delegadas a la aplicación.
- El dominio de **Seguridad** implementa un modelo RBAC estándar y completo.