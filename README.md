# TheBrotherhood — API de gestión de citas

REST API para la gestión de citas de la barbería **TheBrotherhood**

**Proyecto 03 — Desarrollo de Aplicaciones Web**
Integrantes: Alejandro Tapia · Jeremías Cabot

## Problema

La barbería gestiona sus citas de forma manual (WhatsApp y cuaderno), lo que produce cruces de
horario entre barberos, olvidos de reservas y ausencia de un registro ordenado de clientes y
servicios. Esta API centraliza esa gestión.

## Recursos

| Recurso | Descripción |
|---------|-------------|
| `/clientes` | Personas que reservan un servicio. |
| `/barberos` | Profesionales que atienden las citas. |
| `/servicios` | Prestaciones ofrecidas, con precio y duración. |
| `/citas` | Reserva que asocia un cliente, un barbero, un servicio y un horario. |

## Endpoints

### Clientes (RF-01)
| Método | Ruta | Descripción | Éxito |
|--------|------|-------------|-------|
| POST | `/clientes/` | Registrar cliente | 201 |
| GET | `/clientes/` | Listar clientes | 200 |
| GET | `/clientes/{id_cliente}` | Consultar cliente | 200 |
| PUT | `/clientes/{id_cliente}` | Actualizar cliente | 200 |
| DELETE | `/clientes/{id_cliente}` | Eliminar cliente | 200 |

### Barberos (RF-02)
| Método | Ruta | Descripción | Éxito |
|--------|------|-------------|-------|
| POST | `/barberos/` | Registrar barbero | 201 |
| GET | `/barberos/` | Listar barberos | 200 |
| GET | `/barberos/{id_barbero}` | Consultar barbero | 200 |
| PUT | `/barberos/{id_barbero}` | Actualizar barbero | 200 |
| DELETE | `/barberos/{id_barbero}` | Eliminar barbero | 200 |

### Servicios (RF-03, RF-04)
| Método | Ruta | Descripción | Éxito |
|--------|------|-------------|-------|
| POST | `/servicios/` | Registrar servicio | 201 |
| GET | `/servicios/` | Consultar catálogo | 200 |
| GET | `/servicios/{id_servicio}` | Consultar servicio | 200 |
| PUT | `/servicios/{id_servicio}` | Actualizar servicio | 200 |
| DELETE | `/servicios/{id_servicio}` | Eliminar servicio | 200 |

### Citas (RF-05 a RF-09)
| Método | Ruta | Descripción | Éxito |
|--------|------|-------------|-------|
| POST | `/citas/` | Agendar cita (valida disponibilidad) | 201 |
| GET | `/citas/` | Listar y filtrar citas | 200 |
| GET | `/citas/{id_cita}` | Consultar cita | 200 |
| PUT | `/citas/{id_cita}` | Reprogramar cita | 200 |
| PATCH | `/citas/{id_cita}/estado` | Cambiar estado | 200 |
| DELETE | `/citas/{id_cita}` | Cancelar cita | 200 |

Filtros de `GET /citas/`: `id_barbero`, `id_cliente`, `fecha` (AAAA-MM-DD), `estado`,
más paginación con `skip` y `limit`.

### Códigos de respuesta

| Código | Situación |
|--------|-----------|
| 200 | Consulta, actualización o eliminación correcta. |
| 201 | Recurso creado. |
| 404 | El recurso solicitado o referenciado no existe. |
| 409 | El barbero ya tiene una cita agendada en ese horario (RF-07). |
| 422 | Datos de entrada inválidos (campos obligatorios, formato de correo o fecha). |

## Base de datos

Supabase (PostgreSQL), accedido mediante SQLAlchemy sobre una conexión Postgres.

Las tablas del proyecto viven en el esquema **`daw`**. La API se conecta con el rol dedicado
`daw_api`, cuyos privilegios se limitan a ese esquema, de modo que el aislamiento respecto al
resto de la base de datos lo impone PostgreSQL y no la lógica de la aplicación.

- `db/schema.sql` — creación de tablas, índices, RLS, rol y permisos.
- `db/rollback.sql` — revierte por completo los cambios del proyecto.

La regla de disponibilidad (RF-07) se garantiza en dos niveles: la API verifica el horario antes
de insertar, y un índice único parcial (`cita_barbero_horario_unico`) lo impide a nivel de base
de datos incluso ante peticiones simultáneas.

## Ejecución local

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env   # completar con la cadena de conexión

fastapi dev main.py
```

Documentación interactiva (Swagger): http://127.0.0.1:8000/docs

### Variables de entorno

| Variable | Descripción |
|----------|-------------|
| `DATABASE_URL` | Cadena de conexión a PostgreSQL (Supabase). |

Formato:

```
postgresql+psycopg://<usuario>:<password>@<host>:5432/postgres
```

## Pruebas

Con la API en ejecución:

```bash
python tests/prueba_endpoints.py http://127.0.0.1:8000
```

Recorre los requisitos funcionales de extremo a extremo y verifica que cada uno responda con el
código HTTP esperado, incluidos los casos de error (correo inválido, precio negativo, horario
ocupado, recurso inexistente).

## Despliegue

Desplegado en FastAPI Cloud desde este repositorio de GitHub. Las variables de entorno se
configuran en la plataforma.
