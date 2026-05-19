# FleetLogix DB — Proyecto Integrador M2

## Descripción del proyecto

FleetLogix es una empresa de transporte y logística que opera una flota de 200 vehículos realizando entregas de última milla en 5 ciudades principales: Bogotá, Medellín, Cali, Barranquilla y Cartagena.

La empresa venía trabajando con sistemas legacy y hojas de cálculo. El objetivo de este proyecto es modernizar su infraestructura de datos diseñando, poblando y optimizando una base de datos relacional que permita gestionar operaciones a gran escala y tomar decisiones basadas en datos.

**Rol:** Científico de Datos Junior responsable de analizar el modelo relacional, generar datos sintéticos masivos y garantizar la calidad e integridad de la información.

---

## Análisis del Modelo Relacional

El modelo tiene 6 tablas que representan las operaciones de FleetLogix.

### ¿Qué guarda cada tabla?

- `vehicles` → los vehículos de la flota
- `drivers` → los conductores
- `routes` → las rutas entre ciudades
- `trips` → los viajes realizados (conecta vehículo + conductor + ruta)
- `deliveries` → las entregas individuales de cada viaje
- `maintenance` → el historial de mantenimiento de cada vehículo

### ¿Cómo se relacionan?

- Un vehículo puede hacer muchos viajes
- Un conductor puede hacer muchos viajes
- Una ruta puede usarse en muchos viajes
- Un viaje tiene muchas entregas
- Un vehículo tiene muchos registros de mantenimiento

### Claves primarias y foráneas

| Tabla | PK | FK |
|---|---|---|
| vehicles | vehicle_id | — |
| drivers | driver_id | — |
| routes | route_id | — |
| trips | trip_id | vehicle_id, driver_id, route_id |
| deliveries | delivery_id | trip_id |
| maintenance | maintenance_id | vehicle_id |

### Restricciones de integridad

- `license_plate` → UNIQUE NOT NULL → no puede haber dos vehículos con la misma patente
- `employee_code` → UNIQUE NOT NULL → código de empleado único
- `license_number` → UNIQUE NOT NULL → número de licencia único por conductor
- `route_code` → UNIQUE NOT NULL → código de ruta único
- `tracking_number` → UNIQUE NOT NULL → número de seguimiento único por entrega
- `status` en vehicles → DEFAULT 'active'
- `status` en trips → DEFAULT 'in_progress'
- `delivery_status` → DEFAULT 'pending'
- `recipient_signature` → DEFAULT FALSE

---

## Diagrama ER

Generado desde DBeaver conectado a la base de datos `fleetlogix` en PostgreSQL.

*(Ver archivo diagrama_ER.png)*

---

## Script de generación de datos

El script `A1-01_data_generation_estudiantes.py` genera los 505.569 registros respetando las relaciones del modelo y las reglas del negocio.

### Orden de generación

### generate_trips()

Genera los 100.000 viajes. Antes de generar lee de la base los IDs reales de vehículos, conductores y rutas para garantizar que las Foreign Keys sean válidas. Distribuye los viajes a lo largo de 2 años de operación histórica y calcula el consumo de combustible basado en la distancia de cada ruta.

### get_hourly_distribution()

Función auxiliar usada por `generate_trips()`. Define la probabilidad de que un viaje salga en cada hora del día — más viajes en horario laboral y menos de madrugada. Esto hace que los datos sean realistas en lugar de distribuirse de manera uniforme.

---

## Control de calidad

| Validación | Resultado |
|---|---|
| Trips sin vehículo válido | OK |
| Deliveries sin trip válido | OK |
| Trips con arrival < departure | OK |
| Trips excediendo capacidad del vehículo | OK |
| Entregas sin tracking number | OK |

Resumen guardado en `generation_summary.json`.

---

## Cómo ejecutar

1. Instalar dependencias:
```bash
pip install psycopg2-binary pandas numpy faker tqdm
```

2. Crear la base de datos en PostgreSQL y ejecutar el schema:
```bash
psql -U postgres -d fleetlogix -f fleetlogix_db_schema.sql
```

3. Ejecutar el script de generación:
```bash
python A1-01_data_generation_estudiantes.py
```
