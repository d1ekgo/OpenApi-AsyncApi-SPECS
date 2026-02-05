# Validación y reglas de diseño OpenAPI

Este documento establece el marco normativo y las reglas de validación obligatorias para garantizar la consistencia, seguridad y calidad de todas las APIs REST y Webhooks documentadas mediante OpenAPI dentro de la organización.

---

## Sistema de Reglas
El sistema de reglas organiza las validaciones en 5 categorías:

**Estructura:** Cumplimiento del esquema `OpenAPI` y consistencia mínima para que el contrato sea funcional.

**Headers:** Cumplimiento específico del esquema de headers corporativos. Se evalúa la presencia y el uso consistente de los headers `eventId`, `eventType`, `entityId`, `entityType`, `timestamp` y `datetime`.

**Nomenclatura:** Consistencia en nombres de paths, operaciones, parámetros y schemas.

**Formato:** Documento `JSON/YAML` válido y con formato estándar.

**Claridad:** Documentación suficiente de operaciones, parámetros y modelos.

---

## Estructura General de las Reglas
Cada regla dentro del validador tiene la siguiente estructura:

- **Nombre:** Nombre legible de la regla.
- **Severidad:** Define si la regla es un error (bloqueante) o una advertencia (warning).
- **Impacto en Validación:** Indica si el hallazgo debe considerarse **Bloqueante** o **Informativo**.
- **Descripción:** Explica el propósito de la regla y la lógica para analizarla.
- **Expresión:** Referencia a la función/regla que realiza la verificación (o expresión lógica evaluada).
- **Mensaje:** Texto que se muestra cuando no se cumple la regla.

Para aprobar la validación, el documento debe cumplir con los criterios de aceptación definidos al final de este documento.

---

## Severidad de las Reglas
Las reglas se clasifican en dos niveles de severidad que determinan el resultado de la validación:

**Error (error):** Infracción crítica que afecta la interoperabilidad, seguridad o generación de código. Si una regla de este tipo no se cumple, la especificación se considera **INVÁLIDA**.

**Advertencia (warning):** Recomendación o buena práctica documental. El incumplimiento de estas reglas genera una alerta, pero **NO impide** la aprobación del documento.

> Nota: Actualmente el pipeline está configurado en modo **Informativo** para los errores (se reportan como bloqueantes, pero **no detienen** el flujo).

---

## Reglas de Estructura
Las reglas de estructura evalúan la organización y coherencia general del documento `OpenAPI`.

| Regla | Severidad | Impacto en Validación | Descripción | Expresión (referencia) |
| :--- | :---: | :---: | :--- | :--- |
| **ID de la API válido** | ⛔ **Error** | **Bloqueante** | El identificador de la API debe corresponder estrictamente a la URL del repositorio Git donde se aloja el código fuente del servicio. | `info.x-service-id == repoUrl` |
| **Título descriptivo** | ⛔ **Error** | **Bloqueante** | El campo `info.title` debe ser representativo de la funcionalidad del servicio y poseer una longitud mayor a 10 caracteres para evitar nombres genéricos. | `len(info.title) > 10` |
| **Versión de la API** | ⛔ **Error** | **Bloqueante** | La presencia del campo `info.version` es obligatoria para el control del ciclo de vida y la gestión de cambios del contrato. | `exists(info.version)` |
| **Versión de OpenAPI** | ⛔ **Error** | **Bloqueante** | La especificación debe declarar versiones compatibles con el estándar corporativo actual: `3.0.3` o `3.1.x`. | `openapi in ["3.0.3", /^3\.1\./]` |
| **Paths definidos** | ⛔ **Error** | **Bloqueante** | El objeto raíz `paths` debe contener al menos un punto de entrada (endpoint). | `count(paths.*) >= 1` |
| **Operaciones válidas** | ⛔ **Error** | **Bloqueante** | Cada ruta declarada en `paths` debe definir al menos un método HTTP válido (`get`, `post`, `put`, `delete`, `patch`). | `forEachPath: hasAny(get,post,put,delete,patch)` |
| **Responses definidos** | ⛔ **Error** | **Bloqueante** | Todas las operaciones deben incluir obligatoriamente el objeto `responses` con sus respectivos códigos de estado HTTP. | `exists(operation.responses)` |
| **Respuesta de éxito** | ⛔ **Error** | **Bloqueante** | Cada operación debe definir al menos una respuesta satisfactoria dentro del rango HTTP `2xx`. | `exists(responses[/^2\d\d$/])` |
| **Contrato de entrada** | ⛔ **Error** | **Bloqueante** | Operaciones que implican creación o modificación (`post`, `put`, `patch`) deben definir `requestBody`. | `if method in [post,put,patch] -> exists(requestBody)` |
| **Schema de nivel superior** | ⛔ **Error** | **Bloqueante** | Los schemas usados como raíz en un `requestBody` o en una respuesta `2xx` deben ser `type: object`. | `topLevelSchema.type == "object"` |
| **Schemas reutilizables (permitido)** | ⚠️ **Warning** | Informativo | Se permite que los esquemas internos o anidados utilicen `type: array` o tipos simples para facilitar la modularidad. | `advisory()` |
| **Definición de Schemas** | ⛔ **Error** | **Bloqueante** | La especificación debe contener al menos un esquema reutilizable dentro de `components.schemas`. | `count(components.schemas.*) >= 1` |
| **Propiedades de objetos** | ⛔ **Error** | **Bloqueante** | Todo schema declarado como `type: object` debe definir `properties`. | `if schema.type=="object" -> exists(schema.properties)` |
| **Tipado obligatorio** | ⛔ **Error** | **Bloqueante** | Cada propiedad definida dentro de un schema debe declarar `type`. | `forEachProperty: exists(type)` |
| **Booleans no nulos** | ⛔ **Error** | **Bloqueante** | Los campos `boolean` no pueden aceptar `null`. | `if type=="boolean" -> nullable != true` |
| **Prohibición de JSON embebido** | ⛔ **Error** | **Bloqueante** | No se permite modelar estructuras complejas mediante `type: string` (JSON embebido). | `noJsonStringHeuristic()` |
| **Estructuras dinámicas** | ⚠️ **Warning** | Informativo | Para campos con contenido variable o dinámico, se recomienda `oneOf`, `anyOf` o `allOf`. | `advisory()` |

---

## Reglas de Headers
Los headers corporativos garantizan la trazabilidad cross-platform. Deben definirse una única vez en `components.parameters` y ser referenciados mediante `$ref` en cada operación.

### Catálogo de headers corporativos (referencia)
- `eventId`: UUID requerido para seguimiento transaccional.
- `eventType`: categoría/acción técnica ejecutada.
- `entityId`: identificador de negocio del recurso principal.
- `entityType`: clasificación del dominio o tipo de entidad.
- `timestamp`: Unix Epoch en milisegundos.
- `datetime`: ISO-8601 con zona horaria.

### Reglas de validación de headers
| Regla | Severidad | Impacto en Validación | Descripción | Expresión (referencia) |
| :--- | :---: | :---: | :--- | :--- |
| **Headers definidos en components.parameters** | ⛔ **Error** | **Bloqueante** | Los headers corporativos deben existir en `components.parameters` (`in: header`) y ser reutilizables. | `headersExistInComponentsParameters()` |
| **Headers referenciados por operación** | ⛔ **Error** | **Bloqueante** | Cada operación debe referenciar los headers corporativos mediante `$ref` en `parameters`. | `forEachOperation: hasRefsToCorporateHeaders()` |
| **eventId presente** | ⛔ **Error** | **Bloqueante** | Debe existir el header `eventId`. | `headerExists("eventId")` |
| **eventType presente** | ⛔ **Error** | **Bloqueante** | Debe existir el header `eventType`. | `headerExists("eventType")` |
| **entityId presente** | ⛔ **Error** | **Bloqueante** | Debe existir el header `entityId`. | `headerExists("entityId")` |
| **entityType presente** | ⛔ **Error** | **Bloqueante** | Debe existir el header `entityType`. | `headerExists("entityType")` |
| **timestamp presente** | ⛔ **Error** | **Bloqueante** | Debe existir el header `timestamp`. | `headerExists("timestamp")` |
| **datetime presente** | ⛔ **Error** | **Bloqueante** | Debe existir el header `datetime`. | `headerExists("datetime")` |

---

## Reglas de Nomenclatura

| Regla | Severidad | Impacto en Validación | Descripción | Expresión (referencia) |
| :--- | :---: | :---: | :--- | :--- |
| **Estrategia lowerCamelCase** | ⛔ **Error** | **Bloqueante** | Los nombres de propiedades, parámetros query y claves de schemas deben seguir `lowerCamelCase`. | `isLowerCamelCase(name)` |
| **Pluralización de arrays** | ⚠️ **Warning** | Informativo | Propiedades `type: array` deberían nombrarse en plural (`items`, `users`). | `if type=="array" -> isPlural(name)` |
| **Sin números iniciales** | ⛔ **Error** | **Bloqueante** | Las claves de los schemas y nombres de propiedades no pueden iniciar con caracteres numéricos. | `not startsWithDigit(name)` |
| **Sufijo Code** | ⛔ **Error** | **Bloqueante** | Cualquier propiedad que represente un código clasificatorio o de negocio debe finalizar con `Code` (ej. `currencyCode`). | `isCodeField(name, description) -> endsWith("Code")` |
| **Uso de acrónimos** | ⚠️ **Warning** | Informativo | Se recomienda evitar acrónimos crípticos; los nombres deben ser descriptivos. | `advisory()` |
| **Palabras reservadas** | ⛔ **Error** | **Bloqueante** | Prohibido el uso de términos reservados en nombres: `message`, `body`, `payload`, `class`, `default`, `function`. | `notInReservedWords(name)` |

---

## Reglas de Formato
Estas reglas garantizan que el documento sea sintácticamente correcto y legible por máquinas.

| Regla | Severidad | Impacto en Validación | Descripción | Expresión (referencia) |
| :--- | :---: | :---: | :--- | :--- |
| **Documento JSON/YAML válido** | ⛔ **Error** | **Bloqueante** | El documento debe ser parseable como JSON o YAML sin errores. | `parseOk()` |
| **Evitar valores nulos o vacíos** | ⚠️ **Warning** | Informativo | Se recomienda evitar `null` o strings vacíos en metadatos, descripciones o ejemplos. | `noNullOrEmptyHeuristic()` |

---

## Reglas de Claridad y Documentación

| Regla | Severidad | Impacto en Validación | Descripción | Expresión (referencia) |
| :--- | :---: | :---: | :--- | :--- |
| **Descripción de la API** | ⚠️ **Warning** | Informativo | `info.description` debe proveer una explicación detallada sobre el propósito de negocio y el alcance técnico. | `notEmpty(info.description)` |
| **Resumen de operación** | ⚠️ **Warning** | Informativo | Cada operación debe incluir `summary`. | `exists(operation.summary)` |
| **Documentación de parámetros** | ⚠️ **Warning** | Informativo | Todos los parámetros (path, query, header) deberían contar con `description`. | `forEachParameter: hasDescription()` |
| **Documentación de schemas** | ⚠️ **Warning** | Informativo | Se recomienda que cada propiedad en `components.schemas` incluya `description`. | `forEachSchemaProperty: hasDescription()` |
| **Uso de ejemplos** | ⚠️ **Warning** | Informativo | Es recomendable incluir `example` o `examples` para facilitar pruebas y mocking. | `advisory()` |
| **Privacidad de Datos (PII)** | ⛔ **Error** | **Bloqueante** | Prohibida la exposición de datos personales/sensibles en nombres, descripciones o ejemplos. | `noPiiHeuristic()` |
| **Seguridad en errores** | ⛔ **Error** | **Bloqueante** | Respuestas de error no deben contener datos sensibles ni información interna del sistema. | `noSensitiveInErrorSchemas()` |

---

## Criterios de Aceptación y Validación

El modelo de validación se basa en un enfoque de cumplimiento estricto. Actualmente, el pipeline está configurado en modo **Informativo** para los errores.

| Estado | Resultado | Criterio Técnico | Acción en Pipeline |
| :--- | :---: | :--- | :--- |
| **✅ APROBADO** | **CUMPLE** | **0** Errores <br> **0** Advertencias | **Continuar.** El documento se procesa y despliega automáticamente. |
| **⚠️ CON OBSERVACIONES** | **CUMPLE** | **0** Errores <br> **≥1** Advertencias | **Continuar.** Se permite el despliegue, pero se notifican las observaciones al equipo de desarrollo. |
| **⛔ RECHAZADO** | **NO CUMPLE** | **≥1** Errores | **Continuar (Con Alerta).** El documento es inválido y contiene errores bloqueantes. Se publica el reporte en el PR pero **NO se detiene**. |
