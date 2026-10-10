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

+----------------------------------+          +----------------------------------+
|              Ticket              |          |             Usuario              |
+----------------------------------+          +----------------------------------+
| - Id: int                        |          | - Id: int                        |
| - Titulo: string                 |          | - NombreCompleto: string         |
| - Descripcion: string            | 1      1 | - Correo: string                 |
| - Estado: string                 |--------->| - Departamento: string           |
| - Prioridad: string              |          +----------------------------------+
| - FechaCreacion: DateTime        |
| - Solicitante: Usuario           |
+----------------------------------+
| + CambiarEstado(nuevoEstado)     |
+----------------------------------+


---

## 4. Reglas del negocio

### Regla 1: Obligatoriedad de descripción mínima
El campo descripción debe contener información suficiente para que el técnico entienda el problema antes de atenderlo.
- **Entrada válida:** `"La impresora HP del área de contabilidad no enciende al presionar el botón de encendido."`
- **Entrada inválida:** `"No sirve."` (menos de 10 caracteres).
- **Respuesta del sistema:** El sistema bloquea el envío del formulario y muestra el mensaje de error: *"La descripción debe tener al menos 15 caracteres para registrar la solicitud."*

### Regla 2: Formato de correo electrónico institucional
El correo ingresado al registrar una solicitud debe pertenecer a la organización.
- **Entrada válida:** `bryan.sanchez@empresa.com`
- **Entrada inválida:** `bryan123@gmail.com`
- **Respuesta del sistema:** El sistema resalta el campo de correo en rojo y muestra la alerta: *"Debe ingresar un correo corporativo válido con la extensión @empresa.com."*

### Regla 3: Cierre de ticket requiere resolución
Un ticket solo puede pasar al estado "Cerrado" si previamente ha sido marcado como "Resuelto" por el técnico.
- **Entrada válida:** Transición de estado `"Resuelto"` a `"Cerrado"`.
- **Entrada inválida:** Transición de estado `"En Proceso"` a `"Cerrado"` directamente.
- **Respuesta del sistema:** El sistema deshabilita la opción "Cerrado" en la interfaz gráfica y, si se intenta forzar por lógica, arroja una excepción: *"No se puede cerrar un ticket que no ha sido marcado previamente como resuelto."*

---

## 5. Captura o boceto de interfaz

+-----------------------------------------------------------------------------------+
|  MESA DE AYUDA - SISTEMA DE TICKETS                                               |
+-----------------------------------------------------------------------------------+
|  [ + Nuevo Ticket ]     Filtrar por Estado: [ Todos    v ]  Buscar: [________]    |
+-----------------------------------------------------------------------------------+
| ID       | Título                   | Solicitante      | Prioridad | Estado      |
+----------+--------------------------+------------------+-----------+-------------+
| TCK-1001 | Falla en impresora HP    | Ana Montenegro   | Alta      | Abierto     |
| TCK-1002 | Sin acceso a red local   | Carlos Ríos      | Media     | En Proceso  |
| TCK-1003 | Error al abrir software  | Elena Gómez      | Baja      | Resuelto    |
+-----------------------------------------------------------------------------------+


---

## 6. Pregunta y próxima acción

* **Duda para discutir con el docente:** 
  ¿Es recomendable desde la primera versión en C# modelar los estados de los tickets utilizando un tipo enumerado (`enum`) o es preferible manejarlo como cadenas de texto (`string`) para facilitar la vinculación inicial con los componentes del formulario en Blazor?

* **Paso concreto para comenzar la primera versión:**
  Crear la solución en C# mediante la CLI de .NET (`dotnet new blazorserver`) y definir la clase del modelo `Ticket.cs` con sus propiedades básicas y la validación de la descripción mínima.

---

## 7. Organización del trabajo

Al trabajar de manera individual, la distribución del tiempo para el desarrollo de la primera versión funcional se organizará en tres fases equilibradas:

1. **Diseño del Modelo de Datos (35% del tiempo):** Construcción de las clases en C# (`Ticket`, `Usuario`), enums y la lógica de validación de las reglas de negocio.
2. **Desarrollo de la Interfaz con Blazor (40% del tiempo):** Creación de los componentes de la vista en Blazor (`TicketList.razor`, `TicketForm.razor`) y enlazado de datos (*data binding*).
3. **Pruebas y Refactorización (25% del tiempo):** Verificación manual de ingresos de datos válidos/inválidos, flujo de actualización de estados y revisión general del código.
