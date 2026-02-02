# Validación y reglas de diseño OpenAPI
Este documento establece el marco normativo y las reglas de validación obligatorias para garantizar la consistencia, seguridad y calidad de todas las APIs REST y Webhooks documentadas mediante OpenAPI dentro de la organización.

---

## Sistema de Reglas
El sistema de reglas organiza las reglas en 5 categorías: **Estructura**, **Headers**, **Nomenclatura**, **Formato** y **Claridad**.

**Estructura:** Cumplimiento del esquema OpenAPI 3.x.

**Headers:** Cumplimiento específico del esquema de Headers definido por la empresa. Se evalúa la presencia de los headers `eventId`, `eventType`, `entityId`, `entityType`, `timestamp` y `datetime`.

**Nomenclatura:** Consistencia en nombres de paths, operaciones, parámetros y variables.

**Formato:** Documento OpenAPI válido en JSON o YAML con formato estándar.

**Claridad:** Documentación suficiente de operaciones, parámetros, requests y responses.

---

## Estructura General de las Reglas
Cada regla dentro del validador tiene la siguiente estructura:

- **Nombre:** Identificador único de la regla.
- **Severidad:** Define si la regla es un error (bloqueante) o una advertencia (warning).
- **Descripción:** Explica el propósito de la regla y la lógica para analizarla.
- **Expresión:** Referencia a la función o expresión que realiza la verificación.
- **Mensaje:** Texto que se muestra cuando no se cumple la regla.

Para aprobar la validación, el documento debe cumplir con los criterios de aceptación definidos al final de este documento.

---

## Severidad de las Reglas
**Error (error):** Infracción crítica. Si una regla de este tipo no se cumple, la especificación se considera **INVÁLIDA** inmediatamente.

**Advertencia (warning):** Recomendación o buena práctica. El incumplimiento genera una alerta, pero **NO impide** la aprobación del documento.

---

## Reglas de Estructura
| Regla | Severidad | Impacto en Validación | Descripción |
|---|:---:|:---:|---|
| **ID de la API válido** | ⛔ Error | Bloqueante | El id de la API debe ser la URL del repositorio GitHub del servicio. |
| **Título de la API presente y descriptivo** | ⛔ Error | Bloqueante | La especificación debe tener un título con más de 10 caracteres. |
| **Versión de la API presente** | ⛔ Error | Bloqueante | La especificación debe incluir info.version. |
| **Versión de OpenAPI** | ⛔ Error | Bloqueante | La especificación debe usar OpenAPI 3.x o superior. |
| **Paths definidos** | ⛔ Error | Bloqueante | Debe existir al menos un path definido. |
| **Operaciones definidas** | ⛔ Error | Bloqueante | Cada path debe tener al menos una operación. |
| **Método HTTP válido** | ⛔ Error | Bloqueante | Solo se permiten métodos HTTP estándar. |
| **Responses definidos** | ⛔ Error | Bloqueante | Cada operación debe definir responses. |
| **Response de éxito definido** | ⛔ Error | Bloqueante | Debe existir al menos una respuesta 2xx. |
| **RequestBody definido cuando aplica** | ⛔ Error | Bloqueante | Operaciones POST/PUT/PATCH deben definir requestBody. |
| **Schemas definidos** | ⛔ Error | Bloqueante | Debe existir al menos un schema en components.schemas. |
| **Tipo de schema object** | ⛔ Error | Bloqueante | Todos los schemas deben ser type: object. |
| **Propiedades del schema definidas** | ⛔ Error | Bloqueante | Cada schema debe tener properties. |
| **Propiedades del schema con tipo** | ⛔ Error | Bloqueante | Cada propiedad debe definir su tipo. |
| **Evitar duplicación de datos** | ⚠️ Warning | Informativo | No duplicar información entre headers y body. |
| **Booleans no nulos** | ⛔ Error | Bloqueante | Campos booleanos no pueden ser null. |
| **No incluir JSON en strings** | ⛔ Error | Bloqueante | No se permite JSON embebido en strings. |
| **Respetar tipos definidos** | ⛔ Error | Bloqueante | Tipos y formatos deben respetar el contrato. |
| **Campos no duplicados en schemas Body** | ⚠️ Warning | Informativo | No repetir propiedades en schemas de request/response. |

---

## Reglas de Headers
| Regla | Severidad | Impacto en Validación | Descripción |
|---|:---:|:---:|---|
| **eventId presente** | ⛔ Error | Bloqueante | Header obligatorio. |
| **eventType presente** | ⛔ Error | Bloqueante | Header obligatorio. |
| **entityId presente** | ⛔ Error | Bloqueante | Header obligatorio. |
| **entityType presente** | ⛔ Error | Bloqueante | Header obligatorio. |
| **timestamp presente** | ⛔ Error | Bloqueante | Header obligatorio. |
| **datetime presente** | ⛔ Error | Bloqueante | Header obligatorio. |

---

## Reglas de Formato
| Regla | Severidad | Impacto en Validación | Descripción |
|---|:---:|:---:|---|
| **Documento JSON/YAML válido** | ⛔ Error | Bloqueante | El documento debe ser parseable. |
| **Uso correcto de comillas** | ⛔ Error | Bloqueante | En JSON se deben usar comillas dobles. |
| **Uso correcto de llaves y corchetes** | ⛔ Error | Bloqueante | Uso correcto de {} y []. |
| **Separación correcta de claves** | ⛔ Error | Bloqueante | Uso correcto de : y ,. |
| **No incluir valores nulos o vacíos** | ⚠️ Warning | Informativo | Evitar null o strings vacíos. |
| **Campos en inglés** | ⛔ Error | Bloqueante | Todas las keys deben estar en inglés. |

---

## Reglas de Claridad
| Regla | Severidad | Impacto en Validación | Descripción |
|---|:---:|:---:|---|
| **Descripción de la API presente** | ⚠️ Warning | Informativo | info.description debe ser explicativo. |
| **Descripción de la operación presente** | ⚠️ Warning | Informativo | Cada operación debe tener summary. |
| **Descripción de parámetros** | ⚠️ Warning | Informativo | Los parámetros deben tener descripción. |
| **Propiedades con descripción** | ⚠️ Warning | Informativo | Se recomienda documentar properties. |
| **Datos personales mínimos** | ⛔ Error | Bloqueante | No exponer datos sensibles. |
| **Consistencia de tipos** | ⛔ Error | Bloqueante | Un mismo campo debe mantener tipo. |
| **Errores sin datos personales** | ⛔ Error | Bloqueante | No incluir PII en errores. |

---

## Reglas de Nomenclatura
| Regla | Severidad | Impacto en Validación | Descripción |
|---|:---:|:---:|---|
| **lowerCamelCase** | ⛔ Error | Bloqueante | Convención obligatoria. |
| **Nombres de arrays en plural** | ⚠️ Warning | Informativo | Arrays deben terminar en s. |
| **Claves sin números iniciales** | ⛔ Error | Bloqueante | No iniciar claves con números. |
| **Sufijo Code** | ⛔ Error | Bloqueante | Campos de código deben terminar en Code. |
| **Evitar acrónimos** | ⚠️ Warning | Informativo | Usar nombres descriptivos. |
| **No usar palabras reservadas** | ⛔ Error | Bloqueante | Prohibido usar message, body, payload, class, default, function. |

---

## Criterios de Aceptación y Validación
| Estado | Resultado | Criterio Técnico | Acción en Pipeline |
|---|:---:|---|---|
| **APROBADO** | CUMPLE | 0 errores / 0 warnings | Continuar despliegue. |
| **CON OBSERVACIONES** | CUMPLE | 0 errores / ≥1 warnings | Continuar con alertas. |
| **RECHAZADO** | NO CUMPLE | ≥1 errores | Continuar con alerta crítica. |
