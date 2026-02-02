# Validación y reglas de diseño AsyncAPI
Este documento establece el marco normativo y las reglas de validación obligatorias para garantizar la consistencia, seguridad y calidad de todas las APIs y arquitecturas orientadas a eventos de la organización.

## Sistema de Reglas
El sistema de reglas organiza las reglas en 5 categorías, estas son Estructura, Headers, Formato y Claridad.

**Estructura:** Cumplimiento del esquema `AsyncAPI`

**Headers:** Cumplimiento especifico del esquema de Headers definido por la empresa. Se evalúa la presencia de los headers `eventId`, `eventType`, `entityId`, `entityType`, `timestamp` y `datetime`.

**Nomenclatura:** Consistencia en nombres de canales, operaciones y variables.

**Formato:** `JSON/YAML` válido y con formato estándar

**Claridad:** Documentación suficiente de operaciones y mensajes

---

## Estructura General de las reglas
Cada regla dentro del validador tiene la siguiente estructura:

**Nombre:** Identificador único de la regla.

**Severidad:** Define si la regla es un error (bloqueante) o una advertencia (warning).

**Descripción:** Explica el propósito de la regla y la lógica para analizarla.

**Expresión:** Referencia a la función que realiza la verificación. Esta puede ser una función o simplemente la expresión a evaluar.

**Mensaje:** Texto que se muestra cuando no se cumple la regla.

Para aprobar la validación, el documento debe cumplir con los criterios de aceptación definidos al final de este documento.

---

## Severidad de las Reglas
Las reglas se clasifican en dos niveles de severidad que determinan el resultado de la validación:

**Error (error):** Infracción crítica. Si una regla de este tipo no se cumple, la especificación se considera **INVÁLIDA** inmediatamente. Estas reglas garantizan la integridad estructural, la seguridad y la operatividad de la API.

**Advertencia (warning):** Recomendación o buena práctica. El incumplimiento de estas reglas genera una alerta, pero **NO impide** la aprobación del documento. Sin embargo, se espera que los equipos resuelvan estas advertencias en iteraciones futuras.

Un ejemplo de una regla Error sería la regla de 'Canales definidos'. Esta se clasifica como error porque, si no está presente, el archivo AsyncAPI carece de funcionalidad estructural; por lo tanto, su incumplimiento hará que la validación se marque como **RECHAZADA**, generando alertas críticas que deben atenderse, aunque el despliegue continúe por configuración.

Un caso de una regla con Advertencia (warning) es 'Descripción de la API presente'. Que la descripción exista es una buena práctica que mejora la legibilidad, pero su ausencia no rompe la integración. Por lo tanto, el sistema generará una alerta, pero permitirá que el documento sea aprobado.

---

### Reglas de Estructura
Las reglas de estructura evalúan la organización y coherencia general del documento `AsyncAPI`. Se tiene un total de 19 reglas de esta categoría.

| Regla | Severidad | Impacto en Validación | Descripción |
| :--- | :---: | :---: | :--- |
| **ID de la API válido** | ⛔ **Error** | **Bloqueante** | El id de la API debe ser la URL del repositorio GitHub del producer. |
| **Título de la API presente y descriptivo** | ⛔ **Error** | **Bloqueante** | La especificación debe tener un título (info.title) con más de 10 caracteres. |
| **Versión de la API presente** | ⛔ **Error** | **Bloqueante** | La especificación debe incluir info.version. |
| **Versión de AsyncAPI** | ⛔ **Error** | **Bloqueante** | La especificación debe indicar que usa la versión 3.0.0 o superior de AsyncAPI. |
| **Canales definidos** | ⛔ **Error** | **Bloqueante** | Debe existir al menos un canal en la sección channels. |
| **Operaciones definidas** | ⛔ **Error** | **Bloqueante** | Debe existir al menos una operación (operations). |
| **Acción de la operación válida (send/receive)** | ⛔ **Error** | **Bloqueante** | La acción de cada operación debe ser 'send' o 'receive'. |
| **Mensajes definidos en cada canal** | ⛔ **Error** | **Bloqueante** | Cada canal debe tener al menos un mensaje definido. |
| **Payload del mensaje definido** | ⛔ **Error** | **Bloqueante** | Cada mensaje debe incluir un payload. |
| **Schemas definidos** | ⛔ **Error** | **Bloqueante** | La especificación debe definir al menos un schema en components.schemas. |
| **Tipo de schema 'object'** | ⛔ **Error** | **Bloqueante** | Cada schema debe tener el type igual a 'object'. |
| **Propiedades del schema definidas** | ⛔ **Error** | **Bloqueante** | Cada schema debe contar con al menos una propiedad en su sección properties. |
| **Propiedades del schema con tipo** | ⛔ **Error** | **Bloqueante** | Cada propiedad de los schemas debe especificar un tipo. |
| **Evitar duplicación de datos** | ⚠️ **Warning** | Informativo | No duplicar la misma información en headers y body (ej. timestamp, entityId, version). |
| **Booleans no nulos** | ⛔ **Error** | **Bloqueante** | Los campos booleanos no pueden tener valor null. |
| **No incluir JSON en campos String** | ⛔ **Error** | **Bloqueante** | Si se requiere un objeto JSON, definirlo como type: object, no como una cadena con JSON embebido. |
| **Respetar tipos definidos** | ⛔ **Error** | **Bloqueante** | Cada propiedad debe cumplir con el tipo y formato estipulado en el contrato (ej. epoch, ISO8601). |
| **Uso de Message Traits (Headers)** | ⛔ **Error** | **Bloqueante** | Los traits para headers (commonHeaders, transportHeaders, businessHeaders, trackingHeaders) deben existir. |
| **Campos no duplicados en schemas Body** | ⚠️ **Warning** | Informativo | Verifica que no existan propiedades repetidas en schemas que terminen con 'Body'. |

---

### Reglas de Headers
Se encarga se evaluar la presencia de todos los headers importantes definidos por la empresa.

| Regla | Severidad | Impacto en Validación | Descripción |
| :--- | :---: | :---: | :--- |
| **Headers comunes: eventId presente** | ⛔ **Error** | **Bloqueante** | El header común `eventId` es obligatorio. |
| **Headers comunes: eventType presente** | ⛔ **Error** | **Bloqueante** | El header común `eventType` es obligatorio. |
| **Headers comunes: entityId presente** | ⛔ **Error** | **Bloqueante** | El header común `entityId` es obligatorio. |
| **Headers comunes: entityType presente** | ⛔ **Error** | **Bloqueante** | El header común `entityType` es obligatorio. |
| **Headers comunes: timestamp presente** | ⛔ **Error** | **Bloqueante** | El header común `timestamp` es obligatorio. |
| **Headers comunes: datetime presente** | ⛔ **Error** | **Bloqueante** | El header común `datetime` es obligatorio. |
| **Headers comunes: version presente** | ⛔ **Error** | **Bloqueante** | El header común `version` es obligatorio. |
| **Headers comunes: internal presente** | ⛔ **Error** | **Bloqueante** | El header común `internal` es obligatorio. |
| **Header de transporte 'channel' presente** | ⛔ **Error** | **Bloqueante** | Asegurar que `transportHeaders` incluya al menos la clave `channel`. |
| **Headers de negocio presentes y válidos** | ⛔ **Error** | **Bloqueante** | En `businessHeaders`, deben existir `domain`, `subdomain`, `businessCapability`. |
| **Headers de seguimiento presentes** | ⛔ **Error** | **Bloqueante** | En `trackingHeaders`, deben existir `traceId` y `spanId` (parentSpanId es opcional). |

---

### Reglas de Formato
Estas reglas garantizan que el documento sea sintácticamente correcto y legible por máquinas.

| Regla | Severidad | Impacto en Validación | Descripción |
| :--- | :---: | :---: | :--- |
| **Documento JSON/YAML válido y bien indentado** | ⛔ **Error** | **Bloqueante** | El documento debe ser parseable y estar correctamente indentado. |
| **Uso correcto de comillas dobles en JSON** | ⛔ **Error** | **Bloqueante** | En JSON, las cadenas deben llevar comillas dobles, no simples. |
| **Uso adecuado de llaves y corchetes** | ⛔ **Error** | **Bloqueante** | Los objetos deben usar `{}` y los arrays `[]`, sin mezclar formatos. |
| **Separación de claves y valores** | ⛔ **Error** | **Bloqueante** | En JSON, los campos deben separarse correctamente con `:` y `,`. |
| **No incluir valores nulos o vacíos** | ⚠️ **Warning** | Informativo | Se deben evitar valores `null` o cadenas vacías en la especificación. |
| **Campos y propiedades en inglés** | ⛔ **Error** | **Bloqueante** | Todas las llaves de schemas, messages y properties deben estar en inglés. El contenido puede estar en español. Se permite el uso de `info`. |

---

### Reglas de Claridad
Las reglas de claridad se encargan de asegurar la claridad y precisión de la documentación y la información proporcionada.

| Regla | Severidad | Impacto en Validación | Descripción |
| :--- | :---: | :---: | :--- |
| **Descripción de la API presente** | ⚠️ **Warning** | Informativo | Se recomienda que `info.description` no esté vacío y sea explicativo. |
| **Descripción de la operación presente (summary)** | ⚠️ **Warning** | Informativo | Cada operación en `operations` debe incluir un `summary`. |
| **Propiedades del schema con descripción** | ⚠️ **Warning** | Informativo | Se recomienda que cada propiedad en `components.schemas` tenga una `description`. |
| **Evitar repetición entre headers y body** | ⚠️ **Warning** | Informativo | No repetir campos como `timestamp` o `entityId` si ya están en headers. |
| **Datos personales mínimos** | ⛔ **Error** | **Bloqueante** | Evitar exponer datos sensibles sin anonimización o identificadores alternativos. |
| **Consistencia de tipos** | ⛔ **Error** | **Bloqueante** | Un mismo campo debe mantener el mismo tipo en todos los schemas. |
| **Logs y mensajes de error sin datos personales** | ⛔ **Error** | **Bloqueante** | No incluir datos personales en logs o mensajes de error; usar identificadores en su lugar. |

---

### Reglas de Nomenclatura
Estas reglas se encargan de verificar la consistencia y claridad en los nombre de los elementos del `AsyncAPI`.

| Regla | Severidad | Impacto en Validación | Descripción |
| :--- | :---: | :---: | :--- |
| **lowerCamelCase** | ⛔ **Error** | **Bloqueante** | Los nombres de campos, propiedades y mensajes deben seguir la convención lowerCamelCase, excepto los nombres de canales. |
| **Nombres de arrays en plural** | ⚠️ **Warning** | Informativo | Las propiedades con type: array deben terminar en "s". |
| **Claves JSON sin números al principio** | ⛔ **Error** | **Bloqueante** | Ninguna clave debe comenzar con un número. |
| **Sufijo 'Code' para campos de códigos** | ⛔ **Error** | **Bloqueante** | Si la descripción menciona "código" o "code", el nombre del campo debe terminar en "Code". |
| **Nombres claros y evitar acrónimos** | ⚠️ **Warning** | Informativo | Evitar acrónimos no documentados y usar nombres descriptivos en inglés. |
| **No usar palabras reservadas** | ⛔ **Error** | **Bloqueante** | Prohibido el uso de palabras como message, body, payload, class, default, function. |

---

### Criterios de Aceptación y Validación

El nuevo modelo de validación se basa en un enfoque de cumplimiento estricto. Para que un archivo AsyncAPI sea aceptado en el ecosistema de la organización, debe cumplir con los siguientes criterios.

### Estados de Validación

El validador arrojará uno de los siguientes tres estados finales. Actualmente, el pipeline está configurado en modo **Informativo** para los errores.

| Estado | Resultado | Criterio Técnico | Acción en Pipeline |
| :--- | :---: | :--- | :--- |
| **✅ APROBADO** | **CUMPLE** | **0** Errores <br> **0** Advertencias | **Continuar.** <br> El documento se procesa y despliega automáticamente. |
| **⚠️ CON OBSERVACIONES** | **CUMPLE** | **0** Errores <br> **≥1** Advertencias | **Continuar.** <br> Se permite el despliegue, pero se notifica al equipo para futuras correcciones. |
| **⛔ RECHAZADO** | **NO CUMPLE** | **≥1** Errores | **Continuar (Con Alerta).** <br> El documento es inválido y contiene errores bloqueantes. Se publica el reporte en el PR pero **NO se detiene**. |
