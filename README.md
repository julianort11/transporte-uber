# Quedados

- Jose julian ortega navarro

## Prgeuntas Guia
1. ¿Qué entiendes por “endpoint” en el contexto de una API?
- Es una ruta expuesta por el servidor que realiza una accion o devuelve algun recurso

2. ¿Cuál es la diferencia entre un endpoint público y uno privado?
- En un endpoint publico tienes acceso sin necesidad de  tener autenticacion lo que lo vuelve mas vulnerables y no apto para manejar datos sensibles, para usar un endpoint privado se requiere autenticacion o autorizacion, lo que lo vuelve una forma segura de manejar datos delicados

3. ¿Qué información de un usuario consideras confidencial y no debería exponerse?
-  datos sensibles del cliente como contraseñas, ubicaciones en tiempo real, datos bancarios, tokens de autenticacion

4. ¿Por qué es importante definir bien los métodos HTTP (GET, POST, PUT/PATCH, DELETE) en cada endpoint?
- Por que expresan la intencion de la operacion lo que hace que la API sea mas segura, clara y coherente

5. ¿Qué tipo de información requiere autenticación en este sistema?
- Creacion de viajes, inforacion de pagos, ubicacion en tiempo real, acciones del conductor

6. ¿Cómo manejarías la seguridad de la ubicación de conductores y pasajeros?
- Enviar ubicaciones sobre HTTPS; almacenar sólo con consentimiento; reducir precisión para UI pública; tokens efímeros para compartir ubicación; retención limitada; acceso solo por actores autorizados del viaje

7. ¿Qué pasaría si un viaje es solicitado y no hay conductores disponibles? ¿Cómo debería responder la API?
- sugerir alternativas (reintento automático, ampliar zona, estimado de espera)

8. ¿Cómo identificarías los recursos principales de esta aplicación?
- users, drivers (subtipo de user), rides, payments, ratings, vehicles, locations, sessions, receipts, reports

9. ¿Qué ventajas tendría versionar la API (por ejemplo, `/v1/...`) desde el inicio?
- Permite cambios incompatibles futuros sin romper clientes existentes; facilita migraciones, pruebas A/B, y soporte a varias versiones simultáneas
    
10. ¿Por qué es importante documentar las respuestas de error y no solo las exitosas?
- Para que clientes manejen fallos de forma predecible, permitan UX apropiada y faciliten debugging e integración (ej. códigos, campos faltantes, límites)

## Roles permitidos

**Pasajero (user:passenger)**

- Registrar cuenta
- Solicitar viajes
- Cancelar (con reglas)
- Ver historial
- Pagar
- Calificar conductor
- Ver recibos
- Recibir notificaciones

**Conductor (user:driver)**

- Registrar (doc + vehículo)
- Indicar disponibilidad/ubicación
- Aceptar/rechazar solicitudes
- Iniciar/finalizar viaje
- Generar recibo
- Ver calificaciones
- Retirar ganancias

**Administrador (admin)**

- Gestionar usuarios/vehículos
- Ver reportes, forzar cancelaciones
- Emitir reembolsos
- Moderar calificaciones
- Gestionar tarifas/zonas
- Ver logs.

Cada acción está protegida por autorización. Muchas rutas requieren role: driver o role: admin según conveniencia.


## Recursos principales
- users (pasajeros y conductores como sub-roles)
- sessions / auth
- drivers (perfil conductor + vehículo)
- vehicles
- rides (viajes/solicitudes)
- locations (tracking / last known)
- payments
- receipts
- ratings
- reports / admin

  ## Tabla endpoints
| Método | Ruta | Descripción | Parámetros (path / query / body) | Autenticación |
|---|---|---|---|---|
| POST | `/api/v1/auth/register` | Registrar usuario (pasajero o conductor basic) | Body: `{ name, email, password, role: "passenger"|"driver" }` | Público |
| POST | `/api/v1/auth/login` | Login -> devuelve JWT | Body: `{ email, password }` | Público |
| POST | `/api/v1/auth/refresh` | Refresh token (opcional) | Body: `{ refreshToken }` | Público (con refresh token) |
| GET | `/api/v1/users/me` | Obtener perfil del usuario actual | — | Requiere token |
| PATCH | `/api/v1/users/me` | Actualizar perfil (foto, teléfono) | Body opcional campos de perfil | Requiere token |
| POST | `/api/v1/drivers/apply` | Completar registro conductor (docs, vehículo) | Body: `{ licenseNumber, vehicleId, documents: [...] }` y archivos | Requiere token (role: driver) |
| GET | `/api/v1/drivers/:driverId` | Ver perfil público de conductor (calificaciones, vehículo) | path param `driverId` | Público |
| PATCH | `/api/v1/drivers/:driverId/availability` | Cambiar estado disponible/no disponible | Body: `{ available: true/false }` | Requiere token (role: driver) |
| POST | `/api/v1/locations/update` | Enviar ubicación en tiempo real | Body: `{ lat, lng, heading, speed, timestamp }` | Requiere token (driver o passenger durante viaje) |
| GET | `/api/v1/rides/estimate` | Estimar tarifa/tiempo antes de crear viaje | Query: `fromLat, fromLng, toLat, toLng, vehicleType` | Requiere token |
| POST | `/api/v1/rides` | Crear solicitud de viaje (pasajero) | Body: `{ from:{lat,lng,address}, to:{lat,lng,address}, paymentMethodId, vehicleType, notes }` | Requiere token (passenger) |
| GET | `/api/v1/rides/:rideId` | Detalles de un ride | path param `rideId` | Requiere token (involucrado o admin) |
| PATCH | `/api/v1/rides/:rideId/cancel` | Cancelar viaje (pasajero o driver según reglas) | Body: `{ reason, who }` | Requiere token (involucrado) |
| POST | `/api/v1/rides/:rideId/accept` | Conductor acepta solicitud | path `rideId`, Body opcional `{ eta }` | Requiere token (role: driver) |
| POST | `/api/v1/rides/:rideId/start` | Iniciar viaje (driver) | path `rideId`, Body `{ startTime, startLocation }` | Requiere token (driver) |
| POST | `/api/v1/rides/:rideId/complete` | Finalizar viaje y calcular tarifa | Body: `{ endTime, endLocation, totalDistance }` | Requiere token (driver) |
| GET | `/api/v1/payments/methods` | Listar métodos de pago soportados | — | Requiere token |
| POST | `/api/v1/payments/charge` | Procesar pago de un ride (o almacenar intención) | Body: `{ rideId, paymentMethodId, amount, currency }` | Requiere token |
| GET | `/api/v1/payments/history` | Historial de pagos del usuario | Query: `page, perPage, from, to` | Requiere token |
| GET | `/api/v1/receipts/:rideId` | Obtener recibo PDF/JSON de ride | path `rideId` | Requiere token (involucrado) |
| POST | `/api/v1/ratings` | Crear calificación (pasajero->conductor o viceversa) | Body: `{ rideId, fromUserId, toUserId, rating:1-5, comment }` | Requiere token |
| GET | `/api/v1/users/:userId/ratings` | Obtener calificaciones de un usuario | path `userId`, query `page` | Público (limitado), o requiere token para detalle |
| GET | `/api/v1/admin/users` | Admin: listar usuarios (filtros) | Query: `role, status, page` | Requiere token (role: admin) |
| PATCH | `/api/v1/admin/users/:userId/ban` | Admin: banear/desbanear usuario | Body: `{ banned: true/false, reason }` | Requiere token (role: admin) |
| GET | `/api/v1/admin/reports/usage` | Admin: reportes (viajes, ingresos) | Query: `from,to,groupBy` | Requiere token (role: admin) |

## Flujos de uso

1. Cliente (pasajero) solicita estimación: GET /api/v1/rides/estimate?fromLat=...&fromLng=...&toLat=... → recibe {estimatedFare, estimatedTime}.
2. Crear solicitud: POST /api/v1/rides con body { from, to, paymentMethodId }. Respuesta: { rideId, status: "searching", createdAt }.
3. Sistema busca conductores (matching por cercanía y tipo). Si conductor disponible, notifica a X conductores.
4. Conductor acepta: POST /api/v1/rides/:rideId/accept → ride status cambia a accepted, se asocia driverId. Notifica pasajero con ETA.
5. Conductor inicia viaje en pickup: POST /api/v1/rides/:rideId/start → status in_progress.

## Decisiones de diseño y justificación

**Estructura RESTful y versionado**
- Uso /api/v1/... para poder evolucionar sin romper clientes. Endpoints nombrados por recursos (/rides, /users) para claridad

**Autenticación y autorización**
- JWT con expiración corta + refresh tokens para sesiones persistentes. Roles embebidos en token (role: passenger|driver|admin). Todas las rutas sensibles requieren HTTPS y verificación del token. Para operaciones críticas (pagos, reembolsos) posible 2FA opcional.

**Seguridad de la ubicación**
- Ubicación compartida solo cuando ride.status en searching/in_progress y con consentimiento. Se guarda solo lastKnown y trazas del viaje (anónimas y cifradas). En APIs públicas se retorna ubicación con baja precisión (por ejemplo, truncada a 3 decimales) y nunca datos personales.

**Privacidad de datos**
- No exponer campos sensibles (e.g., email solo al propio usuario y admin, tarjetas mostradas enmascaradas). Logs y backups cifrados. Políticas de retención (ej. ubicaciones en tiempo real se borran después de 30 días).

**Pagos**
- No almacenar CVV; usar tokenización de proveedores (Stripe/Adyen). payments/charge registra el intento y el status (pending, succeeded, failed).

**Manejo de falta de conductores**
- Si no hay conductores, se responde con 200 y {available:false, estimatedWait: null} o con 503 si hay una degradación del servicio. También colas y retry exponibles al cliente.

**Escalabilidad**
- Webhooks/event-driven para notificaciones (push), colas para matching y procesamiento de pagos, separación de servicios (auth, rides, payments).

**Errores y consistencia**
- Un esquema de errores uniforme para facilitar manejo en cliente (ver sección siguiente).


## Manejo de errores
**Formato general (JSON):**
```json
{
  "error": {
    "code": "string",        // código interno legible, e.g., "AUTH_INVALID_TOKEN"
    "message": "string",     // texto legible para mostrar
    "status": 401,           // HTTP status
    "details": { ... }       // opcional, campo por campo
  }
}
```

**Esquema de códigos HTTP y uso general**

- 200 OK — éxito general.
- 201 Created — recurso creado.
- 400 Bad Request — payload inválido.
- 401 Unauthorized — token inválido/ausente.
- 403 Forbidden — token válido pero sin permisos.
- 404 Not Found — recurso no existe.
- 409 Conflict — conflicto (p.ej. ride ya aceptado).
- 422 Unprocessable Entity — validación semántica (p.ej. coordenadas fuera de rango).
- 429 Too Many Requests — rate limiting.
- 503 Service Unavailable — degradación (p.ej. no drivers disponibles o gateway caído).

5 errores ejemplo

- 401 — { code: "AUTH_EXPIRED", message: "Token expirado" } — cuando el JWT expiró.
- 403 — { code: "DRIVER_ONLY", message: "Solo conductores pueden aceptar viajes" } — conductor-only action.
- 404 — { code: "RIDE_NOT_FOUND", message: "RideId no encontrado" } — ride inexistente.
- 409 — { code: "RIDE_ALREADY_ACCEPTED", message: "Otro conductor ya aceptó este viaje" } — conflicto en aceptación.
- 503 — { code: "NO_DRIVERS_AVAILABLE", message: "No hay conductores disponibles en su zona" } — al crear ride y no encontrar matches.


## Propuestas de mejora / funcionalidades futuras
1. **Scheduling & pre-booking** — permitir reservar viajes con antelación y gestionar bloques de tiempo
2. **Promociones y wallets** — sistema de cupones, wallet interno y split payments
3. **Rides compartidos / pooling** — soportar múltiples pickups/dropoffs y optimización de rutas
4. **Notificaciones y chat en tiempo real** — canal seguro pasajero-conductor (mensajería temporal)
5. **Analítica avanzada y ML** — predicción de demanda para surge pricing y mejor matching

   
