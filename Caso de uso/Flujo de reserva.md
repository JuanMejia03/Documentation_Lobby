**flujo de reserva - Paso 1 : cliente/Vista**
**Objetivo:** Documentar cómo, desde la vista, se prepara y captura la información para crear una reserva

1) Estructura UI relevante

	**Tabs:** Salas / Auditorios con 
	```html 
	x-data="{ tab: 'salas' }"
	```
	**Contenedores**: #hallContainer y #audienceContainer (renderizados por venue/main.js).
	**Modal calendario:** #renderCalendar con
	```html
	<div id="fullcalendar"></div>
	```
	**Modal formulario**: #reservaModal con 
	```html
	<form id="reservaForm">
	```
	
	### Formulario `#reservaForm`

	**Campos ocultos** (se completan dinámicamente):
	- `sala_id`, `auditorio_id` (uno de los dos)    
	- `userId` (inyectado desde sesión)    
	
	**Campos visibles**:
	- `nombreSolicitante` y `email` (readonly desde sesión)    
	- `fechaReserva` (tipo `date`)    
	- `hora_inicio` y `hora_fin` (texto con Flatpickr, 24h, intervalos configurados)    
	
	**Acciones**:
	- **Confirmar** → `submit` del formulario (interceptado por JS para enviar por `fetch`).    
	- **Cancelar** → cierra el modal y/o limpia estado.


**Paso 2 :** Envío por `fetch/post()`
Desde el **frontend** se construye un objeto, capturando los datos por medio de JavaScript, con la información de la reserva, y se envía mediante la función `post()` hacia el endpoint centralizado (`EndpointAuth.php`).
```javascript
    const body = {
      route: "reservation",
      action: "post",
      data: {
        sala_id: document.getElementById("salaId")?.value || null,
        auditorio_id: document.getElementById("auditorioId")?.value || null,
        usuario_id: userId,
        fechaReserva,
        horaInicio,
        horaFin,
      },
    };
```

**Request** (payload esperado)
```Json
{
  "route": "reservation",'
  "action": "post",
  "data": {
    "sala_id": "<uuid de la sala> | null",
    "auditorio_id": "<uuid del auditorio> | null",
    "usuario_id": "<uuid del usuario desde sesión>",
    "fechaReserva": "YYYY-MM-DD",
    "horaInicio": "HH:mm:ss",
    "horaFin": "HH:mm:ss"
  }
}
```

ejemplo de respuesta (200 OK)
```json
{
  "success": true,
  "message": "Reserva creada correctamente",
  "data": {
    "inserted_id": true
  }
}
```

Un pequeño paréntesis antes de continuar;
El proyecto parte de una plantilla que ya incluye un mecanismo de comunicación entre frontend y Backend.  
Este mecanismo se compone principalmente de dos piezas:

- **`requests.js`**: módulo de utilidades en JavaScript que implementa las funciones `get`, `post`, `put` y `destroy`. Estas funciones encapsulan las peticiones `fetch` hacia un único punto de entrada (`endpoint.php`), gestionan cabeceras comunes (como el token CSRF y `XMLHttpRequest`) y controlan la retroalimentación al usuario mediante el componente `modals`.

- **`endpoint.php`**: archivo del lado del servidor que centraliza las solicitudes entrantes. Se encarga de recibir el método HTTP (`GET`, `POST`, `PUT`, `DELETE`), decodificar los datos enviados, validar el encabezado de seguridad (`X-Requested-With`), y posteriormente delegar la petición a `EndpointAuth` para su validación y enrutamiento hacia el controlador correspondiente.
    

La clase **`EndpointAuth`** añade capas adicionales de seguridad y control:

- Limitación de peticiones por IP (rate limit).
- Bloqueo de acceso directo sin cabecera `XMLHttpRequest`.
- Validación de token CSRF.
- Enrutamiento a los distintos _routers_ (`VenueRouter`, `ReservationRouter`, `StatisticsRouter`) en función del valor `route` recibido.

 Es importante aclarar que este flujo no fue desarrollado como parte de las tareas de este proyecto, sino que proviene de la **plantilla base** con la que se inició el trabajo. Aquí únicamente se describe su funcionamiento para documentar cómo interactúan nuestras implementaciones con dicho mecanismo.

El archivo `ReservationRouter.php` implementa la lógica de enrutamiento específica para el módulo de reservas.
###### Estructura de despacho
El método `getMethod()` mapea cada acción con la función correspondiente del `ReserverController`:
```php
[
  'get'      => [ReserverController, 'getReserver'],
  'delete'   => [ReserverController, 'deleteReserver'],
  'post'     => [ReserverController, 'createReserver'],
  'calendar' => [ReserverController, 'getCalendarEvents'],
]
```

**Paso 3:** Lógica en el Controlador (`ReserverController`)
Cuando el Middleware recibe el Request con `route: "reservation"` y `action: "post"`, delega la operación al método `createReserver()` del **`ReserverController`**.

Este método se encarga de:

1. Validar los datos recibidos.

```php
try {
    $payload = $data->data ?? null;
    if (!$payload) {
        return Responses::error('No se recibió el cuerpo de datos', null, 400);
    }

    $usuarioId   = $payload->usuario_id ?? null;
    $salaId      = $payload->sala_id ?? null;
    $auditorioId = $payload->auditorio_id ?? null;
    $fecha       = $payload->fechaReserva ?? null;
    $horaInicio  = $payload->horaInicio ?? null;
    $horaFin     = $payload->horaFin ?? null;

    if (!$usuarioId || !$fecha || !$horaInicio || !$horaFin) {
        return Responses::error('Faltan datos requeridos', null, 400);
    }

```
2. Determinar el espacio solicitado (sala o auditorio).
3. Verificar que el espacio esté disponible y no en **mantenimiento**.
4. Confirmar que no haya conflictos de horarios.
5. Insertar la nueva reserva en `reservations_tb`.
```php
$inserted = $this->db->table('reservations_tb')->insert([
    'id_user'   => $usuarioId,
    'id_venue'  => $venueId,
    'id_state'  => $estadoId,
    'date'      => $fecha,
    'init_hour' => $horaInicio,
    'final_hour'=> $horaFin
]);
```
6. Responder al frontend con el resultado.

**Paso 4:** Respuesta al Cliente (Frontend)
Cuando el `ReserverController` responde (éxito o error), el **frontend** interpreta el JSON mediante la función `post()` y muestra una notificación al usuario usando **Notiflix**.