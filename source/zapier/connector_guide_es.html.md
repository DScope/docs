---
title: Conectar DataScope con Zapier
---

# Conectar DataScope con Zapier

> Reading this in English or Portuguese? [Switch to the English guide](connector_guide_en.html) | [Versão em português](connector_guide_pt.html).

Esta guía explica cómo conectar DataScope con Zapier para automatizar tu
operación en terreno: disparar Zaps cuando se completa un formulario, cuando
se genera un documento, cuando se asigna una tarea o cuando se registra un
ticket, y actuar sobre DataScope desde las otras aplicaciones que ya usas.

<aside class="notice">
A diferencia del conector de Power Automate, acá no hay nada que importar. La app de DataScope está publicada en el directorio de aplicaciones de Zapier, así que lo único que necesitas para conectarla es tu API Key.
</aside>

## Antes de empezar

Necesitas:

- Una cuenta de Zapier con permisos para crear Zaps en el espacio de trabajo
  donde vas a trabajar.
- Tu API Key de DataScope. La obtienes desde <a href="https://app.mydatascope.com/integrations" target="_blank" rel="noopener noreferrer">app.mydatascope.com/integrations</a>,
  en la sección Integraciones de tu cuenta, en la pestaña API Key.
- Al menos un formulario publicado en DataScope. La mayoría de los disparadores
  y de las actions te piden elegir un formulario, y la lista se llena con los
  formularios de tu cuenta.

## Paso 1: crear el Zap

1. Entra a <a href="https://zapier.com/apps/datascope-forms/integrations" target="_blank" rel="noopener noreferrer">la app de DataScope en Zapier</a> e inicia sesión.
2. Selecciona **Create Zap**, o parte de una de las plantillas listadas en esa
   página. La sección Integraciones de DataScope lista las mismas plantillas,
   agrupadas por caso de uso.
3. En el paso del disparador, busca **DataScope Forms** y selecciónalo.
4. Elige el evento al que quieres reaccionar, de la lista en
   [Qué puedes automatizar](#que-puedes-automatizar).

## Paso 2: crear la conexión

La primera vez que uses DataScope en un Zap, Zapier te pedirá una conexión y
te mostrará un solo campo, **Token Key**. Pega ahí tu API Key de DataScope.

Zapier valida la clave de inmediato, así que una API Key incompleta o
incorrecta falla en este paso y no en silencio más adelante.

La conexión pertenece a tu cuenta de Zapier y puedes reutilizarla en todos los
Zaps que armes. Cada persona que arme Zaps necesita su propia conexión, con la
API Key de su cuenta.

## Paso 3: elegir el formulario

La mayoría de los disparadores muestran una lista **Form** con los formularios
de tu cuenta. Aunque el campo sea opcional, elige un formulario igual.

<aside class="warning">
Dejar <b>Form</b> en blanco crea una suscripción que no queda asociada a ningún formulario, y nunca se dispara. Si necesitas la misma automatización en varios formularios, crea un Zap por formulario en vez de un Zap sin formulario seleccionado.
</aside>

## Paso 4: probar y activar el Zap

El botón **Test trigger** de Zapier trae los últimos eventos que calzan con el
disparador, así que necesita al menos uno para mostrarte datos de ejemplo. Si
tu cuenta todavía no tiene ninguno, envía una respuesta de formulario, genera
un documento o asigna una tarea, según el disparador que elegiste, y prueba de
nuevo.

Cuando los datos de ejemplo se vean bien, agrega los pasos siguientes y
publica el Zap. La suscripción del lado de DataScope se crea al activar el
Zap, no mientras lo estás editando.

## Qué puedes automatizar

La app de DataScope entrega los siguientes disparadores. Los nombres son los
que ves en la lista de disparadores de Zapier:

| Disparador | Se activa cuando | Campo Form |
|---|---|---|
| Forms: New Form Entry | Se envía una respuesta de formulario | Obligatorio |
| Forms: New PDF | Se genera un PDF por respaldo por email o por autonotificación | Opcional, elige uno |
| Forms: Status Changed | Una respuesta de formulario cambia de estado | Opcional, elige uno |
| Tasks: New Assigned Task | Se asigna una tarea | Obligatorio |
| Signatures: New Completed Signature | Todos los firmantes firmaron el documento | Opcional, elige uno |
| Signatures: New Rejected Signature | Un firmante rechaza la solicitud de firma | Opcional, elige uno |
| Signatures: Updated Signature | Cambian las firmas de un documento y todavía quedan pendientes | Opcional, elige uno |
| Tickets: New Ticket (FKA Issue) | Se crea un ticket | No aplica, es a nivel de cuenta |
| Tickets: Changed Status (FKA Issue) | Un ticket cambia de estado | No aplica, es a nivel de cuenta |

Cada disparador entrega los datos del evento como campos individuales, listos
para mapear en los pasos siguientes del Zap sin tener que procesar el JSON
manualmente. En el caso de las respuestas de formulario, cada pregunta llega
como su propio campo, con su nombre, su tipo y su valor.

La app también entrega las siguientes actions, para que un Zap pueda actuar
sobre DataScope y no solo reaccionar a sus eventos:

| Action | Qué hace |
|---|---|
| Tasks: Assign Task V2 | Asigna una tarea de un formulario a un usuario, opcionalmente agendándola y vinculándola a una ubicación o a un lugar |
| Tasks: Assign Task V1 [Legacy] | La versión anterior de la misma action, se mantiene para que los Zaps existentes sigan funcionando |
| Forms: Send Data / New Answer [Beta] | Genera una nueva respuesta de formulario y su PDF a partir de un formulario existente usado como plantilla |
| Change Form Status | Cambia el estado de una respuesta de formulario, identificada por nombre y código de formulario |
| Modify Form Answer | Crea o modifica una respuesta puntual dentro de un formulario ya enviado |
| Tickets: Create Ticket (FKA Issue) | Crea un ticket, opcionalmente tomando sus valores por defecto de un Ticket Type |

### Assign Task: V1 o V2

Usa **V2** para todo lo nuevo. Hace todo lo que hace V1, y además:

- Acepta el signo más en el email del usuario, así que
  `usuario+obra@ejemplo.com` resuelve al usuario correcto.
- Puede apuntar al nuevo módulo de **Lugares**, no solo al módulo antiguo de
  Ubicaciones.
- Puede encontrar una ubicación o un lugar existente por su ID interno, que es
  lo que necesitas cuando varios lugares comparten el nombre.

**V1** sigue disponible para que los Zaps armados sobre ella sigan corriendo.
No hay migración automática: si quieres pasar un Zap de V1 a V2, reemplaza el
paso de la action y vuelve a mapear los campos.

### Ubicaciones o Lugares

**Tasks: Assign Task V2** y **Tickets: Create Ticket** tienen un campo
**Location Type** con dos opciones, y el comportamiento cambia de una forma
que conviene conocer antes de armar el Zap:

| Location Type | Comportamiento cuando nada calza |
|---|---|
| Locations (old module, legacy) | La ubicación se crea en DataScope a partir del nombre y de los demás campos de ubicación que enviaste |
| Places (new module) | No se crea nada. La tarea no se asigna y el Zap falla con `nestable location not found` |

Locations es la opción por defecto, así que los Zaps existentes siguen
funcionando sin cambios. Si eliges Places, crea el lugar en DataScope antes,
o a través de la API de lugares, antes de que corra el Zap.

En **Assign Task V2** la ubicación se busca en este orden: Location / Place
ID, después Location Code, después Location Name y después Location Address.
Gana la primera que calza.

El resto de los campos de ubicación, Location Phone, Location Email, Company
Name, Company Legal ID, Latitude y Longitude, aplican solo al módulo antiguo
de Ubicaciones, donde se guardan en la ubicación que se encontró o se creó. Se
ignoran con Places, porque los lugares nunca se modifican desde esta action.

### Send Data / New Answer está en Beta

**Forms: Send Data / New Answer** genera una respuesta de formulario y su PDF a
partir de un formulario existente, lo que la hace útil para traer a DataScope
datos capturados en otra parte. Dos límites que conviene tener en cuenta:

- **No dispara el resto de la cadena de automatización.** La sincronización con
  Google Sheets, la firma automática y otros webhooks, incluidos los
  disparadores de Zaps, no corren para una respuesta creada así. Un Zap que
  reacciona a `Forms: New Form Entry` no la va a ver.
- **No todos los tipos de pregunta están soportados por completo**, así que
  algunos campos pueden no quedar como esperas. Prueba con el formulario que
  vas a usar de verdad.

## Consideraciones importantes

- **Un Zap activo por formulario y por disparador.** Un disparador asociado a
  un formulario admite una sola suscripción activa por formulario. Activar un
  segundo Zap con el mismo disparador sobre el mismo formulario se rechaza con
  un error que nombra el tipo de disparador y te pide eliminar el anterior.
  Disparadores distintos sobre el mismo formulario no son problema, y tampoco
  el mismo disparador sobre formularios distintos.
- **Si necesitas que pasen varias cosas con un mismo evento, agrega pasos a un
  solo Zap** en vez de crear un segundo Zap con el mismo disparador y
  formulario.
- **Los disparadores de tickets operan a nivel de cuenta.** `Tickets: New
  Ticket` y `Tickets: Changed Status` cubren todos los tickets de la cuenta y
  admiten un Zap activo por cuenta cada uno. No tienen campo Form.
- **Los campos disponibles varían según tu configuración.** Algunos datos, como
  los de identidad de los firmantes o los campos extendidos de tareas,
  aparecen solo si esas funcionalidades están habilitadas en tu cuenta.
- **Los datos de ejemplo de un disparador no son la lista completa de campos.**
  Zapier arma la lista a partir de los eventos recientes, así que una pregunta
  que nadie respondió todavía, o un campo opcional que llegó vacío, puede no
  aparecer hasta que lo haga. Envía una respuesta representativa antes de
  mapear los pasos siguientes.
- **El disparador `Forms: New Form Entry` es visible del lado de DataScope.**
  Se registra en la lista de Webhooks de
  <a href="https://app.mydatascope.com/integrations" target="_blank" rel="noopener noreferrer">app.mydatascope.com/integrations</a>.
  El resto de los disparadores no aparece en esa lista.

## Si algo no funciona

- **La conexión falla al crearse:** verifica que la API Key esté completa y sin
  espacios al inicio o al final.
- **Al activar el Zap falla diciendo que el hook ya existe:** otro Zap de tu
  cuenta, quizás armado por otra persona, ya usa ese disparador sobre ese
  formulario. Apaga el Zap más antiguo y luego activa este. Si el error
  persiste, escríbenos y limpiamos la suscripción que quedó.
- **Test trigger no devuelve datos:** el disparador lee eventos recientes, así
  que una cuenta que todavía no tiene ninguno no tiene nada que mostrar. Genera
  uno y prueba de nuevo.
- **El Zap nunca se dispara:** confirma que haya un formulario seleccionado en
  el disparador, que el Zap esté activo, y revisa el historial del Zap en
  Zapier. Un evento que llegó mientras el Zap estaba apagado no se reenvía.
- **Un Zap que apagaste sigue recibiendo datos:** si el disparador es
  `Forms: New Form Entry`, abre la lista de Webhooks en
  <a href="https://app.mydatascope.com/integrations" target="_blank" rel="noopener noreferrer">app.mydatascope.com/integrations</a>
  y elimina ahí la entrada.
- **No aparecen los campos esperados:** algunos campos dependen de la
  configuración de tu cuenta. Escríbenos y lo revisamos contigo.

Para cualquier duda, escribe a soporte y te acompañamos en la configuración.
