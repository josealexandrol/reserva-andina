# ReservaAndina

**Plataforma de reservas de turismo vivencial para comunidades del Valle Sagrado (Cusco, Perú)**

Proyecto grupal del curso **Cloud Computing** — Producto académico N.º 3.
Arquitectura basada en microservicios con Node.js, PostgreSQL, Docker, Docker Compose y GitHub Actions.

---

## Integrantes

| N.º | Integrante | Rol | Microservicio a cargo | Tarea adicional |
|---|---|---|---|---|
| 1 | JOSE ALEXANDRO LOPEZ MAMANI | Líder del repositorio | Anfitriones | Revisión y fusión de Pull Requests, README |
| 2 | BRANDON CUNZA ROBLES | Colaborador y revisor | Experiencias | Docker Compose; revisión de los PR del líder |
| 3 | ROBERTO ISRAEL MEZA ORORE | Desarrollador | Disponibilidad | — |
| 4 | MARIANO JHOSHUA LAURA PAZ | Desarrollador | Reservas | Orquestación entre servicios |
| 5 | OSCAR AILTHON ESPINOZA NIETO | Desarrollador | Notificaciones | Integración continua (GitHub Actions) |

---|---|---|---|
| 1 | JOSE ALEXANDRO LOPEZ MAMANI | Líder del repositorio | Por definir |
| 2 | BRANDON CUNZA ROBLES | Desarrollador | Por definir |
| 3 | ROBERTO ISRAEL MEZA ORORE | Desarrollador | Por definir |
| 4 | MARIANO JHOSHUA LAURA PAZ | Desarrollador | Por definir |
| 5 | OSCAR AILTHON ESPINOZA NIETO | Desarrollador | Por definir |

---

## 1. Contexto y problemática

**Lugar.** Comunidades rurales del Valle Sagrado de los Incas, en las provincias de Calca y Urubamba (Cusco), como las de Pisac, Chinchero, Ollantaytambo y Huilloc. Estas comunidades reciben turistas que visitan el valle camino a Machu Picchu.

**Necesidad.** Muchas familias ofrecen **turismo vivencial**: hospedaje en casa, talleres de tejido con técnicas tradicionales, gastronomía andina y labores agrícolas. Sin embargo:

- **Dependen de intermediarios** (agencias o guías), que se quedan con buena parte del pago.
- **Gestionan sus reservas por WhatsApp, llamadas o un cuaderno**, lo que causa sobrerreservas, olvidos y cancelaciones sin aviso.
- **El turista no encuentra la oferta directa** ni sabe si hay cupo para su fecha.
- **La asociación comunitaria no sabe cuánto turismo recibe** cada familia, así que no puede distribuir los turnos de forma equitativa.

### Condiciones reales de los usuarios

| Aspecto | Anfitriones (familias) | Turistas |
|---|---|---|
| Internet | Datos móviles intermitentes; señal débil en algunas zonas | Buena conexión (wifi del hotel o datos) |
| Dispositivo | Celular básico o smartphone de gama baja | Smartphone o laptop |
| Hábitos | Usan WhatsApp y llamadas; poca experiencia con apps | Acostumbrados a reservar en línea |
| Idioma | Quechua y español | Español e inglés |

**Decisiones de diseño derivadas de estas condiciones:**

- **El anfitrión no necesita instalar ni aprender una app:** recibe las reservas como mensaje (SMS o WhatsApp) y las confirma con una respuesta simple.
- **El turista usa una web ligera**, que carga rápido incluso con mala señal.
- **La asociación comunitaria** usa un panel con reportes desde una computadora del local comunal.

---

## 2. Usuarios objetivo

1. **Familias anfitrionas:** publican sus experiencias y reciben reservas.
2. **Asociación de turismo comunitario:** registra a las familias, supervisa las reservas y reparte los turnos.
3. **Turistas nacionales y extranjeros:** buscan, reservan y cancelan experiencias.

---

## 3. Propuesta de solución

Un sistema web basado en **microservicios**, donde:

1. La **asociación** registra a las familias anfitrionas y sus experiencias, con precio y capacidad.
2. Cada experiencia tiene un **calendario de cupos por fecha**.
3. El **turista** busca una experiencia, elige la fecha y el número de personas, y reserva.
4. El sistema **valida** que la experiencia exista y que haya cupos suficientes, y **descuenta los cupos**, de modo que no hay sobrerreservas.
5. El **anfitrión recibe una notificación** con los datos de la reserva.
6. La reserva puede **confirmarse o cancelarse**. Al cancelarse, los cupos se **liberan automáticamente** y se notifica al anfitrión.
7. La asociación consulta las reservas por familia y fecha.

**Beneficios:** venta directa sin intermediarios; ninguna sobrerreserva, porque los cupos se controlan automáticamente; el anfitrión trabaja con lo que ya usa (mensajes); y la asociación cuenta con un registro ordenado.

---

## 4. Microservicios

### Arquitectura general

```
                         ┌───────────────────┐
                         │      Turista      │
                         └─────────┬─────────┘
                                   │  REST
                                   ▼
┌──────────────┐  valida   ┌──────────────┐  reserva/libera cupos  ┌────────────────┐
│ Experiencias │◄──────────│   Reservas   │───────────────────────►│ Disponibilidad │
│    :3002     │           │    :3004     │                        │     :3003      │
└──────┬───────┘           └──────┬───────┘                        └────────────────┘
       │ valida anfitrión         │ envía aviso
       ▼                          ▼
┌──────────────┐           ┌────────────────┐
│ Anfitriones  │           │ Notificaciones │
│    :3001     │           │     :3005      │
└──────────────┘           └────────────────┘

Cada servicio tiene su propia base de datos PostgreSQL.
```

### Responsabilidad de cada microservicio

| Microservicio | Puerto | Responsabilidad | Se comunica con |
|---|---|---|---|
| **Anfitriones** | 3001 | Registro de las familias anfitrionas: nombre, comunidad, teléfono e idiomas | — |
| **Experiencias** | 3002 | Catálogo de experiencias: tipo, precio por persona y capacidad diaria | Anfitriones (valida que el anfitrión exista) |
| **Disponibilidad** | 3003 | Cupos por experiencia y fecha; reserva y libera cupos, y evita sobrerreservas | — |
| **Reservas** | 3004 | Crea, confirma y cancela reservas; orquesta a los demás servicios | Experiencias, Disponibilidad y Notificaciones |
| **Notificaciones** | 3005 | Registro y envío simulado de avisos por SMS o WhatsApp al anfitrión | — |

### Endpoints principales

| Microservicio | Endpoints |
|---|---|
| Anfitriones | `GET /health`, `GET /anfitriones`, `GET /anfitriones/:id`, `POST /anfitriones`, `PUT /anfitriones/:id`, `DELETE /anfitriones/:id` |
| Experiencias | `GET /health`, `GET /experiencias`, `GET /experiencias/:id`, `POST /experiencias`, `PUT /experiencias/:id`, `DELETE /experiencias/:id` |
| Disponibilidad | `GET /health`, `GET /disponibilidad?experiencia_id=&fecha=`, `POST /disponibilidad`, `POST /disponibilidad/reservar`, `POST /disponibilidad/liberar` |
| Reservas | `GET /health`, `GET /reservas`, `GET /reservas/:id`, `POST /reservas`, `PATCH /reservas/:id/confirmar`, `PATCH /reservas/:id/cancelar` |
| Notificaciones | `GET /health`, `GET /notificaciones`, `POST /notificaciones` |

### Flujo de una reserva (`POST /reservas`)

1. Reservas consulta a **Experiencias**: si la experiencia no existe, responde `400`.
2. Calcula el total: `precio × personas`.
3. Pide a **Disponibilidad** que descuente cupos: si no hay suficientes, responde `409 Sin cupos`.
4. Guarda la reserva con estado `pendiente`.
5. Pide a **Notificaciones** que avise al anfitrión.
6. Responde `201 Created`.

Al cancelar (`PATCH /reservas/:id/cancelar`), cambia el estado a `cancelada`, pide a **Disponibilidad** que libere los cupos y a **Notificaciones** que avise al anfitrión.

---

## 5. Estructura del repositorio

```
reserva-andina/
├── README.md
├── docker-compose.yml
├── .gitignore
├── .github/workflows/ci.yml
├── anfitriones/
│   ├── src/ (app.js, db.js, server.js)
│   ├── tests/app.test.js
│   ├── init.sql
│   ├── Dockerfile
│   ├── .dockerignore
│   └── package.json
├── experiencias/      (misma estructura)
├── disponibilidad/    (misma estructura)
├── reservas/          (misma estructura)
└── notificaciones/    (misma estructura)
```

---

## 6. Flujo de trabajo del equipo

- La rama `main` está protegida: los cambios solo se incorporan mediante **Pull Request** con al menos **una aprobación**.
- Cada integrante trabaja en su **fork** del repositorio central y propone sus cambios mediante Pull Request. El líder, como dueño del repositorio, trabaja en ramas propias y sus Pull Requests son aprobados por el colaborador.
- El líder revisa, aprueba o solicita cambios antes de incorporar cada contribución.

---

## 7. Ejecución del proyecto

**Requisitos:** Docker y Docker Compose.

```bash
git clone https://github.com/josealexandrol/reserva-andina.git
cd reserva-andina
docker compose up -d --build
docker compose ps
```

Se levantan **10 contenedores**: los 5 microservicios y sus 5 bases de datos PostgreSQL. Cada base de datos se crea y se llena automáticamente con el `init.sql` de su servicio.

| Microservicio | URL | Ejemplo |
|---|---|---|
| Anfitriones | http://localhost:3001 | `GET /anfitriones` |
| Experiencias | http://localhost:3002 | `GET /experiencias` |
| Disponibilidad | http://localhost:3003 | `GET /disponibilidad?experiencia_id=1` |
| Reservas | http://localhost:3004 | `POST /reservas` |
| Notificaciones | http://localhost:3005 | `GET /notificaciones` |

**Ejemplo: crear una reserva**

```bash
curl -X POST http://localhost:3004/reservas \
  -H "Content-Type: application/json" \
  -d '{"experiencia_id":1,"turista_nombre":"Ana Torres","turista_email":"ana@correo.com","fecha":"2026-12-02","personas":2}'
```

Para detener el sistema: `docker compose down` (agregar `-v` para borrar también los datos).

<img width="1917" height="422" alt="image" src="https://github.com/user-attachments/assets/2f38a3f2-e552-4fe1-a384-250701dc78c1" />


---

## 8. Pruebas implementadas

Se usan **Jest** y **Supertest**: **31 pruebas** en total. Las pruebas se ejecutan dentro de cada contenedor, contra su propia base de datos y comunicándose con los demás servicios:

```bash
docker compose exec anfitriones npm test
docker compose exec experiencias npm test
docker compose exec disponibilidad npm test
docker compose exec reservas npm test
docker compose exec notificaciones npm test
```

| Microservicio | Prueba | Funcionalidad que verifica | Comportamiento esperado |
|---|---|---|---|
| Anfitriones | `GET /health` | El servicio está activo | 200 y `servicio: anfitriones` |
| Anfitriones | `GET /anfitriones` | Consulta de todos los anfitriones | 200 y una lista con datos |
| Anfitriones | `GET /anfitriones/1` | Consulta por id | 200 y `id: 1` |
| Anfitriones | `GET /anfitriones/9999` | Anfitrión inexistente | 404 "Anfitrión no encontrado" |
| Anfitriones | `POST /anfitriones sin teléfono` | Validación de campos obligatorios | 400 |
| Anfitriones | `POST /anfitriones válido` | Registro de un anfitrión | 201 y el anfitrión creado |
| Experiencias | `GET /health` | El servicio está activo | 200 y `servicio: experiencias` |
| Experiencias | `GET /experiencias` | Consulta del catálogo | 200 y una lista con datos |
| Experiencias | `GET /experiencias/9999` | Experiencia inexistente | 404 "Experiencia no encontrada" |
| Experiencias | `POST con precio negativo` | Validación del precio | 400 |
| Experiencias | `POST con anfitrión inexistente` | **Comunicación con Anfitriones** | 400 "El anfitrión indicado no existe" |
| Experiencias | `POST válido` | Registro de una experiencia (valida el anfitrión) | 201 y la experiencia creada |
| Disponibilidad | `GET /health` | El servicio está activo | 200 y `servicio: disponibilidad` |
| Disponibilidad | `GET /disponibilidad con filtros` | Consulta por experiencia y fecha | 200 y una sola fila |
| Disponibilidad | `POST /reservar y /liberar` | Descuento y devolución de cupos | Los cupos libres bajan y luego vuelven |
| Disponibilidad | `POST /reservar con 999 personas` | **Prevención de sobrerreservas** | 409 "Sin cupos suficientes" |
| Disponibilidad | `POST /reservar en fecha sin cupos` | Fecha sin disponibilidad registrada | 404 |
| Disponibilidad | `POST /reservar incompleto` | Validación de datos | 400 |
| Reservas | `GET /health` | El servicio está activo | 200 y `servicio: reservas` |
| Reservas | `GET /reservas` | Consulta de todas las reservas | 200 y una lista con datos |
| Reservas | `POST válido y PATCH /cancelar` | **Flujo completo entre servicios**: valida la experiencia, descuenta cupos, avisa al anfitrión y, al cancelar, libera los cupos | 201 y estado `pendiente`; luego `cancelada` y cupos restaurados |
| Reservas | `POST con experiencia inexistente` | **Comunicación con Experiencias** | 400 "La experiencia indicada no existe" |
| Reservas | `POST con 999 personas` | **Comunicación con Disponibilidad** | 409 |
| Reservas | `POST incompleto` | Validación de datos | 400 |
| Reservas | `PATCH /reservas/9999/cancelar` | Reserva inexistente | 404 |
| Notificaciones | `GET /health` | El servicio está activo | 200 y `servicio: notificaciones` |
| Notificaciones | `GET /notificaciones` | Historial de avisos | 200 y una lista con datos |
| Notificaciones | `POST por WhatsApp` | Registro de un aviso | 201 y `canal: whatsapp` |
| Notificaciones | `POST sin canal` | Canal por defecto | 201 y `canal: sms` |
| Notificaciones | `POST sin destinatario` | Validación de campos obligatorios | 400 |
| Notificaciones | `POST con canal "fax"` | Validación del canal | 400 |

Cada integrante documentó la ejecución de las pruebas de su microservicio, con capturas:

| Microservicio | Responsable | Documentación |
|---|---|---|
| Anfitriones | Jose Alexandro Lopez Mamani | [docs/pruebas-anfitriones.md](docs/pruebas-anfitriones.md) |
| Experiencias | Brandon Cunza Robles | [docs/pruebas-experiencias.md](docs/pruebas-experiencias.md) |
| Disponibilidad | Roberto Israel Meza Orore | [docs/pruebas-disponibilidad.md](docs/pruebas-disponibilidad.md) |
| Reservas | Mariano Jhoshua Laura Paz | [docs/pruebas-reservas.md](docs/pruebas-reservas.md) |
| Notificaciones | Oscar Ailthon Espinoza Nieto | [docs/pruebas-notificaciones.md](docs/pruebas-notificaciones.md) |

---

## 9. Integración continua

El flujo [`.github/workflows/ci.yml`](.github/workflows/ci.yml) se ejecuta automáticamente en cada **push** y **Pull Request** hacia `main`:

1. **Descarga el código** del repositorio.
2. **Construye las 5 imágenes Docker** (`docker compose build`), lo que verifica que cada Dockerfile funcione.
3. **Levanta los 10 contenedores** (`docker compose up -d`).
4. **Espera** a que las 5 API respondan en `/health`.
5. **Ejecuta las pruebas** de cada microservicio; si una falla, el flujo se marca en rojo y se muestran los registros.
6. **Detiene y elimina** los contenedores (`docker compose down -v`).

Así, ningún cambio llega a `main` sin que el sistema completo se construya y pase sus pruebas.

<img width="1379" height="868" alt="image" src="https://github.com/user-attachments/assets/607e464f-c8c0-4a09-9708-7b93d46bae81" />
