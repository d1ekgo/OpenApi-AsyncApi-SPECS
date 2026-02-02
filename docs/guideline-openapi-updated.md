# Validación y reglas de diseño OpenAPI
Este documento establece el marco normativo y las reglas de validación obligatorias para garantizar la consistencia, seguridad y calidad de todas las APIs REST y Webhooks documentadas mediante OpenAPI dentro de la organización.

---

## Sistema de Reglas
El sistema de reglas organiza las reglas en 5 categorías: **Estructura**, **Headers**, **Nomenclatura**, **Formato** y **Claridad**.

**Estructura:** Cumplimiento del esquema OpenAPI definido por la organización.

**Headers:** Cumplimiento específico del esquema de Headers definido por la empresa.

**Nomenclatura:** Consistencia en nombres de paths, operaciones, parámetros y variables.

**Formato:** Documento OpenAPI válido en JSON o YAML con formato estándar.

**Claridad:** Documentación suficiente para garantizar entendimiento y correcta adopción de la API.

---

## Estructura General de las Reglas
Cada regla dentro del validador tiene la siguiente estructura:

- **Nombre:** Identificador único de la regla.
- **Severidad:** Define si la regla es un error (bloqueante) o una advertencia (warning).
- **Descripción:** Explica el propósito de la regla.
- **Mensaje:** Texto que se muestra cuando no se cumple la regla.

---

## Severidad de las Reglas
**Error (error):** Infracción crítica. Si una regla de este tipo no se cumple, la especificación se considera **INVÁLIDA**.

**Advertencia (warning):** Buena práctica recomendada. El incumplimiento **no impide** la aprobación del documento.

---

## Reglas de Estructura
| Regla | Severidad | Impacto | Descripción |
|---|:---:|:---:|---|
| **ID de la API válido** | ⛔ Error | Bloqueante | El identificador de la API debe corresponder a la URL del repositorio Git. |
| **Título descriptivo** | ⛔ Error | Bloqueante | info.title debe ser descriptivo y mayor a 10 caracteres. |
| **Versión de la API presente** | ⛔ Error | Bloqueante | info.version es obligatorio. |
| **Versión de OpenAPI permitida** | ⛔ Error | Bloqueante | La especificación debe usar OpenAPI 3.0.3 o 3.1.x, según el estándar corporativo. |
| **Paths definidos** | ⛔ Error | Bloqueante | Debe existir al menos un path. |
| **Operaciones definidas** | ⛔ Error | Bloqueante | Cada path debe definir al menos una operación HTTP válida. |
| **Responses definidos** | ⛔ Error | Bloqueante | Todas las operaciones deben definir responses. |
| **Response de éxito** | ⛔ Error | Bloqueante | Debe existir al menos una respuesta HTTP 2xx. |
| **RequestBody cuando aplica** | ⛔ Error | Bloqueante | Operaciones POST, PUT o PATCH deben definir requestBody. |
| **Schema de nivel superior tipo object** | ⛔ Error | Bloqueante | Los schemas utilizados como requestBody y responses de nivel superior deben ser de tipo object. |
| **Schemas reutilizables permitidos** | ⚠️ Warning | Informativo | Los schemas internos pueden ser de tipo array o tipos simples. |
| **Schemas definidos** | ⛔ Error | Bloqueante | Debe existir al menos un schema en components.schemas. |
| **Schemas con properties** | ⛔ Error | Bloqueante | Los schemas tipo object deben definir properties. |
| **Propiedades con tipo** | ⛔ Error | Bloqueante | Cada propiedad debe declarar su tipo. |
| **Evitar duplicación de datos** | ⚠️ Warning | Informativo | No duplicar información entre headers y body. |
| **Booleans no nulos** | ⛔ Error | Bloqueante | Campos booleanos no pueden ser null. |
| **No incluir JSON en strings** | ⛔ Error | Bloqueante | No se permite modelar estructuras JSON embebidas como strings. |
| **Alternativas a JSON dinámico** | ⚠️ Warning | Informativo | Para estructuras dinámicas se recomienda usar oneOf, anyOf o allOf. |

---

## Reglas de Headers
Los headers corporativos obligatorios deben definirse una única vez en `components.parameters` y ser referenciados mediante `$ref` en todas las operaciones.

| Header | Severidad | Impacto | Descripción |
|---|:---:|:---:|---|
| **eventId** | ⛔ Error | Bloqueante | Identificador único del evento. |
| **eventType** | ⛔ Error | Bloqueante | Tipo de evento. |
| **entityId** | ⛔ Error | Bloqueante | Identificador de la entidad. |
| **entityType** | ⛔ Error | Bloqueante | Tipo de entidad. |
| **timestamp** | ⛔ Error | Bloqueante | Marca de tiempo epoch. |
| **datetime** | ⛔ Error | Bloqueante | Fecha y hora en formato ISO-8601. |

---

## Reglas de Formato
| Regla | Severidad | Impacto | Descripción |
|---|:---:|:---:|---|
| **Documento válido** | ⛔ Error | Bloqueante | El documento debe ser JSON o YAML parseable. |
| **Formato estándar** | ⛔ Error | Bloqueante | Uso correcto de llaves, corchetes y separadores. |
| **Campos en inglés** | ⛔ Error | Bloqueante | Todas las keys deben estar definidas en inglés. |
| **Evitar valores nulos** | ⚠️ Warning | Informativo | Evitar valores null o vacíos. |

---

## Reglas de Claridad
| Regla | Severidad | Impacto | Descripción |
|---|:---:|:---:|---|
| **Descripción de la API** | ⚠️ Warning | Informativo | info.description debe explicar el propósito de la API. |
| **Resumen de operaciones** | ⚠️ Warning | Informativo | Cada operación debe definir summary. |
| **Parámetros documentados** | ⚠️ Warning | Informativo | Todos los parámetros deben tener descripción. |
| **Propiedades documentadas** | ⚠️ Warning | Informativo | Se recomienda documentar cada propiedad. |
| **Ejemplos presentes** | ⚠️ Warning | Informativo | Se recomienda incluir example o examples en schemas. |
| **Protección de datos personales** | ⛔ Error | Bloqueante | No exponer datos personales o sensibles. |
| **Errores sin PII** | ⛔ Error | Bloqueante | Los mensajes de error no deben contener datos sensibles. |

---

## Reglas de Nomenclatura
| Regla | Severidad | Impacto | Descripción |
|---|:---:|:---:|---|
| **lowerCamelCase** | ⛔ Error | Bloqueante | Convención obligatoria para propiedades. |
| **Arrays en plural** | ⚠️ Warning | Informativo | Los nombres de arrays deben estar en plural. |
| **Sin números iniciales** | ⛔ Error | Bloqueante | No iniciar nombres con números. |
| **Sufijo Code** | ⛔ Error | Bloqueante | Campos de código deben finalizar con Code. |
| **Evitar acrónimos** | ⚠️ Warning | Informativo | Usar nombres descriptivos. |
| **Palabras reservadas prohibidas** | ⛔ Error | Bloqueante | No usar message, body, payload, class, default, function. |

---

### Criterios de Aceptación y Validación

El nuevo modelo de validación se basa en un enfoque de cumplimiento estricto. Para que un archivo OpenApi sea aceptado en el ecosistema de la organización, debe cumplir con los siguientes criterios.

### Estados de Validación

El validador arrojará uno de los siguientes tres estados finales. Actualmente, el pipeline está configurado en modo **Informativo** para los errores.

| Estado | Resultado | Criterio Técnico | Acción en Pipeline |
| :--- | :---: | :--- | :--- |
| **✅ APROBADO** | **CUMPLE** | **0** Errores <br> **0** Advertencias | **Continuar.** <br> El documento se procesa y despliega automáticamente. |
| **⚠️ CON OBSERVACIONES** | **CUMPLE** | **0** Errores <br> **≥1** Advertencias | **Continuar.** <br> Se permite el despliegue, pero se notifica al equipo para futuras correcciones. |
| **⛔ RECHAZADO** | **NO CUMPLE** | **≥1** Errores | **Continuar (Con Alerta).** <br> El documento es inválido y contiene errores bloqueantes. Se publica el reporte en el PR pero **NO se detiene**. |
