[guidelines-asyncapi.md](https://github.com/user-attachments/files/25901919/guidelines-asyncapi.md)
# Lineamientos y Estándares de AsyncAPI (Corporativo)

**Estado:** Vigente
**Versión del documento:** 1.6
**Fecha:** 2026-03-09
**Aplicabilidad:** Productores y consumidores de eventos (Arquitecturas Orientadas a Eventos)

---

## Historial de cambios

| Versión | Cambios |

---

## 1. Introducción y Alcance

La organización estandariza AsyncAPI para asegurar **consistencia**, **seguridad**, **trazabilidad** y **calidad** en contratos de eventos, habilitando automatización (validación en CI/CD, generación de documentación y gobierno de cambios).

Este estándar aplica a:

- APIs asincrónicas publicadas en brokers (por defecto **AWS SNS/SQS**) y/o protocolos equivalentes.
- Contratos de eventos de tipo **Created / Updated / Deleted** y otros eventos de negocio.
- Equipos que publican/consumen eventos y deben cumplir con validaciones corporativas (estructura, headers, nomenclatura, formato y claridad).

---

## 2. Especificaciones del Documento AsyncAPI

### 2.1 Versión de AsyncAPI

- El contrato **DEBE** usar **AsyncAPI 3.0.0** como versión base corporativa.
- El contrato **PUEDE** usar **3.1.x** cuando la toolchain corporativa lo soporte y el validador esté actualizado.

### 2.2 Metadatos obligatorios

El contrato **DEBE** incluir los siguientes campos:

- `id`: URL absoluta del repositorio Git del producer (fuente de verdad del contrato).
- `info`:
  - `info.title` **DEBE** ser descriptivo y tener **> 10** caracteres.
  - `info.version` **DEBE** existir en formato SemVer recomendado.
  - `info.description` **DEBERÍA** existir explicando propósito, alcance y evento(s).
  - `info.contact` **DEBERÍA** existir (name + email).
  - `info.license` **DEBERÍA** existir (name + url).
  - `info.tags` **DEBERÍA** existir para categorizar el dominio/capability.
- `defaultContentType`: **DEBE** ser `application/json`, salvo excepción aprobada.

### 2.3 Servers

- `servers` **DEBE** existir y contener al menos un servidor.
- Cada `servers.<n>.protocol` **DEBE** ser uno de los protocolos permitidos (ver §5.1).

---

## 3. Convenciones de Diseño (Design Guidelines)

### 3.1 Nomenclatura de Canales (Channels)

#### 3.1.1 Regla general

- El nombre del canal **DEBE** representar el **recurso de infraestructura** (tópico/cola) y seguir un patrón consistente.
- Para AWS, los canales **DEBEN** distinguir explícitamente **topic** (SNS) y **queue** (SQS).

#### 3.1.2 Patrones corporativos

**SNS Topic (publicación):**
```
topic-<subdomain>-<businessCapability>-<entity>-<event>
```

**SQS Queue (consumo):**
```
queue-<subdomain>-<businessCapability>-<entity>-<event>-<consumer>
```

- `<subdomain>` y `<businessCapability>` **DEBEN** ser acrónimos en **lowercase**.
- `<entity>` **DEBE** ser plural (`pudos`, `orders`, `shipments`).
- `<event>` **DEBE** estar en pasado (`created`, `updated`, `deleted`) o representar el hecho de negocio.

#### 3.1.3 `address` del canal

- `channels.<n>.address` **DEBE** ser el nombre real del recurso o su expresión estándar corporativa.
- `address` **DEBE** coincidir con el recurso real, sin dobles guiones ni typos.
- La key del canal (`channels.<key>`) y su `address` **DEBEN** ser idénticos (mismo string) para evitar drift entre el identificador del canal y el recurso real.

---

### 3.2 Operaciones

#### 3.2.1 Perspectiva correcta (send/receive)

En AsyncAPI v3, la operación se describe desde la perspectiva de la **aplicación**:

- `action: send`: la aplicación **envía/publica** un mensaje hacia el broker.
- `action: receive`: la aplicación **recibe/consume** un mensaje desde el broker.

#### 3.2.2 Reglas mínimas por operación

Cada operación:

- **DEBE** tener `summary` y **DEBERÍA** tener `description`.
- **DEBE** referenciar `channel` mediante `$ref`.
- **DEBE** declarar `messages`.
- Cada elemento en `operations.*.messages` **DEBE** ser `$ref` a un mensaje **definido en el canal** (`#/channels/.../messages/...`), no a `components.messages` directamente. Los mensajes del canal sí pueden referenciar `components.messages`.

**Convención de nombres de operaciones:**

- Productor: `send<Entity><Event>` (ej. `sendPudoCreated`)
- Consumidor: `receive<Entity><Event>` (ej. `receivePudoCreated`)

---

### 3.3 Gobierno EDA: separación de contratos y versionamiento

#### 3.3.1 Separación por tipo de evento (eventType)

- Regla general: **una especificación AsyncAPI por tipo de evento**.
- Excepción permitida: eventos **CRUD** (Created/Updated/Deleted) que operan sobre una misma entidad/recurso y comparten el mismo body (o un body altamente similar y gobernado de manera unificada).

Esta separación facilita ownership, versionamiento, gobierno y trazabilidad: un contrato representa un propósito claro.

#### 3.3.2 Producers (productores) por tipo de evento

- Regla general: **un producer por tipo de evento** (un repositorio/servicio dueño del contrato).
- Excepción CRUD: si se define un contrato unificado para CRUD sobre la misma entidad y body compartido, se permite un producer que publique esas variantes, siempre que el gobierno del schema sea consistente.

#### 3.3.3 Máximo 2 versiones activas por evento

- Por cada `eventType` se permite mantener **máximo 2 versiones activas** (ej. `1.0` y `2.0`).
- Si se requiere una tercera versión, se **DEBE** planificar y ejecutar la **deprecación** de la versión más antigua para volver a 2 versiones activas.

Una versión se considera "activa" cuando aún está publicada o consumida en producción o en transición controlada (migración).

#### 3.3.4 Consideraciones especiales para CRUD (Updated)

- En eventos **Updated**, se debe definir explícitamente si la operación envía el **payload completo (full payload)** o **solo campos actualizados (partial update)**.
- Si el evento Updated es partial update, los campos que no siempre viajan **DEBEN** ser **opcionales** en el schema (no incluirlos en `required`), y se debe documentar explícitamente la semántica.

---

### 3.4 Políticas de filtrado SNS/SQS (Filter Policy)

Las políticas de filtrado para suscripciones SNS/SQS son objetos JSON que definen qué mensajes recibe un suscriptor. Cuando se publica un mensaje, SNS evalúa atributos y/o cuerpo contra la política y **solo entrega** al suscriptor si se cumplen las condiciones. El filtrado no tiene costo asociado.

#### 3.4.1 Regla corporativa: eventType + version (obligatorio)

Si se aplica un filtro por `eventType`, es **obligatorio** incluir también `version` en el filtro.

**Filtro simple (1 eventType + 1 versión):**
```json
{
  "eventType": ["orderModified"],
  "version": ["1.0"]
}
```

**1 eventType + múltiples versiones:**
```json
{
  "eventType": ["orderModified"],
  "version": ["1.0", "2.0"]
}
```

**Filtro compuesto ($or) con múltiples eventTypes y versiones específicas:**
```json
{
  "$or": [
    { "eventType": ["rateCalculated"], "version": ["1.0"] },
    { "eventType": ["paymentReceived"], "version": ["1.0"] },
    { "eventType": ["orderLabelsChangeRequested"], "version": ["1.0"] }
  ]
}
```

#### 3.4.2 Documentación en el contrato

Para asegurar la trazabilidad de la política, se recomienda declararla en el contrato mediante una extensión corporativa en el canal de consumo:

- `channels.<queue>.x-sns-filterPolicy` (JSON), o
- `channels.<queue>.bindings.<protocol>.x-sns-filterPolicy` si el binding lo soporta.

El validador corporativo puede usar esta extensión para verificar la presencia de `eventType` y `version`.

---

## 4. Estructura de Mensajes y Payloads

### 4.1 Esquemas

- El payload **DEBE** definirse mediante **JSON Schema** en `components.schemas`.
- Cada schema usado como payload **DEBE** ser `type: object` con `properties` tipadas por propiedad.
- Si una propiedad es `array`, **DEBE** definir `items`.
- Está prohibido embeber JSON en `string`; si se requiere estructura, usar `object`.

Aunque la mensajería SNS/SQS pueda transportar el payload como string, el contrato **modela el objeto JSON** que la aplicación serializa y deserializa.

#### 4.1.1 Normativa corporativa del body (payload)

El body del evento debe cumplir las siguientes reglas:

- **Minimalismo:** no transmitir datos personales; enfocarse en información estrictamente necesaria (preferiblemente IDs) y disponibilizar APIs para consultar detalles cuando aplique.
- **No JSON embebido en string:** si se necesita estructura, definir `type: object` con propiedades.
- **Respeto de tipos:** los valores deben respetar tipos y formatos definidos en el contrato (ej. `timestamp` epoch ms, `datetime` RFC3339).
- **Arrays:** todo `type: array` debe definir `items`.
- **Sin duplicidad:** no repetir la misma data con nombres distintos, ni repetir data entre headers y payload (ej. `timestamp`).
- **Campos útiles:** evitar campos en desuso, vacíos o nulos.
- **Sufijo Code:** si un campo representa un código, el nombre **debería** terminar en `Code` (ej. `regionCode`).
- **Palabras reservadas:** evitar palabras reservadas de JavaScript como nombres de propiedades. Están prohibidas: `message`, `body`, `payload`, `class`, `default`, `function`, `null`, `true`, `false`, `return`, `var`, `let`, `const`, `new`, `this`, `typeof`, `delete`, `void`, `instanceof`, `in`, `if`, `else`, `for`, `while`, `do`, `break`, `continue`, `switch`, `case`, `throw`, `try`, `catch`, `finally`, `import`, `export`, `extends`, `super`, `yield`, `async`, `await`, `static`, `with`, `debugger`, `eval`, `arguments`, `prototype`, `constructor`.
- **Caracteres Unicode no imprimibles:** los nombres de propiedades solo deben contener caracteres ASCII imprimibles (U+0020–U+007E). Están prohibidos los caracteres de control (U+0000–U+001F, U+007F) que son invisibles y rompen SDKs de generación de código.
- **Consistencia de `example` con `type`:** el valor del campo `example` debe ser compatible con el `type` declarado en la misma propiedad. Por ejemplo, `type: integer` con `example: "5"` es inválido (el valor debe ser numérico, no string). Lo mismo aplica para `boolean`, `array` y `object`.
- **Booleanos no nulos:** las propiedades `type: boolean` **NO DEBEN** ser `nullable: true` ni tener `default: null`.
- **Sin duplicación de datos:** evitar propiedades en el mismo schema que contengan la misma información semántica bajo nombres distintos.
- **Identificadores de entidades:** el payload **DEBERÍA** incluir al menos un campo identificador de la entidad principal del evento (propiedad terminada en `Id` o llamada `id`).

---

### 4.2 Cabeceras corporativas obligatorias (Message Traits)

Los mensajes **DEBEN** referenciar vía `traits` los cuatro grupos corporativos. El orden de los `$ref` no es relevante, pero **deben estar todos presentes**:

- `commonHeaders`
- `transportHeaders`
- `businessHeaders`
- `trackingHeaders`

Los traits se declaran en `components.messageTraits.*` y se referencian con `$ref`.

#### 4.2.1 `commonHeaders` (obligatorios)

| Campo | Tipo | Descripción |
|---|---|---|
| `eventId` | string | UUID o Hex32 |
| `eventType` | string | lowerCamelCase en pasado (recurso + verbo pasado) |
| `entityId` | string | Alfanumérico |
| `entityType` | string | Literal plural recomendado |
| `timestamp` | integer (int64) | Epoch en milisegundos UTC — representa el tiempo del **evento de negocio** |
| `datetime` | string (date-time) | ISO 8601 / RFC 3339 (UTC) |
| `version` | string | `MAJOR.MINOR` |
| `internal` | boolean | `true` = solo integraciones internas; `false` = puede ser externo |

#### 4.2.2 `transportHeaders` (obligatorios)

| Campo | Tipo | Descripción |
|---|---|---|
| `channel` | string (enum) | Origen del evento: `web`, `mobile`, `api`, `batch_process`, `internal_system`, `external_partner` |

#### 4.2.3 `businessHeaders` (obligatorios)

| Campo | Tipo | Descripción |
|---|---|---|
| `domain` | string | Acrónimo **lowercase** |
| `subdomain` | string | Acrónimo **lowercase** |
| `businessCapability` | string | Acrónimo **lowercase** |

#### 4.2.4 `trackingHeaders` (obligatorios)

| Campo | Tipo | Descripción |
|---|---|---|
| `traceId` | string | Hex32 en minúscula |
| `spanId` | string | Hex16 en minúscula |
| `parentSpanId` | string | Opcional. Si existe, Hex16 en minúscula |

#### 4.2.5 Semántica de trazabilidad (traceId/spanId/parentSpanId/eventId)

Para correlacionar flujos EDA donde un evento origina otros (SNS → SQS → SNS), se aplica la siguiente semántica:

- `traceId` es **global**: se mantiene a lo largo de todos los tramos del flujo con un origen común.
- `spanId` identifica el tramo actual: en AWS se modela como Hex16 asociado al ARN/identificador del recurso de mensajería (tópico o cola) del tramo actual.
- `parentSpanId` referencia el tramo padre: en el primer tópico (raíz) no existe; en un evento derivado, corresponde al `spanId` del tramo anterior.
- `eventId` identifica el evento específico (UUID o Hex32). Si el evento no tiene un ID propio, puede coincidir con `traceId`.

**Ejemplo (conceptual):**

| Tramo | `traceId` | `spanId` | `parentSpanId` | `eventId` |
|---|---|---|---|---|
| Primer tópico (`topic-sns-1`) | Hex32(id-global) | Hex16(ARN-topic-sns-1) | null / ausente | Hex32(evento-original) |
| Segundo tópico (`topic-sns-2`) | Hex32(id-global) | Hex16(ARN-topic-sns-2) | Hex16(ARN-topic-sns-1) | Hex32(evento-nuevo) |

Los headers deben insertarse en SNS como **atributos del mensaje** (Message Attributes), no exclusivamente en el body.

---

### 4.3 Propiedades DME obligatorias

Cuando aplique integración con DME, el contrato **DEBE** incluir:

- `x-dme-path-reference` en `components.schemas.<Schema>` y en `components.messages.<Message>`, con el **mismo valor** en ambos.
- `x-dme-fullpayload` en `components.messages` (**boolean**):
  - `true`: Event Sourcing (payload completo).
  - `false`: EDA minimalista (solo IDs; detalle disponible vía API externa).
- `x-dme-operation` en `components.messages`: **DEBE** ser `C` | `U` | `D`.
- `x-dme-version` en `components.schemas`: versión del schema requerida por el proceso DME.
- `x-dme-securityPolicyTags` en propiedades sensibles: **DEBE** ser **array de strings** (referencias a policy tags).

**Policy Tags corporativos:**

| Clasificación | Referencia |
|---|---|
| `personal_information` | `projects/tecnogia-arqdat-governance-prd/locations/us/taxonomies/8649944501028248774/policyTags/8984512984767038865` |
| `sensitive_data` | `projects/tecnogia-arqdat-governance-prd/locations/us/taxonomies/8649944501028248774/policyTags/5134120500434546380` |

---

### 4.4 Reglas de minimización y duplicidad

- Los eventos **DEBEN** ser minimalistas: enviar solo lo necesario para que el consumidor procese.
- No repetir data entre **headers** y **payload** (ej. `timestamp`, `entityId`).
- Evitar duplicidad de campos que contengan la misma información bajo nombres distintos.
- Si un evento incluye PII por necesidad, **DEBE** aplicar policy tags DME.

---

## 5. Seguridad y Protocolos

### 5.1 Protocolos permitidos

El contrato **DEBE** declarar `servers.<n>.protocol` usando uno de los protocolos aprobados:

| Categoría | Protocolos |
|---|---|
| **Aprobados por defecto (plataforma actual)** | `sns`, `sqs` |
| **Permitidos bajo aprobación de Arquitectura** | `kafka`, `amqp`, `mqtt`, `ws`/`websocket`, `http` (streaming/webhooks) |

### 5.2 `securitySchemes` y `servers.security`

- En ambientes productivos, los servidores **DEBERÍAN** documentar el mecanismo de seguridad (IAM, credenciales, mTLS, etc.).
- Si se modela seguridad en el contrato, `components.securitySchemes` y `servers.<n>.security` **DEBEN** estar alineados.

Para AWS SNS/SQS, el acceso por IAM suele gestionarse fuera del contrato; en ese caso, se debe documentar el mecanismo en `servers.<n>.description`.

---

## 6. Sistema de Reglas (Validador) y Criterios de Aceptación

### 6.1 Categorías de reglas

El validador organiza las validaciones en cinco categorías:

- **Estructura:** cumplimiento del esquema AsyncAPI.
- **Headers:** cumplimiento del esquema de headers corporativos (traits + formato mínimo).
- **Nomenclatura:** consistencia en nombres de canales, operaciones y variables.
- **Formato:** documento YAML/JSON válido y estilo base.
- **Claridad:** documentación suficiente y ausencia de anti-patrones.

### 6.2 Severidad

| Nivel | Descripción |
|---|---|
| **⛔ Error** | Bloqueante. Si existe al menos un error, el contrato **NO** aprueba. |
| **⚠️ Warning** | No bloqueante. Se registra como deuda técnica. |

### 6.3 Determinístico vs Heurístico

- **Spectral (determinístico):** valida estructura, presencia, unicidad, patrones y tipos. Incluye desde v1.6 detección de tildes/ñ en llaves y validación de booleanos no nulos.
- **Copilot (heurístico):** evalúa claridad, coherencia semántica y consistencia conceptual sin funciones custom. Catálogo de 18 reglas desde v1.6, incluyendo duplicación de datos, identificadores de entidades, evaluación semántica de llaves en inglés y consistencia de `example` con `type`.

### 6.4 Determinístico vs Heurístico: criterios de aprobación

El aprobador corporativo **DEBE** basar el rechazo en reglas determinísticas para evitar falsos negativos/positivos:

- **Reglas determinísticas (hard gate):** reglas con severidad **⛔ Error** del Anexo A (parseo, estructura, headers, patterns).
- **Reglas heurísticas (IA):** reglas de **Claridad** (calidad de descripciones, minimalismo, nombres semánticos) como **⚠️ Warning** o recomendaciones.

---

## 6.5 Anexo A — Reglas del validador

Este anexo es **normativo**. Cada regla se evalúa en CI/CD mediante el validador corporativo. Los ejemplos son mínimos e ilustran el patrón de cumplimiento/incumplimiento.

---

### Estructura

Evalúa la organización y coherencia general del documento AsyncAPI.

| Regla | Severidad | Descripción | Ejemplo de código |
|---|---:|---|---|
| ID de la API válido | ⛔ Error | El `id` **DEBE** ser la URL absoluta del repositorio Git del producer. | <pre><code class="language-yaml"># ❌&#10;id: "pudos-producer"&#10;# ✅&#10;id: "https://github.com/acme-org/pudos-producer"</code></pre> |
| Versión de AsyncAPI | ⛔ Error | El contrato **DEBE** usar AsyncAPI **3.0.0** (baseline corporativa) o **3.1.x** (cuando esté habilitado). | <pre><code class="language-yaml"># ❌&#10;asyncapi: 2.6.0&#10;# ✅&#10;asyncapi: 3.0.0</code></pre> |
| Título de la API presente y descriptivo | ⛔ Error | `info.title` **DEBE** existir y tener más de 10 caracteres. | <pre><code class="language-yaml"># ❌&#10;info:&#10;  title: "PUDOs"&#10;# ✅&#10;info:&#10;  title: "PUDOs Eventos de puntos pickup"</code></pre> |
| Versión de la API presente | ⛔ Error | `info.version` **DEBE** existir (SemVer recomendado). | <pre><code class="language-yaml"># ❌&#10;info:&#10;  title: "PUDOs Eventos"&#10;# ✅&#10;info:&#10;  title: "PUDOs Eventos"&#10;  version: "1.0.0"</code></pre> |
| Versión de la API en SemVer | ⛔ Error | `info.version` **DEBE** cumplir formato SemVer `MAJOR.MINOR.PATCH`. | <pre><code class="language-yaml"># ❌&#10;info:&#10;  version: "1"&#10;# ✅&#10;info:&#10;  version: "1.0.0"</code></pre> |
| Contacto presente | ⚠️ Warning | `info.contact` **DEBERÍA** existir (name y email). | <pre><code class="language-yaml"># ✅&#10;info:&#10;  contact:&#10;    name: "Equipo Arquitectura y APIs"&#10;    email: "arquitectura@acme-org.example"</code></pre> |
| Licencia presente | ⚠️ Warning | `info.license` **DEBERÍA** existir (name y url). | <pre><code class="language-yaml"># ✅&#10;info:&#10;  license:&#10;    name: "Apache 2.0"&#10;    url: "https://www.apache.org/licenses/LICENSE-2.0"</code></pre> |
| defaultContentType definido | ⛔ Error | `defaultContentType` **DEBE** ser `application/json`, salvo excepción aprobada. | <pre><code class="language-yaml"># ❌&#10;defaultContentType: "text/plain"&#10;# ✅&#10;defaultContentType: "application/json"</code></pre> |
| Servers definidos | ⛔ Error | La especificación **DEBE** declarar al menos un `server`. | <pre><code class="language-yaml"># ✅&#10;servers:&#10;  production-sns:&#10;    host: "arn:aws:sns:us-east-1:000000000000"&#10;    protocol: sns&#10;    description: "SNS Prod"</code></pre> |
| Canales definidos | ⛔ Error | La especificación **DEBE** declarar al menos un canal en `channels`. | <pre><code class="language-yaml"># ✅&#10;channels:&#10;  topic-pdoohd-pudom-pudos-created:&#10;    address: "topic-pdoohd-pudom-pudos-created"&#10;    messages: {}</code></pre> |
| `address` de canal presente | ⛔ Error | Cada canal **DEBE** definir `address` correspondiente al nombre real del recurso. | <pre><code class="language-yaml"># ❌&#10;channels:&#10;  topic-x:&#10;    messages: {}&#10;# ✅&#10;channels:&#10;  topic-x:&#10;    address: "topic-x"&#10;    messages: {}</code></pre> |
| Mensajes definidos en cada canal | ⛔ Error | Cada canal **DEBE** tener al menos un mensaje en `channels.*.messages`. | <pre><code class="language-yaml"># ✅&#10;channels:&#10;  topic-x:&#10;    address: "topic-x"&#10;    messages:&#10;      entityCreated.message:&#10;        $ref: "#/components/messages/EntityCreatedMessage"</code></pre> |
| Operaciones definidas | ⛔ Error | Debe existir al menos una operación en `operations`. | <pre><code class="language-yaml"># ✅&#10;operations:&#10;  sendEntityCreated:&#10;    action: send&#10;    channel:&#10;      $ref: "#/channels/topic-x"&#10;    messages:&#10;      - $ref: "#/channels/topic-x/messages/entityCreated.message"</code></pre> |
| Acción de la operación válida (send/receive) | ⛔ Error | `operations.*.action` **DEBE** ser `send` o `receive`. | <pre><code class="language-yaml"># ❌&#10;action: publish&#10;# ✅&#10;action: send</code></pre> |
| Canal referenciado por operación | ⛔ Error | `operations.*.channel` **DEBE** ser un `$ref` a `#/channels/...`. | <pre><code class="language-yaml"># ❌&#10;channel: topic-x&#10;# ✅&#10;channel:&#10;  $ref: "#/channels/topic-x"</code></pre> |
| Mensajes declarados por operación | ⛔ Error | `operations.*.messages` **DEBE** existir y no estar vacío. | <pre><code class="language-yaml"># ✅&#10;messages:&#10;  - $ref: "#/channels/topic-x/messages/entityCreated.message"</code></pre> |
| Mensajes de operación referencian mensajes del canal | ⛔ Error | Cada `$ref` en `operations.*.messages` **DEBE** apuntar a `#/channels/<canal>/messages/<messageKey>`, no directamente a `components.messages`. | <pre><code class="language-yaml"># ❌&#10;messages:&#10;  - $ref: "#/components/messages/EntityCreatedMessage"&#10;# ✅&#10;messages:&#10;  - $ref: "#/channels/topic-x/messages/entityCreated.message"</code></pre> |
| Payload del mensaje definido | ⛔ Error | Cada `components.messages.*` **DEBE** incluir `payload` (idealmente `$ref` a schema). | <pre><code class="language-yaml"># ✅&#10;EntityCreatedMessage:&#10;  name: "entityCreated"&#10;  payload:&#10;    $ref: "#/components/schemas/EntityCreatedBody"</code></pre> |
| Schemas definidos | ⛔ Error | La especificación **DEBE** definir al menos un schema en `components.schemas`. | <pre><code class="language-yaml"># ✅&#10;components:&#10;  schemas:&#10;    EntityCreatedBody:&#10;      type: object&#10;      properties:&#10;        entityId:&#10;          type: string</code></pre> |
| Schema object con properties | ⛔ Error | Cada schema usado como payload **DEBE** ser `type: object` y definir `properties`. | <pre><code class="language-yaml"># ❌&#10;EntityCreatedBody:&#10;  type: string&#10;# ✅&#10;EntityCreatedBody:&#10;  type: object&#10;  properties:&#10;    entityId:&#10;      type: string</code></pre> |
| Arrays con items | ⛔ Error | Toda propiedad `type: array` **DEBE** definir `items`. | <pre><code class="language-yaml"># ❌&#10;items:&#10;  type: array&#10;# ✅&#10;items:&#10;  type: array&#10;  items:&#10;    type: string</code></pre> |
| protocolVersion de server presente | ⚠️ Warning | Cada `servers.*.protocolVersion` **DEBERÍA** existir para declarar la versión del protocolo/binding. | <pre><code class="language-yaml"># ✅&#10;servers:&#10;  production-sns:&#10;    protocol: sns&#10;    protocolVersion: "1.0.0"</code></pre> |
| Host ARN válido para SNS/SQS | ⚠️ Warning | Si `protocol` es `sns` o `sqs`, `servers.*.host` **DEBERÍA** ser un ARN base válido. | <pre><code class="language-yaml"># ❌&#10;host: "sns-prod"&#10;# ✅&#10;host: "arn:aws:sns:us-east-1:772932014686"</code></pre> |

---

### Headers

| Regla | Severidad | Descripción | Ejemplo de código |
|---|---:|---|---|
| Message Traits corporativos presentes | ⛔ Error | Cada mensaje **DEBE** referenciar los 4 traits corporativos: `commonHeaders`, `transportHeaders`, `businessHeaders`, `trackingHeaders`. | <pre><code class="language-yaml"># ✅&#10;traits:&#10;  - $ref: "#/components/messageTraits/commonHeaders"&#10;  - $ref: "#/components/messageTraits/transportHeaders"&#10;  - $ref: "#/components/messageTraits/businessHeaders"&#10;  - $ref: "#/components/messageTraits/trackingHeaders"</code></pre> |
| commonHeaders: campos mínimos | ⛔ Error | `commonHeaders` **DEBE** definir: `eventId`, `eventType`, `entityId`, `entityType`, `timestamp`, `datetime`, `version`, `internal`. | <pre><code class="language-yaml"># ✅&#10;commonHeaders:&#10;  headers:&#10;    type: object&#10;    required: [eventId,eventType,entityId,entityType,timestamp,datetime,version,internal]</code></pre> |
| timestamp: epoch ms (int64) | ⛔ Error | `timestamp` **DEBE** ser `integer` (int64) en milisegundos UTC (13+ dígitos). | <pre><code class="language-yaml"># ❌&#10;timestamp: { type: string }&#10;# ✅&#10;timestamp: { type: integer, format: int64, example: 1731008444499 }</code></pre> |
| datetime: RFC3339 (date-time) | ⛔ Error | `datetime` **DEBE** tener `format: date-time` (RFC3339 UTC). | <pre><code class="language-yaml"># ❌&#10;datetime: { type: string }&#10;# ✅&#10;datetime: { type: string, format: date-time, example: "2024-11-07T19:40:44.499Z" }</code></pre> |
| version: MAJOR.MINOR | ⛔ Error | `version` **DEBE** cumplir `MAJOR.MINOR` (ej. `1.0`). | <pre><code class="language-yaml"># ❌&#10;version: { type: string, example: "v1" }&#10;# ✅&#10;version: { type: string, pattern: "^\\d+\\.\\d+$", example: "1.0" }</code></pre> |
| internal: boolean no nulo | ⛔ Error | `internal` **DEBE** ser boolean y **NO PUEDE** ser `null`. | <pre><code class="language-yaml"># ❌&#10;internal: null&#10;# ✅&#10;internal: true</code></pre> |
| transportHeaders: channel presente | ⛔ Error | `transportHeaders` **DEBE** incluir `channel` con valores permitidos definidos por enum. | <pre><code class="language-yaml"># ✅&#10;channel:&#10;  type: string&#10;  enum: [web,mobile,api,batch_process,internal_system,external_partner]</code></pre> |
| businessHeaders: dominio/subdominio/capability | ⛔ Error | `businessHeaders` **DEBE** incluir `domain`, `subdomain`, `businessCapability` en lowercase. | <pre><code class="language-yaml"># ❌&#10;domain: { type: string, example: "PUDO" }&#10;# ✅&#10;domain: { type: string, pattern: "^[a-z0-9]+$", example: "pdoohd" }</code></pre> |
| trackingHeaders: traceId hex32 | ⛔ Error | `traceId` **DEBE** ser hex32 en minúscula. | <pre><code class="language-yaml"># ❌&#10;traceId: { type: string, example: "ABC" }&#10;# ✅&#10;traceId: { type: string, pattern: "^[0-9a-f]{32}$" }</code></pre> |
| trackingHeaders: spanId hex16 | ⛔ Error | `spanId` **DEBE** ser hex16 en minúscula. | <pre><code class="language-yaml"># ❌&#10;spanId: { type: string, example: "123" }&#10;# ✅&#10;spanId: { type: string, pattern: "^[0-9a-f]{16}$" }</code></pre> |
| trackingHeaders: parentSpanId opcional pero válido | ⚠️ Warning | Si `parentSpanId` existe, **DEBE** ser hex16 en minúscula. | <pre><code class="language-yaml"># ❌&#10;parentSpanId: { type: string, example: "XYZ" }&#10;# ✅&#10;parentSpanId: { type: string, pattern: "^[0-9a-f]{16}$" }</code></pre> |
| eventId: UUID o hex32 | ⛔ Error | `eventId` **DEBE** ser UUID (RFC4122) **o** Hex32. | <pre><code class="language-yaml"># ✅ (UUID)&#10;eventId: { type: string, pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-..." }&#10;# ✅ (Hex32)&#10;eventId: { type: string, pattern: "^[0-9a-f]{32}$" }</code></pre> |
| eventType: lowerCamelCase | ⛔ Error | `eventType` **DEBE** ser `lowerCamelCase`, sin espacios, guiones ni caracteres especiales. | <pre><code class="language-yaml"># ❌&#10;eventType: "pudo_created"&#10;# ✅&#10;eventType: "pudoCreated"</code></pre> |
| transportHeaders: channel requerido | ⛔ Error | `transportHeaders.headers.required` **DEBE** contener `channel`. Complementa la validación de presencia y enum del campo. | <pre><code class="language-yaml"># ❌&#10;transportHeaders:\n  headers:\n    properties:\n      channel: {...}  # sin required\n# ✅\n  headers:\n    required: [channel]\n    properties:\n      channel: {...}</code></pre> |
| trackingHeaders: traceId/spanId requeridos | ⛔ Error | `trackingHeaders.headers.required` **DEBE** contener tanto `traceId` como `spanId`. Complementa la validación de formato hex32/hex16. | <pre><code class="language-yaml"># ❌&#10;trackingHeaders:\n  headers:\n    properties:\n      traceId: {...}  # sin required\n# ✅\n  headers:\n    required: [traceId, spanId]\n    properties:\n      traceId: {...}</code></pre> |
| Message Traits corporativos aplicados en cada mensaje | ⛔ Error | Cada `components.messages.*` **DEBE** declarar `traits` con al menos 4 `$ref` corporativos. Complementa la validación de existencia de los traits en `components.messageTraits`. | <pre><code class="language-yaml"># ❌&#10;PudoCreatedMessage:\n  name: pudoCreated\n  payload: {...}  # sin traits\n# ✅\n  traits:\n    - $ref: '#/components/messageTraits/commonHeaders'\n    - $ref: '#/components/messageTraits/transportHeaders'\n    - $ref: '#/components/messageTraits/businessHeaders'\n    - $ref: '#/components/messageTraits/trackingHeaders'</code></pre> |

---

### Nomenclatura

| Regla | Severidad | Descripción | Ejemplo de código |
|---|---:|---|---|
| Canales siguen patrón corporativo | ⛔ Error | Los nombres de canales **DEBEN** seguir: `topic-<subdomain>-<businessCapability>-<entity>-<event>` (SNS) y `queue-<subdomain>-<businessCapability>-<entity>-<event>-<consumer>` (SQS). | <pre><code class="language-yaml"># ❌&#10;channels:&#10;  pudosCreated:&#10;    address: "pudosCreated"&#10;# ✅&#10;channels:&#10;  topic-pdoohd-pudom-pudos-created:&#10;    address: "topic-pdoohd-pudom-pudos-created"</code></pre> |
| `address` coincide con el nombre real | ⛔ Error | `channels.*.address` **DEBE** ser idéntico a la key del canal. La verificación de igualdad exacta (key vs valor) requiere evaluación semántica (**Copilot**); Spectral solo valida que `address` exista y siga el patrón `topic-`/`queue-`. | <pre><code class="language-yaml"># ❌&#10;address: "queue--pdoohd-pudom-pudos-created"&#10;# ✅&#10;address: "queue-pdoohd-pudom-pudos-created-pudos"</code></pre> |
| Operaciones con prefijo send/receive | ⛔ Error | Operaciones **DEBEN** nombrarse `send<Entity><Event>` o `receive<Entity><Event>` (UpperCamelCase después del prefijo). | <pre><code class="language-yaml"># ❌&#10;publishPudoCreated: {}&#10;# ✅&#10;sendPudoCreated: {}&#10;receivePudoCreated: {}</code></pre> |
| Message name en lowerCamelCase | ⛔ Error | `components.messages.*.name` **DEBE** ser `lowerCamelCase` y representar el hecho en pasado (ej. `pudoCreated`). | <pre><code class="language-yaml"># ❌&#10;name: "Pudo_Created"&#10;# ✅&#10;name: "pudoCreated"</code></pre> |
| Schemas en PascalCase | ⚠️ Warning | Los nombres de `components.schemas` **DEBERÍAN** ser PascalCase (ej. `PudoCreatedBody`). | <pre><code class="language-yaml"># ❌&#10;schemas:&#10;  pudoCreatedBody: {}&#10;# ✅&#10;schemas:&#10;  PudoCreatedBody: {}</code></pre> |
| Propiedades lowerCamelCase | ⛔ Error | Las propiedades de schemas **DEBEN** estar en `lowerCamelCase`. Está prohibido `snake_case`. | <pre><code class="language-yaml"># ❌&#10;pickup_point_id: { type: string }&#10;# ✅&#10;pickupPointId: { type: string }</code></pre> |
| Nombres de arrays en plural | ⚠️ Warning | Propiedades con `type: array` **DEBERÍAN** estar en plural (ej. `items`, `orders`). | <pre><code class="language-yaml"># ❌&#10;item:&#10;  type: array&#10;# ✅&#10;items:&#10;  type: array</code></pre> |
| Claves JSON sin números al inicio | ⛔ Error | Ninguna clave de schema **PUEDE** comenzar con un número. | <pre><code class="language-yaml"># ❌&#10;1stField: { type: string }&#10;# ✅&#10;firstField: { type: string }</code></pre> |
| Sufijo Code para campos de códigos | ⚠️ Warning | Si el `description` indica que un campo es un código, el nombre **DEBERÍA** terminar en `Code` (ej. `statusCode`). Si el nombre ya termina en `Code`, no se debe sugerir `CodeCode`; en ese caso ajustar el `description`. | <pre><code class="language-yaml"># ❌&#10;status:&#10;  type: string&#10;  description: "Código de estado"&#10;# ✅&#10;statusCode:&#10;  type: string&#10;  description: "Código de estado"</code></pre> |
| Evitar palabras reservadas en propiedades | ⚠️ Warning | En `components.schemas.*.properties`, evitar nombres que colisionen con palabras reservadas de JavaScript ES2023: `message`, `body`, `payload`, `class`, `default`, `function`, `null`, `true`, `false`, `return`, `var`, `let`, `const`, `new`, `this`, `typeof`, `delete`, `void`, `instanceof`, `in`, `if`, `else`, `for`, `while`, `do`, `break`, `continue`, `switch`, `case`, `throw`, `try`, `catch`, `finally`, `import`, `export`, `extends`, `super`, `yield`, `async`, `await`, `static`, `with`, `debugger`, `eval`, `arguments`, `prototype`, `constructor`. | <pre><code class=\"language-yaml\"># ❌\n properties:\n  payload:\n    type: string\n# ✅\nproperties:\n  payloadHash:\n    type: string</code></pre> |
| Claves sin caracteres Unicode no imprimibles | ⛔ Error | Los nombres de propiedades solo deben contener caracteres ASCII imprimibles (U+0020–U+007E). Están prohibidos los caracteres de control (U+0000–U+001F, U+007F) que son invisibles y rompen SDKs de generación de código. | <pre><code class=\"language-yaml\"># ❌ (carácter de control invisible embebido)\npropName: { type: string }\n# ✅\npropName: { type: string }</code></pre> |
| Llaves en inglés | ⛔ Error | Las keys de `schemas`, `messages` y `properties` **DEBEN** estar en inglés y en lowerCamelCase. **Spectral** detecta keys con tildes o ñ. **Copilot** evalúa semánticamente si la key pertenece al español aunque esté en ASCII. | <pre><code class="language-yaml"># ❌ (Spectral)&#10;teléfono: { type: string }&#10;# ❌ (Copilot)&#10;nombreCliente: { type: string }&#10;# ✅&#10;customerName: { type: string }</code></pre> |
| Keys de mensajes de canal terminan en `.message` | ⚠️ Warning | En `channels.*.messages`, las claves **DEBERÍAN** terminar en `.message` para consistencia (ej. `pudoCreated.message`). | <pre><code class="language-yaml"># ❌&#10;messages:&#10;  pudoCreated:&#10;    $ref: "#/components/messages/PudoCreatedMessage"&#10;# ✅&#10;messages:&#10;  pudoCreated.message:&#10;    $ref: "#/components/messages/PudoCreatedMessage"</code></pre> |
| entityType en plural | ⚠️ Warning | `entityType` **DEBERÍA** representarse en plural (ej. `orders`, `pudos`). | <pre><code class="language-yaml"># ❌&#10;entityType: "order"&#10;# ✅&#10;entityType: "orders"</code></pre> |

---

### Formato

| Regla | Severidad | Descripción | Ejemplo de código |
|---|---:|---|---|
| Documento YAML/JSON parseable | ⛔ Error | El documento **DEBE** ser parseable (YAML/JSON válido). | <pre><code class="language-yaml"># ❌&#10;channels:&#10;  topic-x:&#10;    description: "..."  bad&#10;# ✅&#10;channels:&#10;  topic-x:&#10;    description: "..."</code></pre> |
| Indentación estándar | ⚠️ Warning | El documento **DEBERÍA** usar 2 espacios y **NO DEBERÍA** usar tabs. | <pre><code class="language-yaml"># ❌&#10;channels:&#10;\ttopic-x:&#10;# ✅&#10;channels:&#10;  topic-x:</code></pre> |
| JSON: comillas dobles | ⚠️ Warning | Si el contrato se entrega en JSON, las cadenas **DEBERÍAN** usar comillas dobles (estándar JSON). | <pre><code class="language-json">{ "info": { "title": "..." } }</code></pre> |

---

### Claridad

| Regla | Severidad | Descripción | Ejemplo de código |
|---|---:|---|---|
| Descripción de la API presente | ⚠️ Warning | `info.description` **DEBERÍA** existir y explicar el propósito del contrato. | <pre><code class="language-yaml"># ✅&#10;info:&#10;  description: "Contrato de eventos para la creación de PUDOs."</code></pre> |
| Descripción de canales presente | ⚠️ Warning | `channels.*.description` **DEBERÍA** existir y ser entendible. | <pre><code class="language-yaml"># ✅&#10;channels:&#10;  topic-x:&#10;    description: "Tópico de creación de PUDOs"</code></pre> |
| Operación con summary | ⚠️ Warning | Cada operación **DEBERÍA** incluir `summary` (corto y descriptivo). | <pre><code class="language-yaml"># ✅&#10;sendPudoCreated:&#10;  action: send&#10;  summary: "Envía evento de creación de PUDOs"</code></pre> |
| Operación con description | ⚠️ Warning | Cada operación **DEBERÍA** incluir `description` con contexto funcional (qué, cuándo, quién consume). | <pre><code class="language-yaml"># ✅&#10;description: "Se publica cuando se crea un PUDO en el dominio de mensajería."</code></pre> |
| No incluir JSON en campos string | ⛔ Error | Si se requiere estructura, **DEBE** modelarse como `object`. Está prohibido JSON embebido en `string`. | <pre><code class="language-yaml"># ❌&#10;address:&#10;  type: string&#10;  example: "{\\"street\\":\\"X\\"}"&#10;# ✅&#10;address:&#10;  type: object&#10;  properties:&#10;    street: { type: string }</code></pre> |
| No duplicar headers en body | ⛔ Error | No repetir campos que ya existen en headers (ej. `timestamp`, `entityId`) dentro del payload. | <pre><code class="language-yaml"># ❌&#10;properties:&#10;  timestamp: { type: integer }&#10;# ✅ (usar los headers del trait)</code></pre> |
| Eventos minimalistas (PII) | ⚠️ Warning | Los eventos **DEBERÍAN** evitar datos personales; preferir IDs + API de consulta. Si hay PII, **DEBE** aplicar policy tags DME. | <pre><code class="language-yaml"># ❌&#10;properties:&#10;  rut:&#10;    type: string&#10;# ✅&#10;properties:&#10;  customerId:&#10;    type: string</code></pre> |
| Separación por tipo de evento | ⚠️ Warning | Se **RECOMIENDA** mantener una especificación por `eventType` (excepción CRUD sobre misma entidad/body compartido). | — |
| Máximo 2 versiones activas por eventType | ⚠️ Warning | Por cada `eventType` se **RECOMIENDA** mantener máx. 2 versiones activas; si existe una tercera, deprecar la más antigua. | — |
| Filter Policy incluye version si filtra por eventType | ⚠️ Warning | Si se documenta una Filter Policy, debe incluir `eventType` **y** `version` (usar `$or` para combinaciones). | <pre><code class="language-json">{ "eventType": ["orderModified"], "version": ["1.0"] }</code></pre> |
| Updated: documentar si es partial update | ⚠️ Warning | En eventos `*Updated`, declarar si el payload es completo o parcial, y modelar campos opcionales cuando corresponda. | — |
| Evitar campos en desuso, vacíos o nulos | ⚠️ Warning | El payload **DEBERÍA** evitar campos que no aportan datos (en desuso, vacíos o nulos). | <pre><code class="language-yaml"># ❌&#10;obsoleteField: { type: string, example: "" }</code></pre> |
| Booleanos no nulos | ⚠️ Warning | Las propiedades `type: boolean` **NO DEBEN** ser `nullable: true` ni tener `default: null`. | <pre><code class="language-yaml"># ❌&#10;internal:&#10;  type: boolean&#10;  nullable: true&#10;# ✅&#10;internal:&#10;  type: boolean</code></pre> |
| Duplicación de datos entre campos | ⚠️ Warning | Evitar propiedades en el mismo schema que contengan la misma información semántica bajo nombres distintos. Ante duda razonable, no reportar. | <pre><code class="language-yaml"># ❌&#10;properties:&#10;  date: { type: string }&#10;  creationDate: { type: string }&#10;# ✅&#10;properties:&#10;  createdAt: { type: string, format: date-time }</code></pre> |
| Tipo de dato de `example` consistente con `type` | ⛔ Error | El valor de `example` debe ser compatible con el `type` declarado. `type: integer` requiere número sin comillas; `type: boolean` requiere `true`/`false` sin comillas; `type: array` requiere una lista. Un ejemplo mal tipado rompe generación de SDKs y mocks. | <pre><code class=\"language-yaml\"># ❌\nquantity:\n  type: integer\n  example: \"5\"\nactive:\n  type: boolean\n  example: \"true\"\n# ✅\nquantity:\n  type: integer\n  example: 5\nactive:\n  type: boolean\n  example: true</code></pre> |
| Identificadores de entidades en el evento | ⚠️ Warning | El schema del payload **DEBERÍA** incluir al menos un campo identificador de la entidad principal (propiedad terminada en `Id` o llamada `id`). | <pre><code class="language-yaml"># ❌&#10;properties:&#10;  name: { type: string }&#10;# ✅&#10;properties:&#10;  orderId: { type: string }</code></pre> |

---

## 7. Ejemplo Práctico (AsyncAPI válido)

```yaml
asyncapi: 3.0.0
id: 'https://github.com/acme-org/pdoohd-pudom-pudos-producer'
info:
  title: 'PUDOs | Eventos de puntos pickup'
  version: '1.2.0'
  description: 'Contrato de eventos para creación de PUDOs.'
  contact:
    name: 'Equipo Arquitectura y APIs'
    email: 'arquitectura@acme-org.example'
  license:
    name: 'Apache 2.0'
    url: 'https://www.apache.org/licenses/LICENSE-2.0'
  tags:
    - name: 'pudos'
      description: 'Eventos del dominio PUDOs'

defaultContentType: 'application/json'

servers:
  production-sns:
    host: 'arn:aws:sns:us-east-1:000000000000'
    protocol: sns
    protocolVersion: '1.0.0'
    description: 'SNS Productivo (IAM gestionado por infraestructura).'

  production-sqs:
    host: 'arn:aws:sqs:us-east-1:000000000000'
    protocol: sqs
    protocolVersion: '1.0.0'
    description: 'SQS Productivo (IAM gestionado por infraestructura).'

channels:
  topic-pdoohd-pudom-pudos-created:
    address: 'topic-pdoohd-pudom-pudos-created'
    description: 'Tópico SNS: evento de creación de PUDOs.'
    messages:
      pudoCreated.message:
        $ref: '#/components/messages/PudoCreatedMessage'

  queue-pdoohd-pudom-pudos-created-pudos:
    address: 'queue-pdoohd-pudom-pudos-created-pudos'
    description: 'Cola SQS: consumo del evento de creación de PUDOs.'
    messages:
      pudoCreated.message:
        $ref: '#/components/messages/PudoCreatedMessage'

operations:
  sendPudoCreated:
    action: send
    summary: 'Publica evento de creación de PUDOs.'
    description: 'Se publica cuando se crea un PUDO en el dominio de mensajería.'
    channel:
      $ref: '#/channels/topic-pdoohd-pudom-pudos-created'
    messages:
      - $ref: '#/channels/topic-pdoohd-pudom-pudos-created/messages/pudoCreated.message'

  receivePudoCreated:
    action: receive
    summary: 'Consume evento de creación de PUDOs.'
    description: 'Se consume cuando llega el evento pudoCreated a la cola SQS para procesamiento downstream.'
    channel:
      $ref: '#/channels/queue-pdoohd-pudom-pudos-created-pudos'
    messages:
      - $ref: '#/channels/queue-pdoohd-pudom-pudos-created-pudos/messages/pudoCreated.message'

components:
  messageTraits:
    commonHeaders:
      headers:
        type: object
        required: [eventId,eventType,entityId,entityType,timestamp,datetime,version,internal]
        properties:
          eventId:
            type: string
            description: 'UUID o hex32.'
          eventType:
            type: string
            description: 'lowerCamelCase en pasado (ej. pudoCreated).'
          entityId:
            type: string
          entityType:
            type: string
            description: 'Entidad en plural.'
          timestamp:
            type: integer
            format: int64
            description: 'Epoch ms UTC (evento de negocio).'
            example: 1731008444499
          datetime:
            type: string
            format: date-time
            description: 'RFC3339 UTC.'
            example: '2024-11-07T19:40:44.499Z'
          version:
            type: string
            pattern: '^\d+\.\d+$'
            example: '1.0'
          internal:
            type: boolean
            description: 'true = solo integraciones internas; false = puede ser externo.'

    transportHeaders:
      headers:
        type: object
        required: [channel]
        properties:
          channel:
            type: string
            enum: [web,mobile,api,batch_process,internal_system,external_partner]

    businessHeaders:
      headers:
        type: object
        required: [domain,subdomain,businessCapability]
        properties:
          domain:
            type: string
            pattern: '^[a-z0-9]+$'
            example: 'pdoohd'
          subdomain:
            type: string
            pattern: '^[a-z0-9]+$'
            example: 'pdoohd'
          businessCapability:
            type: string
            pattern: '^[a-z0-9]+$'
            example: 'pudom'

    trackingHeaders:
      headers:
        type: object
        required: [traceId,spanId]
        properties:
          traceId:
            type: string
            pattern: '^[0-9a-f]{32}$'
          spanId:
            type: string
            pattern: '^[0-9a-f]{16}$'
          parentSpanId:
            type: string
            pattern: '^[0-9a-f]{16}$'

  messages:
    PudoCreatedMessage:
      x-dme-path-reference: 'PudoCreatedBody'
      x-dme-fullpayload: true
      x-dme-operation: 'C'
      name: 'pudoCreated'
      title: 'pudoCreated'
      summary: 'Evento de creación de un PUDO.'
      traits:
        - $ref: '#/components/messageTraits/commonHeaders'
        - $ref: '#/components/messageTraits/transportHeaders'
        - $ref: '#/components/messageTraits/businessHeaders'
        - $ref: '#/components/messageTraits/trackingHeaders'
      payload:
        $ref: '#/components/schemas/PudoCreatedBody'

  schemas:
    PudoCreatedBody:
      x-dme-path-reference: 'PudoCreatedBody'
      x-dme-version: '1.0.0'
      type: object
      required: [pickupPointId]
      properties:
        pickupPointId:
          type: string
          description: 'Identificador del punto pickup.'
          example: 'PUDO-9249'
        contactEmail:
          type: string
          format: email
          description: 'Email de contacto (si aplica).'
          example: 'contacto@acme-org.example'
          x-dme-securityPolicyTags:
            - 'projects/tecnogia-arqdat-governance-prd/locations/us/taxonomies/8649944501028248774/policyTags/8984512984767038865'
```

---

## 8. Checklist para equipos

- [ ] `asyncapi: 3.0.0` (o versión aprobada 3.1.x).
- [ ] `id` es URL absoluta del repositorio del producer.
- [ ] `info` completo (title / version / description / contact / license / tags).
- [ ] `defaultContentType: application/json`.
- [ ] `servers` declarados con `protocol` y `protocolVersion`.
- [ ] Gobierno EDA: contrato separado por `eventType` (o CRUD unificado cuando aplique).
- [ ] Versionamiento: máx. **2 versiones activas** por `eventType`; si hay una tercera, existe plan de deprecación.
- [ ] Si hay Filter Policy SNS/SQS, incluye `eventType` + `version` y está documentada (ej. `x-sns-filterPolicy`).
- [ ] Para eventos `*Updated`, definido si es full payload o partial update; schema modela campos opcionales.
- [ ] Canales nombrados con `topic-...` / `queue-...`, y `address` correcto.
- [ ] Operaciones con `action: send|receive`, `summary`, `channel $ref` y `messages` referenciando mensajes del canal.
- [ ] Mensajes con los 4 traits corporativos de headers.
- [ ] `timestamp` (epoch ms) y `datetime` (RFC3339 UTC) en `commonHeaders`.
- [ ] Sin duplicar headers en el payload.
- [ ] Keys en **inglés** (evitar términos en español; usar equivalentes como `taxId`, `city`, `createdAt`, etc.).
- [ ] Campos que representan códigos terminan en `Code` (ej. `regionCode`).
- [ ] Propiedades DME presentes (si aplica) y policy tags en campos PII.
