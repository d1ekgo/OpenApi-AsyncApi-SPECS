# Validación y reglas de diseño OpenAPI

Este documento establece el marco normativo y las reglas de validación obligatorias para garantizar la consistencia, seguridad y calidad de todas las APIs REST y Webhooks documentadas mediante OpenAPI dentro de la organización.

---

## Sistema de Reglas
El sistema organiza las validaciones en 5 dominios críticos:

* **Estructura:** Garantiza la integridad del esquema según el estándar de arquitectura organizacional.
* **Headers:** Asegura la trazabilidad y el cumplimiento del modelo de mensajería corporativo mediante cabeceras obligatorias.
* **Nomenclatura:** Establece los estándares semánticos para paths, operaciones, parámetros y esquemas.
* **Formato:** Valida la correcta sintaxis, tipado y legibilidad del documento fuente.
* **Claridad:** Asegura que la documentación técnica sea detallada y autosuficiente para facilitar el consumo por parte de terceros.

---

## Estructura General de las Reglas
Cada regla dentro del motor de validación contiene los siguientes atributos:

- **Nombre:** Identificador único de la regla.
- **Severidad:** Define si el hallazgo es un error crítico (`error`) o una buena práctica (`warning`).
- **Descripción:** Especificación técnica detallada que explica la lógica y el propósito de la validación.
- **Mensaje:** Notificación descriptiva que recibe el desarrollador para resolver el incumplimiento.

---

## Severidad de las Reglas
* **Error (error):** Infracción crítica que afecta la interoperabilidad, seguridad o generación de código. Si una regla de este tipo no se cumple, la especificación se considera **INVÁLIDA**.
* **Advertencia (warning):** Recomendación de diseño o buena práctica documental. El incumplimiento genera una alerta pero **no invalida** el documento.

---

## Reglas de Estructura

| Regla | Severidad | Descripción Técnica |
| :--- | :---: | :--- |
| **ID de la API válido** | ⛔ Error | El identificador de la API debe corresponder estrictamente a la URL del repositorio Git donde se aloja el código fuente del servicio. |
| **Título descriptivo** | ⛔ Error | El campo `info.title` debe ser representativo de la funcionalidad del servicio y poseer una longitud mayor a 10 caracteres para evitar nombres genéricos. |
| **Versión de la API** | ⛔ Error | La presencia del campo `info.version` es obligatoria para el control del ciclo de vida y la gestión de cambios del contrato. |
| **Versión de OpenAPI** | ⛔ Error | La especificación debe declarar versiones compatibles con el estándar corporativo actual: `3.0.3` o `3.1.x`. |
| **Paths definidos** | ⛔ Error | El objeto raíz `paths` debe contener al menos un punto de entrada (endpoint) para que la especificación sea considerada funcional. |
| **Operaciones** | ⛔ Error | Cada ruta declarada en `paths` debe definir al menos un método HTTP válido (`get`, `post`, `put`, `delete`, `patch`). |
| **Responses definidos** | ⛔ Error | Todas las operaciones deben incluir obligatoriamente el objeto `responses` con sus respectivos códigos de estado HTTP. |
| **Respuesta de éxito** | ⛔ Error | Cada operación debe definir, como mínimo, una respuesta satisfactoria dentro del rango HTTP `2xx` para asegurar un flujo de terminación exitoso. |
| **Contrato de entrada** | ⛔ Error | Las operaciones que implican creación o modificación de recursos (`post`, `put`, `patch`) deben definir explícitamente un `requestBody`. |
| **Schema de nivel superior** | ⛔ Error | Los esquemas utilizados como raíz en un `requestBody` o en un `response` exitoso deben ser estrictamente de tipo `type: object`. |
| **Schemas reutilizables** | ⚠️ Warning | Se permite que los esquemas internos o anidados utilicen `type: array` o tipos simples para facilitar la modularidad. |
| **Definición de Schemas** | ⛔ Error | La especificación debe contener al menos un esquema reutilizable dentro de la sección `components.schemas`. |
| **Propiedades de objetos** | ⛔ Error | Todo esquema declarado como `type: object` debe definir explícitamente sus propiedades mediante el atributo `properties`. |
| **Tipado obligatorio** | ⛔ Error | Cada propiedad definida dentro de un esquema debe declarar mandatoriamente su atributo `type`. |
| **Booleans no nulos** | ⛔ Error | Los campos de tipo `boolean` no pueden aceptar valores `null`. Se deben definir como obligatorios o incluir un valor por defecto. |
| **Prohibición de JSON embebido** | ⛔ Error | No se permite el modelado de estructuras complejas mediante cadenas de texto (`type: string`). Se debe utilizar la jerarquía de objetos de OpenAPI. |
| **Estructuras dinámicas** | ⚠️ Warning | Para campos con contenido variable o dinámico, se recomienda el uso de combinadores como `oneOf`, `anyOf` o `allOf`. |

---

## Reglas de Headers
Los headers corporativos garantizan la trazabilidad cross-platform. Deben definirse una única vez en `components.parameters` y ser referenciados mediante `$ref` en cada operación.

| Header | Severidad | Descripción Técnica |
| :--- | :---: | :--- |
| `eventId` | ⛔ Error | Identificador único universal (UUID) requerido para realizar el seguimiento transaccional de cada petición en los logs. |
| `eventType` | ⛔ Error | Define la categoría o nombre de la acción técnica que se está ejecutando. |
| `entityId` | ⛔ Error | Identificador de negocio del recurso principal sobre el cual impacta la operación. |
| `entityType` | ⛔ Error | Clasificación del dominio o tipo de entidad al que pertenece el identificador de negocio proporcionado. |
| `timestamp` | ⛔ Error | Marca de tiempo en formato Unix Epoch (milisegundos) que registra el instante preciso de la petición. |
| `datetime` | ⛔ Error | Fecha y hora de la transacción representada bajo el estándar ISO-8601, incluyendo el desplazamiento de zona horaria. |

---

## Reglas de Nomenclatura

| Regla | Severidad | Descripción Técnica |
| :--- | :---: | :--- |
| **Estrategia CamelCase** | ⛔ Error | Los nombres de todas las propiedades, parámetros de consulta (query) y claves de esquemas deben seguir la convención `lowerCamelCase`. |
| **Pluralización de Arrays** | ⚠️ Warning | Las propiedades que representen colecciones o listas de elementos (`type: array`) deben ser nombradas en plural (ej. `items`, `users`). |
| **Sin números iniciales** | ⛔ Error | Las claves de los esquemas y nombres de propiedades no pueden iniciar con caracteres numéricos para evitar errores de compilación en SDKs. |
| **Sufijo Code** | ⛔ Error | Cualquier propiedad que represente un código clasificatorio o de negocio debe finalizar obligatoriamente con el sufijo `Code` (ej. `currencyCode`). |
| **Uso de Acrónimos** | ⚠️ Warning | Se recomienda evitar el uso de acrónimos crípticos; los nombres deben ser descriptivos para mejorar la legibilidad del contrato. |
| **Palabras Reservadas** | ⛔ Error | Prohibido el uso de términos reservados por lenguajes de programación en nombres de propiedades: `message`, `body`, `payload`, `class`, `default`, `function`. |

---

## Reglas de Claridad y Documentación

| Regla | Severidad | Descripción Técnica |
| :--- | :---: | :--- |
| **Descripción de la API** | ⚠️ Warning | El campo `info.description` debe proveer una explicación detallada sobre el propósito de negocio y el alcance técnico de la API. |
| **Resumen de Operación** | ⚠️ Warning | Cada método HTTP debe incluir un campo `summary` que describa de forma breve y concisa la acción realizada. |
| **Documentación de Parámetros** | ⚠️ Warning | Todos los parámetros (path, query, header) deben contar con el atributo `description` explicando su uso y restricciones. |
| **Documentación de Schemas** | ⚠️ Warning | Se recomienda que cada propiedad dentro de `components.schemas` incluya una descripción clara sobre el dato que almacena. |
| **Uso de Ejemplos** | ⚠️ Warning | Es altamente recomendable incluir el campo `example` o `examples` en los esquemas para facilitar las pruebas y el mocking de la API. |
| **Privacidad de Datos (PII)** | ⛔ Error | Queda estrictamente prohibida la exposición de datos personales o sensibles en nombres de campos, descripciones o ejemplos. |
| **Seguridad en Errores** | ⛔ Error | Los mensajes de error y esquemas de respuesta de falla no deben contener datos sensibles de los usuarios ni información interna del sistema. |

---

## Criterios de Aceptación y Validación

| Estado | Resultado | Criterio Técnico | Acción en Pipeline |
| :--- | :---: | :--- | :--- |
| **✅ APROBADO** | **CUMPLE** | **0** Errores <br> **0** Advertencias | **Continuar.** El documento se procesa y despliega automáticamente. |
| **⚠️ CON OBSERVACIONES** | **CUMPLE** | **0** Errores <br> **≥1** Advertencias | **Continuar.** Se permite el despliegue, pero se notifica al equipo de desarrollo las observaciones. |
| **⛔ RECHAZADO** | **NO CUMPLE** | **≥1** Errores | **Continuar (Con Alerta).** El documento contiene errores técnicos bloqueantes que deben ser corregidos obligatoriamente, aunque por el momento no detiene el flujo. |
