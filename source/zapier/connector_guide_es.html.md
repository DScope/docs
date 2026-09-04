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

Pégala tal como DataScope te la muestra, sin espacios al inicio ni al final.
Si la conexión se acepta pero después el disparador no devuelve ningún dato, la
API Key es lo primero que conviene revisar.

La conexión pertenece a tu cuenta de Zapier y puedes reutilizarla en todos los
Zaps que armes. Cada persona que arme Zaps necesita su propia conexión, con la
API Key de su cuenta.

## Paso 3: elegir el formulario

La mayoría de los disparadores muestran una lista **Form** con los formularios
de tu cuenta. Aunque el campo sea opcional, elige un formulario igual.

<aside class="warning">
Dejar <b>Form</b> en blanco no te deja un Zap funcional: no entrega nada. Si necesitas la misma automatización en varios formularios, crea un Zap por formulario en vez de un Zap sin formulario seleccionado.
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
| Tickets: New Ticket | Se crea un ticket | No aplica, es a nivel de cuenta |
| Tickets: Changed Status | Un ticket cambia de estado | No aplica, es a nivel de cuenta |

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
| Tickets: Create Ticket | Crea un ticket, opcionalmente tomando sus valores por defecto de un Ticket Type |

### Assign Task: V1 o V2

Usa **V2** para todo lo nuevo. Hace todo lo que hace V1, y además:

- Acepta el signo más en el email del usuario, así que
  `usuario+obra@ejemplo.com` resuelve al usuario correcto.
- Puede apuntar al módulo de lugares nuevo, no solo al antiguo.
- Puede encontrar una ubicación o un lugar existente por su ID interno, que es
  lo que necesitas cuando varios lugares comparten el nombre.

**V1** sigue disponible para que los Zaps creados sobre ella sigan corriendo.
No hay migración automática: si quieres pasar un Zap de V1 a V2, reemplaza el
paso de la action y vuelve a mapear los campos.

<aside class="warning">
En las dos versiones, si el email o el nombre de usuario que envías no coincide con un usuario de tu cuenta, la tarea <b>no</b> se rechaza: se asigna a otro usuario de la cuenta. Mapea el campo de usuario desde un dato que controles, y revisa a quién quedó asignada la primera ejecución.
</aside>

### Locations o Places

**Tasks: Assign Task V2** y **Tickets: Create Ticket** tienen un campo
**Location Type** con dos opciones, y las dos acciones no se comportan igual. En
**Assign Task V2**:

<aside class="notice">
El campo <b>Location Type</b> llegó en la versión 2.1.3, que está liberada a cuentas seleccionadas. Si no lo ves en ninguna de las dos actions, tu Zap está en una versión anterior y apunta solo al módulo de lugares antiguo. Revisa el [Historial de versiones](#historial-de-versiones).
</aside>


| Location Type | Comportamiento cuando nada coincide |
|---|---|
| Locations (old module, legacy) | El lugar se crea en DataScope a partir del Location Name y de los demás campos de ubicación que enviaste. Sin Location Name no hay con qué crearlo, así que la tarea se asigna sin lugar |
| Places (new module) | No se crea nada. La tarea no se asigna y el Zap falla con `nestable location not found` |

**Tickets: Create Ticket** nunca crea un lugar, con ninguna de las dos opciones.
Si el lugar que envías no coincide con un registro existente, el ticket se crea
sin lugar asociado y el Zap reporta éxito. Solo busca por Location ID y después
por Location Name.

Locations es la opción por defecto, así que los Zaps existentes siguen
funcionando sin cambios. Si eliges Places, crea el lugar en DataScope antes de
que corra el Zap.

En **Assign Task V2** la ubicación se busca en este orden: Location / Place
ID, después Location Code, después Location Name y después Location Address.
Gana la primera que calza.

El resto de los campos de ubicación, Location Phone, Location Email, Company
Name, Company Legal ID, Latitude y Longitude, aplican solo al módulo de lugares
antiguo, donde se guardan en el lugar que se encontró o se creó. Se
ignoran con Places, porque los lugares nunca se modifican desde esta action.

### Send Data / New Answer está en Beta

**Forms: Send Data / New Answer** crea una respuesta de formulario a partir de un
formulario existente, lo que la hace útil para traer a DataScope datos capturados
en otra parte. Tres cosas que conviene tener en cuenta:

- **El PDF se genera solo si completas el campo Emails.** Ese campo es el que
  envía el documento, y generarlo es parte de enviarlo. Si lo dejas vacío,
  obtienes la respuesta de formulario sin PDF.
- **No dispara el resto de la cadena de automatización.** La sincronización con
  Google Sheets, la firma automática y los webhooks de respuestas de formulario
  no corren para una respuesta creada así, por lo que un Zap que reacciona a
  `Forms: New Form Entry` no la va a ver. `Forms: New PDF` es la excepción: si
  completas el campo Emails, ese Zap sí se dispara.
- **No todos los tipos de pregunta están soportados por completo**, así que
  algunos campos pueden no quedar como esperas. Prueba con el formulario que vas
  a usar de verdad.

## Consideraciones importantes

- **Un Zap activo por formulario y por disparador, con una excepción.**
  `Forms: New PDF`, `Forms: Status Changed`, `Tasks: New Assigned Task` y los
  tres disparadores de firma admiten una sola suscripción activa por formulario.
  Activar un segundo Zap con el mismo disparador sobre el mismo formulario se
  rechaza con un error que nombra el tipo de disparador y te pide eliminar el
  anterior. Disparadores distintos sobre el mismo formulario no son problema, y
  tampoco el mismo disparador sobre formularios distintos.
  **`Forms: New Form Entry` no pasa por esa validación**: un segundo Zap sobre el
  mismo formulario se acepta, y después los dos reciben todos los envíos. Agrega
  pasos a un solo Zap en vez de eso.
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

- **La conexión se acepta pero nada funciona:** la prueba de conexión no rechaza
  de forma confiable una API Key incorrecta, así que una clave mala aparece más
  tarde como un disparador que no devuelve ningún dato. Vuelve a copiar la clave
  desde la sección Integraciones, sin espacios al inicio ni al final, y reemplaza
  la conexión.
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

## Historial de versiones

Versiones de la app de DataScope en Zapier, de la más nueva a la más antigua. Un
Zap sigue corriendo con la versión con la que fue creado, así que un Zap antiguo
puede comportarse distinto a uno nuevo creado con el mismo disparador, y en
cualquiera de los dos casos no hay nada que instalar ni actualizar de tu lado.
Algunas versiones se liberan a cuentas seleccionadas antes de pasar a ser la
predeterminada para los Zaps nuevos, así que la versión detrás de tu Zap no
siempre es la actual.

Dos números, 2.0.0 y 2.1.0, nunca se publicaron. Por eso las notas de
publicación registradas en 2.0.1 y 2.1.1 describen solo la diferencia respecto a
un draft que ya no existe. Los bullets de abajo describen, en cambio, la
diferencia respecto a la versión publicada anterior.

<aside class="notice">
DataScope registra la versión que reporta la app cuando activas el Zap y se crea la suscripción, y usa ese número para decidir por dónde entrega los eventos: por debajo de 2.0.0 la vía anterior, en 2.0.0 o superior la vía actual. Como nunca se publicó una 2.0.0, en la práctica todos los Zaps de la línea 1.14 usan la vía anterior y todos los de la línea 2.x usan la actual. El número registrado solo cambia cuando la suscripción se vuelve a crear, así que apagar y volver a activar un Zap lo vuelve a registrar. <code>Forms: New Form Entry</code> es anterior a las suscripciones con versión y siempre usa la vía anterior.
</aside>

<!--
Plantilla de entrada del historial. La más nueva va primero: agrega la versión
nueva como el primer bloque debajo de este comentario, y agrega el mismo bloque
en connector_guide_en, connector_guide_es y connector_guide_pt para que los tres
idiomas queden iguales. Una entrada es el número de versión en negrita,
**X.Y.Z**, opcionalmente seguido de un paréntesis con su estado de liberación,
después una línea en blanco, después un bullet por cambio. Prioriza la nota de
publicación de la versión en la plataforma de desarrollo de Zapier; si la nota
falta o se queda corta, diffea la definición de esa versión contra la publicada
anterior y describe eso. Describe solo lo que el lector puede verificar en la
app o en esta guía. Nunca publiques la cantidad de usuarios ni de tareas por
versión. Agrega una fecha solo si se conoce la fecha de publicación.
-->

**2.1.3** (liberada a cuentas seleccionadas)

- Agrega el campo `Location Type` en `Tasks: Assign Task V2` y en
  `Tickets: Create Ticket`, con la opción `Places (new module)` junto a la de
  Locations antigua.
- Agrega `Find Location / Place by ID` en `Tasks: Assign Task V2`, para que una
  tarea pueda apuntar a un lugar existente por su ID interno.
- Saca el sufijo `(FKA Issue)` de los dos disparadores de tickets y de
  `Tickets: Create Ticket`. Un Zap creado en una versión anterior sigue
  mostrando el nombre antiguo.
- `User Email` en `Tasks: Assign Task V2` y `Form Name` en
  `Change Form Status` pasan a ser listas desplegables, así eliges el usuario o
  el formulario en vez de escribirlo.
- Renombra etiquetas de campos de salida en
  `Signatures: New Completed Signature`. `Signer Rut` ahora es
  `Signer National ID`, y `If Signer is a Datascope User` ahora es
  `Signer Is External User`, que es lo que el campo realmente informa. Las
  claves detrás de las etiquetas no cambian, así que los mapeos existentes
  siguen funcionando.
- Reescribe los textos de ayuda de la mayoría de los campos, incluidos todos
  los campos de ubicación de `Tasks: Assign Task V2`, que ahora indican a qué
  módulo aplica cada uno. En `Modify Form Answer` corrige la etiqueta
  `From Code` a `Form Code`, y la descripción de
  `Tasks: Assign Task V1 [Legacy]` ahora dice que la V2 es la que hay que usar.

**2.1.2** (predeterminada para los Zaps nuevos)

- Agrega la action `Tasks: Assign Task V2` y renombra la anterior como
  `Tasks: Assign Task V1 [Legacy]`. La V2 acepta direcciones de email de usuario
  que contienen el signo más.

**2.1.1**

- Renombra los dos disparadores de tickets. `Findings: New Finding` pasa a
  `Tickets: New Ticket (FKA Issue)`, y `Findings: Changed Status` pasa a
  `Tickets: Changed Status (FKA Issue)`.
- Agrega la action `Tickets: Create Ticket (FKA Issue)`.
- Corrige un error con la fecha de creación y con la fecha de expiración
  relativa.

**2.0.2**

- El Location Name deja de ser obligatorio en la action `Assign Task`.
- Cambia algunos tipos de campos de salida.

**2.0.1**

- La primera versión publicada de la línea 2.x, y la primera con los tres
  disparadores de firma y los dos de tickets, que en ese momento se llamaban
  `Findings: New Finding` y `Findings: Changed Status`.
- Consolida lo que los drafts de 1.14 habían agregado por separado: el
  disparador de estado, la action `Send Data`, y los campos de obligatoriedad y
  de respuesta tardía en `Assign Task`.
- Antepone su área al nombre de cada disparador y de cada action: `Forms:`,
  `Tasks:` o `Signatures:`.
- Mejora un valor por defecto.

**1.14.3**

- Agrega `Make this task mandatory` y `Allow the task to be answered after the
  due date has been reached` a la action `Assign Task`.
- No trae el disparador `Status Changed` ni la action `Send Data` que sí tiene
  1.14.2, así que un Zap en esta versión tiene tres disparadores y tres actions.

**1.14.2**

- Agrega la action `Send Data`.
- Renombra el disparador `Change Status` como `Status Changed`, y su campo Form
  pasa a ser opcional.

**1.14.1**

- Agrega el disparador `Change Status`.

**1.14.0**

- La base de la línea 1.14: `New Form Entry`, `New Assigned Task` y `New PDF`
  como disparadores, y `Assign Task`, `Change Form Status` y
  `Modify Form Answer` como actions. Acá `Assign Task` exige un Location Name.
- Los tres disparadores de firma, los dos de tickets, `Assign Task V2` y
  `Create Ticket` existen solo en la línea 2.x. Para usar cualquiera de ellos,
  vuelve a crear el Zap para que tome la versión actual.
