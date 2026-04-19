# Plan de Datos de Prueba — Sistema de Gestión Aeronáutica

**Proyecto:** Sistema de Gestión Aeronáutica  
**Insumo base:** `modelo_postgresql.sql`  

> **Nota:** Los scripts de inserción definitivos serán suministrados por el instructor. Este plan define el orden lógico y las validaciones esperadas.

---

## Principios del plan

- Los datos se insertan respetando el orden de dependencias entre tablas (de menor a mayor dependencia)
- Cada dominio se carga de forma completa antes de pasar al siguiente
- Se valida la integridad referencial después de cada grupo de inserciones
- Los datos son controlados, mínimos y representativos — no masivos
- Los scripts se organizan en `02_dml/00_inserts/` por dominio funcional

---

## Orden de inserción por dominio

---

## FASE 1 — Datos de referencia base (sin dependencias externas)

**Dominio:** Geografía y Referencia  

| Orden | Tabla            | Motivo                                      |
|------|------------------|----------------------------------------------|
| 1    | time_zone        | Sin dependencias                             |
| 2    | continent        | Sin dependencias                             |
| 3    | currency         | Sin dependencias                             |
| 4    | country          | Depende de continent                         |
| 5    | state_province   | Depende de country                           |
| 6    | city             | Depende de state_province y time_zone        |
| 7    | district         | Depende de city                              |
| 8    | address          | Depende de district                          |

**Validaciones:**
- Verificar que los códigos ISO de `country` sean únicos (alpha-2 y alpha-3)
- Verificar que `city` tenga referencia válida a `time_zone`
- Verificar que `address` tenga coordenadas dentro del rango CHECK (-90/90, -180/180)

---

## FASE 2 — Entidad operadora central

**Dominio:** Aerolínea  

| Orden | Tabla   | Motivo                 |
|------|----------|------------------------|
| 9    | airline  | Depende de country     |

**Validaciones:**
- Verificar longitud exacta de `iata_code` (2 chars) e `icao_code` (3 chars)
- Verificar unicidad de `airline_code`

---

## FASE 3 — Identidad de personas

**Dominio:** Identidad  

| Orden | Tabla            | Motivo                                      |
|------|------------------|----------------------------------------------|
| 10   | person_type      | Sin dependencias                             |
| 11   | document_type    | Sin dependencias                             |
| 12   | contact_type     | Sin dependencias                             |
| 13   | person           | Depende de person_type y country             |
| 14   | person_document  | Depende de person y document_type            |
| 15   | person_contact   | Depende de person y contact_type             |

**Validaciones:**
- Verificar que `gender_code` sea 'F', 'M' o 'X'
- Verificar que `expires_on` sea posterior a `issued_on`
- Verificar unicidad del documento por tipo + país + número

---

## FASE 4 — Seguridad y accesos

**Dominio:** Seguridad  

| Orden | Tabla                | Motivo                                      |
|------|----------------------|----------------------------------------------|
| 16   | user_status          | Sin dependencias                             |
| 17   | security_role        | Sin dependencias                             |
| 18   | security_permission  | Sin dependencias                             |
| 19   | user_account         | Depende de person y user_status              |
| 20   | user_role            | Depende de user_account y security_role      |
| 21   | role_permission      | Depende de security_role y security_permission |

**Validaciones:**
- Verificar que `password_hash` no esté vacío ni sea texto plano
- Verificar unicidad de `username`
- Verificar que no existan roles duplicados por usuario

---

## FASE 5 — Clientes y fidelización

**Dominio:** Clientes y Fidelización  

| Orden | Tabla                    | Motivo                                      |
|------|--------------------------|----------------------------------------------|
| 22   | customer_category        | Sin dependencias                             |
| 23   | benefit_type             | Sin dependencias                             |
| 24   | loyalty_program          | Depende de airline y currency                |
| 25   | loyalty_tier             | Depende de loyalty_program                   |
| 26   | customer                 | Depende de airline, person y category        |
| 27   | loyalty_account          | Depende de customer y loyalty_program        |
| 28   | loyalty_account_tier     | Depende de loyalty_account y loyalty_tier    |
| 29   | miles_transaction        | Depende de loyalty_account                   |
| 30   | customer_benefit         | Depende de customer y benefit_type           |

**Validaciones:**
- Verificar que `miles_delta` no sea cero
- Verificar que `transaction_type` sea 'EARN', 'REDEEM' o 'ADJUST'
- Verificar unicidad de cliente por aerolínea + persona

---

## FASE 6 — Infraestructura aeroportuaria

**Dominio:** Aeropuerto  

| Orden | Tabla               | Motivo                 |
|------|---------------------|------------------------|
| 31   | airport             | Depende de address     |
| 32   | terminal            | Depende de airport     |
| 33   | boarding_gate       | Depende de terminal    |
| 34   | runway              | Depende de airport     |
| 35   | airport_regulation  | Depende de airport     |

**Validaciones:**
- Verificar longitud de `iata_code` (3) e `icao_code` (4)
- Verificar que `length_meters` > 0
- Verificar que fechas de regulación sean coherentes

---

## FASE 7 — Flota de aeronaves

**Dominio:** Aeronaves  

| Orden | Tabla                  | Motivo                                      |
|------|------------------------|----------------------------------------------|
| 36   | aircraft_manufacturer  | Sin dependencias                             |
| 37   | aircraft_model         | Depende de manufacturer                      |
| 38   | cabin_class            | Sin dependencias                             |
| 39   | aircraft               | Depende de airline y model                   |
| 40   | aircraft_cabin         | Depende de aircraft y cabin_class            |
| 41   | aircraft_seat          | Depende de aircraft_cabin                    |
| 42   | maintenance_provider   | Depende de address                           |
| 43   | maintenance_type       | Sin dependencias                             |
| 44   | maintenance_event      | Depende de aircraft, type y provider         |

**Validaciones:**
- Verificar unicidad de asiento (fila + columna)
- Verificar coherencia de fechas de servicio
- Verificar que `max_range_km` > 0 si existe

---

## FASE 8 — Operaciones de vuelo

**Dominio:** Operaciones de Vuelo  

| Orden | Tabla               | Motivo                                      |
|------|---------------------|----------------------------------------------|
| 45   | flight_status       | Sin dependencias                             |
| 46   | delay_reason_type   | Sin dependencias                             |
| 47   | fare_class          | Depende de cabin_class                       |
| 48   | fare                | Depende de airline, airport, class, currency |
| 49   | flight              | Depende de airline, aircraft y status        |
| 50   | flight_segment      | Depende de flight y airport                  |
| 51   | flight_delay        | Depende de segment y delay_reason            |

**Validaciones:**
- Origen ≠ destino
- Fechas coherentes (arrival > departure)
- `delay_minutes` > 0

---

## FASE 9 — Ventas y reservas

**Dominio:** Ventas y Reservas  

| Orden | Tabla                   | Motivo                                      |
|------|-------------------------|----------------------------------------------|
| 52   | reservation_status      | Sin dependencias                             |
| 53   | sale_channel            | Sin dependencias                             |
| 54   | ticket_status           | Sin dependencias                             |
| 55   | reservation             | Depende de customer                          |
| 56   | reservation_passenger   | Depende de reservation                       |
| 57   | sale                    | Depende de reservation y currency            |
| 58   | ticket                  | Depende de sale y pasajero                   |
| 59   | ticket_segment          | Depende de ticket y flight_segment           |
| 60   | seat_assignment         | Depende de ticket_segment y aircraft_seat    |
| 61   | baggage                 | Depende de ticket_segment                    |

**Validaciones:**
- Tipos de pasajero válidos
- Unicidad por pasajero en reserva
- Peso de equipaje > 0

---

## FASE 10 — Abordaje

**Dominio:** Abordaje  

| Orden | Tabla                | Motivo                                      |
|------|----------------------|----------------------------------------------|
| 62   | boarding_group       | Sin dependencias                             |
| 63   | check_in_status      | Sin dependencias                             |
| 64   | check_in             | Depende de ticket_segment                    |
| 65   | boarding_pass        | Depende de check_in                          |
| 66   | boarding_validation  | Depende de boarding_pass y gate              |

**Validaciones:**
- Unicidad de check-in por segmento
- Unicidad de boarding pass
- Valores válidos de validación

---

## FASE 11 — Pagos

**Dominio:** Pagos  

| Orden | Tabla                | Motivo                                      |
|------|----------------------|----------------------------------------------|
| 67   | payment_status       | Sin dependencias                             |
| 68   | payment_method       | Sin dependencias                             |
| 69   | payment              | Depende de sale                              |
| 70   | payment_transaction  | Depende de payment                           |
| 71   | refund               | Depende de payment                           |

**Validaciones:**
- `amount` > 0
- Tipos de transacción válidos
- Coherencia de fechas en refund

---

## FASE 12 — Facturación

**Dominio:** Facturación  

| Orden | Tabla          | Motivo                        |
|------|----------------|-------------------------------|
| 72   | tax            | Sin dependencias              |
| 73   | exchange_rate  | Depende de currency (x2)      |
| 74   | invoice_status | Sin dependencias              |
| 75   | invoice        | Depende de sale               |
| 76   | invoice_line   | Depende de invoice y tax      |

**Validaciones:**
- `rate_percentage` ≥ 0
- Monedas distintas en exchange_rate
- Fechas coherentes en invoice
- Validación de cantidad y precio