# Hito 0: Propuesta Inicial del Sistema de Tickets

## 1. Situación y usuario

En entornos operativos o de servicios administrativos donde no se cuenta con una herramienta centralizada de mesa de ayuda, las solicitudes de soporte técnico se reciben a través de múltiples canales informales, como mensajes de texto, correos electrónicos sueltos, llamadas breves o peticiones verbales en los pasillos. 

Esta fragmentación genera varios problemas graves para el flujo de trabajo:
- **Pérdida de solicitudes y falta de visibilidad:** Muchas tareas quedan atrapadas en la bandeja de entrada personal de los técnicos o se olvidan al no quedar registradas en un lugar común.
- **Falta de priorización:** Los incidentes urgentes (como la caída de una red local o un fallo en la facturación) compiten en el mismo nivel con tareas secundarias (como el mantenimiento de un accesorio).
- **Incertidumbre del usuario:** Quien solicita la ayuda no tiene forma de saber en qué estado se encuentra su petición, si fue asignada a alguien o cuándo será resuelta, lo que genera interrupciones constantes preguntando por el avance.

Para resolver esto, el sistema estará dirigido a dos roles principales:
1. **El usuario solicitante:** Quien necesita reportar una falla o requerimiento y requiere saber si su ticket ya fue recibido, quién lo está atendiendo y si hubo algún avance o cambio de estado.
2. **El técnico de soporte:** Quien necesita visualizar una lista ordenada de tareas pendientes, filtrar por urgencia, actualizar el avance del trabajo y cerrar la solicitud una vez solucionada.

---

## 2. Acciones y resultados

El sistema ofrecerá un flujo sencillo guiado por las siguientes acciones:

* **Registrar un ticket:**
  - *Acción:* El usuario llena un formulario básico indicando su nombre, departamento, categoría del problema, prioridad estimada y una descripción detallada de la falla.
  - *Resultado:* El sistema crea la solicitud, le asigna un código identificador único (ejemplo: `TCK-1001`), establece su estado inicial como "Abierto" y muestra un mensaje de confirmación con el resumen de los datos ingresados.

* **Listar y filtrar tickets:**
  - *Acción:* El técnico o usuario accede al panel principal de tickets.
  - *Resultado:* El sistema despliega una tabla que muestra los tickets ordenados por fecha o prioridad. Si se aplican filtros (por estado "Abierto" o prioridad "Alta"), la lista se actualiza instantáneamente mostrando solo los elementos coincidentes.

* **Consultar detalle de un ticket:**
  - *Acción:* La persona hace clic en un ticket específico de la lista.
  - *Resultado:* Se abre una vista detallada con la descripción completa, la fecha exacta de registro, el técnico asignado y el historial de cambios de estado.

* **Cambiar de estado a un ticket:**
  - *Acción:* El técnico selecciona un nuevo estado en una lista desplegable (ejemplo: cambiar de "Abierto" a "En Proceso" o "Resuelto").
  - *Resultado:* El sistema valida el cambio, actualiza la etiqueta visible del estado, registra la fecha de modificación y muestra una notificación en pantalla confirmando la actualización.

---

## 3. Clases iniciales

Para representar este dominio dentro de C#, definiremos la clase principal `Ticket` y una clase colaboradora `Usuario`.

### Clase `Ticket`
Contiene los datos centrales de la solicitud y controla las reglas de transición de estado:
- `Id` (int): Identificador único.
- `Titulo` (string): Breve descripción del problema.
- `Descripcion` (string): Explicación detallada.
- `Estado` (string/enum): Puede ser "Abierto", "En Proceso", "Resuelto" o "Cerrado".
- `Prioridad` (string/enum): "Baja", "Media" o "Alta".
- `FechaCreacion` (DateTime): Fecha y hora en que se registró.
- `Solicitante` (Usuario): Referencia al usuario que creó el ticket.

*Regla de estado dentro de `Ticket`:* Un ticket no puede pasar directamente de "Abierto" a "Cerrado"; debe cambiar primero a "En Proceso" o "Resuelto".

### Clase `Usuario`
Representa a la persona que interactúa con el sistema:
- `Id` (int)
- `NombreCompleto` (string)
- `Correo` (string)
- `Departamento` (string)

### Colaboración entre clases
La clase `Ticket` posee una propiedad de tipo `Usuario` (agregación). Cuando se consulta o crea un ticket, la clase `Ticket` delega en `Usuario` la identificación de quién solicita el soporte y a qué departamento pertenece el incidente.
