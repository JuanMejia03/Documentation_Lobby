## 3. Diagramas de proceso

A continuación se describen los principales procesos del sistema **GERS Lobby**, con su respectiva lógica y puntos de integración con la base de datos y el middleware.

---

### 3.1 Proceso de inicio de sesión

**Descripción:**  
El usuario accede al sistema proporcionando sus credenciales. El `EndpointAuth.php` valida la autenticidad, gestiona la sesión y asigna el rol correspondiente (`Administrador`, `Usuario`).

**Flujo:**

1. Usuario ingresa con los datos corporativos( correo, contraseña) en el  login.
    
2. Frontend envía los datos a `EndpointAuth.php` vía **fetch POST**.
    
3. Valida:
    
    - Existencia del usuario en la API de node.
    
4. Si es válido → se crea sesión en `Session.php`.
    
5. Se asigna rol con `UserRoles::from()`.
    
6. Redirección al panel correspondiente (admin o usuario).
    

**Diagrama:**  

![[Captura de pantalla 2025-09-01 101544 1.png]]

---

### 3.2 Proceso de reserva

**Descripción:**  
Los usuarios (administradores o registrados) pueden realizar reservas de salas o auditorios. El sistema verifica la disponibilidad, el estado del recurso y registra la reserva en la base de datos.

**Flujo:**

1. Usuario selecciona fecha, hora, sala/auditorio desde el **calendario FullCalendar**.
    
2. El sistema abre el **modal de reserva** personalizado.
    
3. Al confirmar, se envía la solicitud vía `fetch POST` → `ReserverController.php`.
    
4. Verifica:
    
    - Que el recurso no esté en estado **Mantenimiento** (tabla `venue_tb`).
        
    - Que no exista otra reserva en el mismo horario (`reservation_tb`).
        
5. Si está disponible → inserta en `reservation_tb`.
    
6. Se actualiza en tiempo real el calendario con la nueva reserva.
    

**Diagrama:**  

![[Captura de pantalla 2025-09-01 104339.png]]

---

### 3.3 Proceso administrativo

**Descripción:**  
Los administradores acceden al panel de control para gestionar recursos, usuarios y estadísticas. El sistema consulta en la base de datos y presenta información mediante gráficas interactivas.

**Flujo:**

1. Administrador accede a pestaña **“Estadísticas”** en el panel.
    
2. Se dispara un **fetch GET** hacia `StatisticsController.php`.
    
3. El backend consulta en BD (`reservation_tb`, `venue_tb`, `users_tb`).
    
4. Los datos son procesados y devueltos en formato **JSON**.
    
5. Frontend renderiza la información en gráficas (ej. **Chart.js**) y tablas dinámicas.
    
6. Si se crea o elimina una reserva → la gráfica se actualiza automáticamente.
    

**Diagrama:**  

![[Captura de pantalla 2025-09-01 112506.png]]