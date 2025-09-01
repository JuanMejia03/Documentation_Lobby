el sistema de Gers lobby esta desarrollado base a la arquitectura MVC (Modelo Vista Controlador), separando la lógica, la gestión de datos y la interfaz de usuario. 


 • Frontend: HTML5, JavaScript, TailwindCSS, Alpine.js y FullCalendar (estos tres últimos profundizaremos mas adelante, en el apartado de las librerías).
 • Backend: PHP con Programación Orientada a Objetos (POO).
 • Base de datos: MySQL.

 El flujo del sistema inicia con las solicitudes desde el cliente (navegador), que se envían a través de fetch hacia el Middleware (empírico de la plantilla usada, si requieres tener mas información para el uso de la plantilla consulta la documentación), el cual autentica y direcciona las peticiones al controlador correspondiente.