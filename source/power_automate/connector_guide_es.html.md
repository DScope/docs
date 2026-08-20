---
title: Conectar DataScope con Power Automate
---

# Conectar DataScope con Power Automate

> Reading this in English or Portuguese? [Switch to the English guide](connector_guide_en.html) | [Versão em português](connector_guide_pt.html).

Esta guía explica cómo habilitar la integración de DataScope en Power Automate
para automatizar tu operación en terreno: disparar flujos cuando se completa un
formulario, cuando se genera un documento, cuando se asigna una tarea o cuando
se registra un ticket.

## Antes de empezar

Necesitas:

- Una licencia de Power Automate que habilite conectores personalizados. Si no
  estás seguro, consúltalo con el administrador de tu tenant de Microsoft.
- Permisos para crear conectores en el entorno de Power Automate donde vas a
  trabajar.
- Tu API Key de DataScope. La obtienes desde **app.mydatascope.com/integrations**,
  en la sección Integraciones de tu cuenta.
- El archivo `apiDefinition.swagger.json` que te entregamos junto con esta guía.

## Paso 1: crear el conector

1. Entra a [make.powerautomate.com](https://make.powerautomate.com) e inicia
   sesión.
2. Selecciona el entorno donde quieres crear el conector, arriba a la derecha.
3. En el menú lateral, abre **Más** y luego **Descubrir todo**.
4. Selecciona **Conectores personalizados**.
5. Haz clic en **Nuevo conector** y elige **Importar un archivo de OpenAPI**.
6. Escribe un nombre para el conector, por ejemplo DataScope, selecciona el
   archivo <a href="https://raw.githubusercontent.com/DScope/docs/main/source/power_automate/apiDefinition.swagger.json" target="_blank" rel="noopener noreferrer"><code>apiDefinition.swagger.json</code></a> y continúa.

## Paso 2: revisar la configuración general

El archivo ya trae configurado el host, la ruta base y la descripción, así que
no necesitas modificar nada en esta pantalla.

Si quieres que el conector muestre el logo de DataScope, descarga el
<a href="https://raw.githubusercontent.com/DScope/docs/main/source/power_automate/datascope_icon.png" target="_blank" rel="noopener noreferrer">ícono de DataScope (PNG)</a>
y súbelo en el campo de ícono, luego define el color de fondo. Es opcional y
solo afecta la apariencia.

## Paso 3: configurar la autenticación

El archivo ya declara el tipo de autenticación. Verifica que la pantalla de
seguridad muestre:

- Tipo de autenticación: **Clave de API**
- Nombre del parámetro: **Authorization**
- Ubicación: **Encabezado**

Continúa hasta el final del asistente y selecciona **Crear conector**.

## Paso 4: crear la conexión

Al usar el conector por primera vez en un flujo, Power Automate te pedirá una
conexión. Pega ahí tu API Key de DataScope.

Cada usuario que arme flujos necesita crear su propia conexión con la API Key
de su cuenta.

## Actualizar a una versión más nueva del conector

Cuando publicamos una versión nueva de `apiDefinition.swagger.json` (por ejemplo, para agregar un disparador o corregir un campo), no necesitas crear un conector nuevo — actualizas el que ya existe:

1. En [make.powerautomate.com](https://make.powerautomate.com), ve a **Datos > Conectores personalizados**.
2. Abre el conector de DataScope, luego abre su **editor de Swagger** (o vuelve a importar el archivo desde la pestaña **General**, según la versión del portal en la que estés).
3. Reemplaza la definición con el archivo nuevo y selecciona **Actualizar conector**.

Algunas cosas que vale la pena saber antes de hacer esto:

- **Esto actualiza el conector para todos en el entorno**, no solo para ti — no es algo acotado a un solo flujo.
- **Los disparadores ya configurados siguen funcionando mientras su operación subyacente no haya cambiado.** Los disparadores nuevos quedan disponibles para agregar a flujos; los que ya existen no se agregan ni se quitan automáticamente de los flujos que ya los usan.
- **Revisa el ícono, el color de fondo y la configuración de autenticación después de actualizar.** La documentación de Microsoft no confirma explícitamente si volver a importar un archivo siempre preserva eso, así que trátalo como "verifica, no asumas" en vez de algo garantizado.
- **Si el disparador de un flujo deja de comportarse como esperas después de una actualización, elimina y vuelve a crear la conexión de ese disparador** en el flujo afectado. La propia guía de Microsoft para actualizar la definición de un conector personalizado recomienda esto como la forma de asegurar que un flujo tome el cambio correctamente.

## Qué puedes automatizar

El conector entrega los siguientes disparadores:

| Disparador | Se activa cuando |
|---|---|
| Nueva respuesta | Se envía una respuesta de formulario |
| Nuevo PDF | Se genera un documento PDF |
| Cambio de estado | Una respuesta de formulario cambia de estado |
| Nueva tarea asignada | Se asigna una tarea |
| Nuevo ticket | Se registra un ticket |
| Cambio de estado de ticket | Un ticket cambia de estado |
| Firma completada | Se completa la firma de un documento |
| Firma rechazada | Se rechaza una solicitud de firma |
| Firma actualizada | Se actualizan las firmas de un documento |

Cada disparador entrega los datos del evento como contenido dinámico, listos
para usar en los pasos siguientes del flujo sin necesidad de procesar el JSON
manualmente.

## Consideraciones importantes

- **Una conexión activa por formulario.** Los disparadores asociados a un
  formulario admiten una sola conexión activa a la vez. Si necesitas varios
  flujos sobre el mismo formulario, encadénalos desde un único disparador.
- **Los disparadores de tickets operan a nivel de cuenta.** Admiten una sola
  conexión activa por cuenta.
- **Los campos disponibles varían según tu configuración.** Algunos datos, como
  los de planificación o los campos extendidos de tareas, aparecen solo si esas
  funcionalidades están habilitadas en tu cuenta.
- **El conector funciona en el entorno donde lo creaste.** Si trabajas con
  varios entornos, repite la importación en cada uno.
- **El disparador "Nueva respuesta" no se limpia solo al apagarlo.** Si detienes
  o eliminas un flow que usa el disparador "Nueva respuesta" (`hooks_flow`), la
  suscripción no se elimina automáticamente del lado de DataScope. Para
  detenerlo por completo, también debes ir a
  [app.mydatascope.com/integrations](https://app.mydatascope.com/integrations)
  y eliminar ahí la conexión correspondiente. El resto de los disparadores
  (formularios, PDFs, tareas, tickets, firmas) sí se limpian automáticamente
  al apagar el flow en Power Automate.

## Si algo no funciona

- **La conexión falla al crearse:** verifica que la API Key esté completa y sin
  espacios al inicio o al final.
- **El flujo no se dispara:** confirma que no exista otra conexión activa sobre
  el mismo formulario, y revisa el historial del flujo en Power Automate.
- **No aparecen los campos esperados:** algunos campos dependen de la
  configuración de tu cuenta. Escríbenos y lo revisamos contigo.

Para cualquier duda, escribe a soporte y te acompañamos en la configuración.
