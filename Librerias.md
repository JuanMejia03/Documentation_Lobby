
## 2. Librerías

El sistema **GERS Lobby** utiliza diversas librerías y frameworks que optimizan la experiencia de usuario y simplifican el proceso de desarrollo. A continuación, se describen:

- **TailwindCSS**
    
    - Se utiliza como framework de estilos CSS para garantizar un diseño moderno, limpio y responsive en todos los dispositivos (móvil, tablet, PC, TV).
        
    - Facilita la creación de componentes reutilizables sin necesidad de escribir CSS desde cero.
        
- **Alpine.js**
    
    - Implementado para dotar de interactividad ligera a la interfaz de usuario.
        
    - Permite controlar pestañas dinámicas (ej. secciones de reservas, estadísticas y gráficas) sin recargar la página.
        
    - Ideal para lógica simple en el frontend, evitando el uso de librerías más pesadas.
        
- **FullCalendar**
    
    - Integrado en el módulo de administración y en la vista de reservas.
        
    - Permite visualizar, crear y gestionar reservas en formato calendario.
        
    - Brinda soporte para eventos dinámicos, actualizaciones en tiempo real y compatibilidad con fetch.
        
- **Chart.js**
    
    - Usado en el panel administrativo para generar gráficas dinámicas y estadísticas de uso.
        
    - Facilita la representación de datos como reservas por sala, por usuario o por periodo de tiempo.
        
    - Se actualiza automáticamente cuando se crean o eliminan reservas.
        
- **Notiflix / SweetAlert**
    
    - Se emplean para notificaciones y alertas visuales dentro de la aplicación.
        
    - Mejoran la experiencia del usuario al mostrar mensajes de confirmación, éxito o error de forma intuitiva.
        
- **Composer (autoload)**
    
    - Utilizado en el backend para la gestión de dependencias en PHP.
        
    - Permite cargar automáticamente controladores, modelos y librerías de terceros.
        
    - Facilita la integración de herramientas como el ORM personalizado y otros paquetes de soporte.