# Lineamientos y Estándares de OpenAPI (Corporativo)

**Versión del documento:** 1.6
**Fecha:** 2026-03-05

---

## Historial de cambios

| Versión | Cambios |

---

## 1. Introducción y Alcance

La organización estandariza OpenAPI para asegurar **consistencia**, **interoperabilidad**, **seguridad**, **observabilidad** y **calidad** en APIs HTTP, habilitando automatización (validación en CI/CD, generación de documentación y SDKs, pruebas contractuales y gobierno de cambios).

Este estándar aplica a:

- APIs HTTP expuestas por servicios internos y/o externos.
- APIs de integración entre dominios y capacidades de negocio.
- Equipos que publican/consumen APIs y deben cumplir con validaciones corporativas.

### 1.1 Referencias normativas

Este estándar se basa en las siguientes fuentes oficiales:

- **OpenAPI Specification (OAS) 3.0.x** (OpenAPI Initiative).
- HTTP Semantics (RFC 9110) y registro IANA de **HTTP Status Codes**.
- **Problem Details** para manejo de errores (RFC 9457).
- **W3C Trace Context** para propagación de trazas (`traceparent`/`tracestate`).

### 1.2 Equivalencia normativa (AsyncAPI → OpenAPI)

Este estándar adapta las reglas transversales de modelado (datos, nomenclatura, metadatos y versionado) a contratos REST en OpenAPI, descartando reglas propias de arquitecturas orientadas a eventos (EDA).

**Reglas que aplican 1:1 a OpenAPI:**

- **Llaves en inglés + lowerCamelCase:** `properties` y `parameters` deben ser legibles y consistentes.
- **Schemas estrictos:** bodies de request/response deben modelarse como `type: object` con `properties`. Arrays deben definir `items`.
- **No JSON embebido en string:** si existe estructura, modelar como `object`.
- **Versionado SemVer del contrato:** `info.version` debe ser `MAJOR.MINOR.PATCH`; en la URL se usa **solo MAJOR** (ej. `/v1`).
- **Metadatos mínimos:** `info.title`, `info.description` y `summary/description` en operaciones para claridad.

**Adaptaciones REST (desde reglas transaccionales AsyncAPI):**

- En REST se modela con `paths` y métodos HTTP (`GET/POST/PUT/PATCH/DELETE`) en lugar de `channels/operations`.
- Los headers EDA (`eventId`, `eventType`, etc.) no aplican; se reemplazan por **trazabilidad HTTP** basada en W3C Trace Context (`traceparent`/`tracestate`).

**Reglas que no aplican a OpenAPI:**

- `channels`, `address`, `operations.action` (send/receive).
- Headers EDA: `eventId`, `eventType`, `commonHeaders`, `transportHeaders`, bindings SNS/SQS y Filter Policies.

---

## 2. Especificaciones del Documento OpenAPI

### 2.1 Versión de OpenAPI

- La especificación **DEBE** usar **OpenAPI 3.0.x**. El baseline recomendado es **OpenAPI 3.0.3**.
- La especificación **NO DEBERÍA** usar 3.1+ salvo decisión corporativa explícita, ya que esta versión introduce cambios al modelo de JSON Schema.

El campo `openapi` declara la versión del **estándar OpenAPI** y es independiente de `info.version`, que representa la versión del contrato de la API.

```yaml
openapi: "3.0.3"
info:
  title: "Orders API - Gestión de órdenes"
  version: "1.2.0"   # SemVer del contrato
```

---

### 2.2 Metadatos obligatorios

El contrato **DEBE** incluir los siguientes campos:

- `openapi`: versión del estándar (ej. `3.0.3`).
- `info`:
  - `info.title` **DEBE** ser descriptivo y tener **> 10** caracteres.
  - `info.version` **DEBE** existir en formato SemVer **MAJOR.MINOR.PATCH**.
  - `info.description` **DEBERÍA** existir con el propósito y alcance de la API.
  - `info.contact` **DEBERÍA** existir (name + email).
  - `info.license` **DEBERÍA** existir (name + url).
- `servers`: **DEBE** existir y contener al menos un servidor (ver §2.3).
- `paths`: **DEBE** existir y contener al menos un path.

Elementos recomendados:

- `tags`: **DEBERÍA** existir para clasificar capacidades/dominios.
- `externalDocs`: **DEBERÍA** existir cuando aplique (runbook, wiki, portal, etc.).

#### 2.2.1 Extensión corporativa: fuente de verdad del contrato

OpenAPI no define un campo `id` normativo como AsyncAPI. Para identificar la fuente de verdad del contrato, el documento **DEBE** incluir:

- `info.x-source-repo`: URL absoluta del repositorio Git donde reside el contrato.

```yaml
info:
  title: "Orders API"
  version: "1.0.0"
  x-source-repo: "https://github.com/acme-org/orders-api"
```

---

### 2.3 Servers

- `servers` **DEBE** declarar al menos un servidor con `url`.
- En APIs productivas, `servers` **DEBERÍA** incluir al menos un server de **producción** y uno de **sandbox/stage**.
- `servers[].description` **DEBERÍA** explicar el uso, restricciones o mecanismo de autenticación.
- En producción, `servers[].url` **DEBE** usar `https://` (ver §5.2).

#### 2.3.1 Estándar de URLs ARCHBX

Para `servers[].url`, ARCHBX define escenarios según el tipo de exposición. El estándar general omite cualquier prefijo `/api/` antes del `{capability}`; dicho prefijo solo se permite si existe un estándar específico de plataforma o legado debidamente documentado.

| Escenario | Uso | Patrón `servers[].url` | Notas |
|---:|---|---|---|
| 1 | Gateway / no-prod (dev/qa/stage) | `https://api.<env>.blue.cl/{capability}/{service}/v{MAJOR}` | `https` obligatorio cuando se publica vía Gateway. |
| 2 | Gateway / producción | `https://api.blue.cl/{capability}/{service}/v{MAJOR}` | Sin `/api/` por defecto. |
| 3 | BFF (Backend-for-Frontend) | `https://int.api[.<env>].blue.cl/bff/{application-name}/v{MAJOR}` | `int.api` como subdominio fijo; `/bff/` como prefijo obligatorio. En prod se omite `.<env>`. |
| 4a | DNS interno (microservicios intra-cluster) | `http://{service-name}.{ns-name}/v{MAJOR}` | El host **DEBE** incluir `.{ns-name}`. `http` permitido solo intra-cluster. |
| 4b | Ingress externo (microservicios expuestos) | `https://{service-name}.blueexpress.tech/v{MAJOR}` | `https` obligatorio. Usar solo cuando el servicio se expone fuera del clúster. |

**Reglas críticas:**

- `info.version` **DEBE** ser SemVer (`MAJOR.MINOR.PATCH`).
- En URLs se usa **solo MAJOR**: `/v1`, `/v2`, etc. Está **prohibido** `/v1.2/` o `/v1.2.3/`.
- En el escenario 3, el prefijo `/bff/` es **obligatorio** y el subdominio `int.api` es fijo.
- En el escenario 4a intra-cluster, el request completo queda como: `http://{service-name}.{ns-name}/v{MAJOR}/{resource}`, donde `{resource}` proviene de `paths`.
- En el escenario 4b Ingress externo, el host es `{service-name}.blueexpress.tech` y **DEBE** usar `https`.

---

## 3. Convenciones REST (Design Guidelines)

### 3.1 Nomenclatura de Paths

#### 3.1.1 Regla general

- Los `paths` **DEBEN** modelar **recursos** (sustantivos), no acciones.
- Las rutas **DEBEN** usar **kebab-case** en minúsculas.
- Los recursos colección **DEBEN** ir en **plural**.
- Está prohibido el uso de verbos CRUD en el path (`/create`, `/update`, `/delete`, `/get`).

**Estándar ARCHBX — URL base obligatoria:**

- `servers[].url` **DEBE** incluir versionado **MAJOR** en la ruta: `/v{MAJOR}` (ej. `/v1`).
- El host **DEBE** seguir el patrón `api.<env>.blue.cl` (ej. `api.dev.blue.cl`) o `api.blue.cl` (prod).
- La ruta base **DEBE** seguir: `/{capability}/{service}/v{MAJOR}` (kebab-case, sin subdominios por servicio).

```yaml
# ✅
/orders
/orders/{orderId}
/orders/{orderId}/items

# ❌
/getOrders
/order/create
/orders/update-status
```

Ejemplo de URL base correcta:
```
https://api.dev.blue.cl/georf/geographical-data/v1
```

Anti-patrón (NO permitido):
```
https://georf.api.dev.blue.cl/georf/srv/geographical-data
```

#### 3.1.2 Jerarquía de recursos (sub-recursos)

Se permite expresar jerarquía cuando existe una relación clara entre recursos:

```
/customers/{customerId}/accounts/{accountId}
```

Se deben evitar jerarquías excesivamente profundas, ya que dificultan la legibilidad y el mantenimiento del contrato.

#### 3.1.3 Variables de path

- Las variables `{...}` **DEBEN** ser `lowerCamelCase` (ej. `{customerId}`).
- Cada path param **DEBE** declararse en `parameters` con `in: path`, `required: true` y `schema` definido.

---

### 3.2 Versionamiento (Contrato vs URL vs Backend)

#### 3.2.1 Versión del contrato (`info.version`)

El contrato **DEBE** especificarse siempre con **SemVer**:

- **MAJOR:** cambios incompatibles (breaking changes).
- **MINOR:** cambios retrocompatibles (nuevos campos opcionales, nuevos endpoints).
- **PATCH:** correcciones que no alteran el comportamiento (fixes de documentación).

```yaml
info:
  version: "1.2.0"
```

#### 3.2.2 Versión en la URL (solo MAJOR)

En la URL se usa **solo MAJOR** para evitar la proliferación de rutas por cambios retrocompatibles:

- Está **prohibido** usar `MINOR` o `PATCH` en la URL (ej. `/v1.2/`, `/v1.2.3/`).
- Se recomienda ubicar el versionado lo más a la izquierda posible en la URL.

```
✅  https://api.example.com/v1/customers
✅  https://api.example.com/{domain}/v1/customers
```

#### 3.2.3 Versionamiento del backend

El backend (microservicio/lógica) puede versionar de forma independiente al contrato. Cuando se incrementa el MAJOR del contrato, suele liberarse una nueva versión del backend para permitir convivencia temporal y migración controlada de clientes.

---

### 3.3 Operaciones (HTTP Methods)

Los métodos HTTP deben usarse conforme a su semántica estándar:

| Método | Uso |
|---|---|
| `GET` | Consulta de recursos |
| `POST` | Creación de recursos o acciones no idempotentes |
| `PUT` | Reemplazo idempotente de un recurso |
| `PATCH` | Actualización parcial de un recurso |
| `DELETE` | Eliminación de un recurso |

#### 3.3.1 Reglas generales por método

- `GET` para filtrado y búsqueda simple vía query params.
- `POST` para búsquedas complejas con múltiples criterios en body: `POST /v1/customers/searches`.

---

### 3.4 Parámetros (Path / Query / Header)

- Los parámetros **DEBEN** declarar `schema` con sus tipos.
- Se deben evitar parámetros sin tipo definido.
- Los parámetros reutilizables deben centralizarse en `components.parameters` y referenciarse con `$ref`.

#### 3.4.1 Query parameters (paginación y filtros)

Para endpoints de colecciones se recomienda documentar los siguientes parámetros estándar:

- `limit` (int > 0): número máximo de registros a retornar.
- `offset` (int >= 0): desplazamiento para paginación.
- `sort` (por defecto ascendente; prefijo `-` para descendente).
- `fields` (selección de campos a incluir en la respuesta).

```
?limit=200&offset=400
?sort=-popularity
?fields=name,code,date
```

#### 3.4.2 Headers de trazabilidad (W3C Trace Context)

- Cada operación **DEBE** documentar el header `traceparent` conforme al estándar W3C Trace Context.
- Formato: `version-trace-id-parent-id-trace-flags` (ej. `00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01`).
- `tracestate` es opcional, pero **DEBERÍA** documentarse cuando el ecosistema lo utilice (APM, gateways, etc.).

Se recomienda modelar estos headers como `components.parameters` reutilizables y referenciarlos con `$ref` en todas las operaciones.

---

## 4. Estructura de Mensajes y Payloads

### 4.1 Body (Request/Response)

- Los campos JSON (schemas) **DEBEN** ser `lowerCamelCase`. No se debe mezclar `snake_case` ni `PascalCase` en propiedades.
- **Palabras reservadas:** evitar palabras reservadas de JavaScript como nombres de propiedades. Están prohibidas: `message`, `body`, `payload`, `class`, `default`, `function`, `null`, `true`, `false`, `return`, `var`, `let`, `const`, `new`, `this`, `typeof`, `delete`, `void`, `instanceof`, `in`, `if`, `else`, `for`, `while`, `do`, `break`, `continue`, `switch`, `case`, `throw`, `try`, `catch`, `finally`, `import`, `export`, `extends`, `super`, `yield`, `async`, `await`, `static`, `with`, `debugger`, `eval`, `arguments`, `prototype`, `constructor`.
- **Caracteres Unicode no imprimibles:** los nombres de propiedades solo deben contener caracteres ASCII imprimibles (U+0020–U+007E). Están prohibidos los caracteres de control (U+0000–U+001F, U+007F) que son invisibles y rompen SDKs de generación de código.
- **Consistencia de `example` con `type`:** el valor del campo `example` debe ser compatible con el `type` declarado en la misma propiedad. Por ejemplo, `type: integer` con `example: "5"` es inválido (el valor debe ser numérico, no string). Lo mismo aplica para `boolean`, `array` y `object`.
- `requestBody` en `POST/PUT/PATCH` **DEBERÍA** definir `content` con media types (`application/json`).
- Para payloads complejos, se deben documentar `examples` o `example` en `content`.

#### 4.1.1 No duplicidad de IDs (path vs body)

Si un identificador ya está presente en el `path` (ej. `/orders/{orderId}`), **NO** debe repetirse en el body JSON del request/response asociado. La duplicidad solo se permite cuando es estrictamente necesaria (compatibilidad con legado, integración externa, etc.) y debe quedar **justificada** en el campo `description` del elemento duplicado.

```yaml
# ❌ Anti-patrón: orderId ya viene en el path
paths:
  /orders/{orderId}:
    put:
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                orderId:
                  type: string
```

---

### 4.2 Componentes, Esquemas y Reutilización

#### 4.2.1 Schema Object en OpenAPI 3.0.x

En OpenAPI 3.0.x, el `Schema Object` se basa en un subconjunto de JSON Schema alineado históricamente a Draft 4. Se deben evitar keywords de dialectos posteriores (ej. 2020-12), ya que no son compatibles con esta versión del estándar.

#### 4.2.2 Reglas de modelado de schemas

- Los schemas de entidades/DTOs **DEBEN** ser `type: object` cuando corresponda.
- Si una propiedad es `array`, **DEBE** declarar `items`.
- Está prohibido embeber JSON como `string`; si existe estructura, usar `object`.
- Los modelos reutilizables deben centralizarse en `components.schemas`.
- Las `responses`, `parameters` y `requestBodies` reutilizables deben centralizarse en `components`.
- Las propiedades `type: boolean` **NO DEBEN** ser `nullable: true` ni tener `default: null`. Siempre deben tener un valor definido (`true` o `false`).
- Se deben evitar propiedades en el mismo schema que contengan la misma información semántica bajo nombres distintos (duplicación de datos).

---

## 5. Seguridad y Protocolos

### 5.1 `securitySchemes` y `security`

- La API **DEBE** declarar `components.securitySchemes` con al menos un esquema (ej. `oauth2`, `apiKey`, `bearer`). Requerido por el estándar corporativo de Zero Trust (JWT / Client Credentials). Esta condición es validada por **Spectral** (⛔ Error).
- La API **DEBERÍA** declarar `security` a nivel raíz como configuración global por defecto.
- Si una operación es pública, **DEBE** sobreescribir con `security: []`.

Ejemplo (OAuth2):

```yaml
components:
  securitySchemes:
    oauth2-bx-devportal-dev:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://desa.sso.bluex.cl/auth/realms/nonbx-externals/protocol/openid-connect/auth
          tokenUrl: https://desa.sso.bluex.cl/auth/realms/nonbx-externals/protocol/openid-connect/token
          scopes: {}
security:
  - oauth2-bx-devportal-dev: []
```

### 5.2 HTTPS

- Los `servers[].url` en producción **DEBEN** usar `https://`.
- Excepción: en DNS interno (escenario 4) se permite `http://` exclusivamente intra-cluster.
- La existencia de un server no-TLS debe justificarse (legado) y ser aprobada por Arquitectura.

### 5.3 Manejo de errores estándar (RFC 9457)

Las respuestas de error corporativas se basan en **Problem Details** (RFC 9457).

**Requisitos obligatorios (⛔ Error crítico):**

- Para respuestas **4xx** y **5xx**, el contrato **DEBE** declarar media type `application/problem+json` y schema `ProblemDetails`.
- El schema `ProblemDetails` **DEBE** incluir los campos `type`, `title`, `status` y `detail`. El campo `instance` es opcional.

Definición del schema:

```yaml
components:
  schemas:
    ProblemDetails:
      type: object
      required: [type, title, status, detail]
      properties:
        type:
          type: string
          format: uri
          example: "https://errors.blue.cl/problem/invalid-request"
        title:
          type: string
          example: "Solicitud inválida"
        status:
          type: integer
          format: int32
          example: 400
        detail:
          type: string
          example: "El campo 'customerId' es obligatorio."
        instance:
          type: string
          format: uri
          example: "urn:blue:orders:error:8f0d3b"
```

Uso en responses:

```yaml
responses:
  "400":
    description: "Bad Request"
    content:
      application/problem+json:
        schema:
          $ref: "#/components/schemas/ProblemDetails"
```

### 5.4 Códigos de estado recomendados

| Método | Código | Descripción |
|---|---|---|
| `POST` | `201 Created` | Recurso creado; incluir header `Location`. |
| `POST` | `202 Accepted` | Procesamiento asíncrono. |
| `POST` | `409 Conflict` | El recurso ya existe. |
| `GET` | `200 OK` | Resultado exitoso. |
| `GET` | `404 Not Found` | ID inválido o recurso inexistente. |
| `PUT/PATCH` | `200 OK` o `204 No Content` | Actualización exitosa. |
| `PUT/PATCH` | `400 Bad Request` / `404 Not Found` / `409 Conflict` | Errores de actualización. |
| `DELETE` | `200 OK` o `204 No Content` | Eliminación exitosa. |
| `DELETE` | `202 Accepted` | Eliminación asíncrona. |
| `DELETE` | `404 Not Found` | Recurso inexistente. |

---

## 6. Sistema de Reglas (Validador) y Criterios de Aceptación

### 6.1 Categorías de reglas

El validador organiza las validaciones en cinco categorías:

- **Estructura:** cumplimiento mínimo del esquema OpenAPI (hard gate).
- **Headers:** trazabilidad y observabilidad por operación.
- **Nomenclatura:** consistencia de naming en paths, operationId, schemas y params.
- **Formato:** documento YAML/JSON válido y estilo base.
- **Claridad:** documentación suficiente y buenas prácticas de diseño.

### 6.2 Severidad

| Nivel | Descripción |
|---|---|
| **⛔ Error** | Bloqueante. Si existe al menos un error, el contrato **NO** aprueba. |
| **⚠️ Warning** | No bloqueante. Se registra como deuda técnica. |

### 6.3 Determinístico vs Heurístico

- **Spectral (determinístico):** valida estructura, presencia, unicidad, patrones y tipos. Incluye desde v1.4 la validación de `Servers URL estándar ARCHBX` (con soporte a los 4 escenarios ARCHBX desde v1.6) y cobertura completa de `RFC 9457 Problem Details`. Desde v1.6 valida también `Autenticación documentada` (`components.securitySchemes`) como ⛔ Error.
- **Copilot (heurístico):** evalúa claridad, coherencia, ejemplos y consistencia conceptual sin funciones custom. Catálogo de 17 reglas desde v1.6, incluyendo reglas de duplicación, refuerzo semántico de llaves en inglés, no-duplicación de IDs entre path y body, semántica de variables de path y consistencia de `example` con `type`.

---

## 6.4 Anexo A — Reglas del validador

Este anexo es **normativo**. Cada regla se evalúa en CI/CD mediante el validador corporativo.

**Leyenda de herramientas:** `Spectral` = determinístico (bloquea en CI/CD). `Copilot` = heurístico (PR comment). `Ambas` = cobertura complementaria en ambas herramientas.

---

### Estructura

| Regla | Severidad | Herramienta | Descripción | Ejemplo de código |
|---|---:|:---:|---|---|
| Versión de OpenAPI | ⛔ Error | Spectral | El documento **DEBE** declarar `openapi: 3.0.x` (baseline recomendado: `3.0.3`). | <pre><code class="language-yaml"># ❌&#10;openapi: "3.1.0"&#10;# ✅&#10;openapi: "3.0.3"</code></pre> |
| Título presente y descriptivo | ⛔ Error | Spectral | `info.title` **DEBE** existir y tener más de 10 caracteres. | <pre><code class="language-yaml"># ❌&#10;info: { title: "Orders" }&#10;# ✅&#10;info: { title: "Orders API - Gestión de órdenes" }</code></pre> |
| Versión del contrato presente (SemVer) | ⛔ Error | Spectral | `info.version` **DEBE** existir y seguir `MAJOR.MINOR.PATCH`. | <pre><code class="language-yaml"># ❌&#10;info: { version: "v1" }&#10;# ✅&#10;info: { version: "1.2.0" }</code></pre> |
| Repositorio fuente presente | ⛔ Error | Spectral | `info.x-source-repo` **DEBE** ser URL absoluta del repositorio Git. | <pre><code class="language-yaml"># ✅&#10;info:&#10;  x-source-repo: "https://github.com/acme-org/orders-api"</code></pre> |
| Servers definidos | ⛔ Error | Spectral | El documento **DEBE** declarar `servers` con al menos 1 `url`. | <pre><code class="language-yaml"># ❌&#10;servers: []&#10;# ✅&#10;servers:&#10;  - url: "https://api.acme.com"</code></pre> |
| Paths definidos | ⛔ Error | Spectral | El documento **DEBE** declarar `paths` y contener al menos un path. | <pre><code class="language-yaml"># ❌&#10;paths: {}&#10;# ✅&#10;paths:&#10;  /health:&#10;    get:&#10;      responses:&#10;        "200":&#10;          description: "OK"</code></pre> |
| Operación tiene operationId | ⛔ Error | Spectral | Cada operación **DEBE** tener `operationId` **único** en todo el documento. | <pre><code class="language-yaml"># ✅&#10;get:&#10;  operationId: "getHealth"</code></pre> |
| Operación tiene responses | ⛔ Error | Spectral | Cada operación **DEBE** definir `responses` con al menos un status code. | <pre><code class="language-yaml"># ✅&#10;responses:&#10;  "200": { description: "OK" }</code></pre> |
| Response tiene description | ⛔ Error | Spectral | Cada `responses.<code>` **DEBE** tener `description` no vacío. | <pre><code class="language-yaml"># ✅&#10;"200": { description: "OK" }</code></pre> |
| requestBody con content cuando aplica | ⚠️ Warning | Spectral | En `POST/PUT/PATCH`, `requestBody` **DEBERÍA** definir `content` con media types. | <pre><code class="language-yaml"># ✅&#10;requestBody:&#10;  content:&#10;    application/json:&#10;      schema: { $ref: "#/components/schemas/Foo" }</code></pre> |
| Arrays con items definidos | ⛔ Error | Spectral | Toda propiedad `type: array` **DEBE** definir `items`. | <pre><code class="language-yaml"># ❌&#10;tags:&#10;  type: array&#10;# ✅&#10;tags:&#10;  type: array&#10;  items:&#10;    type: string</code></pre> |
| Autenticación documentada | ⛔ Error | Spectral | La API **DEBE** declarar `components.securitySchemes` con al menos un esquema de seguridad (oauth2, apiKey, bearer). Requerido por el estándar corporativo de Zero Trust. Operaciones públicas deben sobreescribir con `security: []`. | <pre><code class="language-yaml"># ❌&#10;components: {}&#10;# ✅&#10;components:&#10;  securitySchemes:&#10;    oauth2-bx: { type: oauth2 }</code></pre> |

---

### Headers

| Regla | Severidad | Herramienta | Descripción | Ejemplo de código |
|---|---:|:---:|---|---|
| Trazabilidad W3C traceparent por operación | ⛔ Error | Spectral | Cada operación **DEBE** documentar `traceparent` como header parameter, idealmente reutilizable vía `$ref`. | <pre><code class="language-yaml"># ✅&#10;parameters:&#10;  - $ref: "#/components/parameters/TraceparentHeader"</code></pre> |
| `tracestate` opcional documentado | ⚠️ Warning | Spectral | **DEBERÍA** documentarse `tracestate` como header opcional. | <pre><code class="language-yaml"># ✅&#10;parameters:&#10;  - $ref: "#/components/parameters/TracestateHeader"</code></pre> |

El header `x-request-id` se considera **legacy/opcional**. Si se mantiene por compatibilidad puede documentarse, pero no es obligatorio cuando se emplea trazabilidad W3C (`traceparent`/`tracestate`).

---

### Nomenclatura

| Regla | Severidad | Herramienta | Descripción | Ejemplo de código |
|---|---:|:---:|---|---|
| Paths en kebab-case y minúsculas | ⛔ Error | Spectral | `paths` **DEBEN** ser minúsculas y usar kebab-case (excepto variables `{...}`). | <pre><code class="language-yaml"># ❌&#10;/GetOrders: {}&#10;# ✅&#10;/orders: {}</code></pre> |
| Recursos en plural | ⚠️ Warning | Copilot | Colecciones **DEBERÍAN** estar en plural (`/orders`, `/customers`). Singletons válidos: `/health`, `/status`, `/me`, `/config`. | <pre><code class="language-yaml"># ❌&#10;/order: {}&#10;# ✅&#10;/orders: {}</code></pre> |
| Prohibido verbos CRUD exactos en el path | ⛔ Error | Spectral | No usar `/create`, `/update`, `/delete`, `/get`, `/list`, `/find`, etc. como segmentos del path. | <pre><code class="language-yaml"># ❌&#10;/customers/create&#10;# ✅&#10;/customers</code></pre> |
| Verbos en paths (camelCase) | ⛔ Error | Copilot | Segmentos fijos del path **NO DEBEN** comenzar con verbo de acción en camelCase (ej. `createOrder`, `getUsers`). Complementa la regla Spectral de verbos exactos. | <pre><code class="language-yaml"># ❌&#10;/createOrder&#10;# ✅&#10;/orders</code></pre> |
| Variables de path en lowerCamelCase | ⛔ Error | Spectral | Variables de path deben ser `lowerCamelCase` (ej. `{orderId}`, `{customerId}`). No usar `snake_case`. | <pre><code class="language-yaml"># ❌&#10;/orders/{order_id}&#10;# ✅&#10;/orders/{orderId}</code></pre> |
| Propiedades en lowerCamelCase | ⛔ Error | Spectral | Las propiedades de schemas **DEBEN** estar en `lowerCamelCase`. Está prohibido `snake_case` en JSON. | <pre><code class="language-yaml"># ❌&#10;properties: { order_id: { type: string } }&#10;# ✅&#10;properties: { orderId: { type: string } }</code></pre> |
| Schemas en PascalCase | ⚠️ Warning | Spectral | Los nombres en `components.schemas` **DEBERÍAN** ser PascalCase. | <pre><code class="language-yaml"># ❌&#10;components: { schemas: { order: {} } }&#10;# ✅&#10;components: { schemas: { Order: {} } }</code></pre> |
| Llaves en inglés | ⛔ Error | Spectral + Copilot | `properties`, `parameters` y nombres en `components` **DEBEN** estar en inglés y lowerCamelCase. **Spectral** detecta keys con tildes o ñ (`áéíóúñ`). **Copilot** evalúa semánticamente si la key pertenece al español (ej. `nombreCliente`, `fechaPago`) aunque esté en ASCII. | <pre><code class="language-yaml"># ❌ (Spectral)&#10;properties: { teléfono: { type: string } }&#10;# ❌ (Copilot)&#10;properties: { nombreCliente: { type: string } }&#10;# ✅&#10;properties: { customerName: { type: string } }</code></pre> |
| operationId en lowerCamelCase | ⛔ Error | Spectral | `operationId` **DEBE** seguir lowerCamelCase (ej. `listOrders`, `createOrder`). | <pre><code class="language-yaml"># ❌&#10;operationId: "List_Orders"&#10;# ✅&#10;operationId: "listOrders"</code></pre> |
| operationId describe la acción HTTP | ⚠️ Warning | Copilot | El verbo inicial del `operationId` **DEBERÍA** ser coherente con el método HTTP (ej. `GET` → `list/get/fetch`, `POST` → `create/submit`). | <pre><code class="language-yaml"># ❌&#10;get:&#10;  operationId: "createOrders"&#10;# ✅&#10;get:&#10;  operationId: "listOrders"</code></pre> |
| Claves JSON sin números al inicio | ⛔ Error | Spectral | Ninguna propiedad de schema **PUEDE** comenzar con un número. | <pre><code class="language-yaml"># ❌&#10;properties: { 1stField: { type: string } }&#10;# ✅&#10;properties: { firstField: { type: string } }</code></pre> |
| Nombres de arrays en plural | ⚠️ Warning | Copilot | Propiedades con `type: array` **DEBERÍAN** estar en plural (ej. `items`, `orders`). | <pre><code class="language-yaml"># ❌&#10;item:&#10;  type: array&#10;  items: { type: string }&#10;# ✅&#10;items:&#10;  type: array&#10;  items: { type: string }</code></pre> |
| Sufijo Code para campos de códigos | ⚠️ Warning | Copilot | Si el `description` indica que un campo es un "código/codigo/code", el nombre **DEBERÍA** terminar en `Code`. | <pre><code class="language-yaml"># ❌&#10;status:&#10;  type: string&#10;  description: "Código de estado"&#10;# ✅&#10;statusCode:&#10;  type: string&#10;  description: "Código de estado"</code></pre> |
| Evitar palabras reservadas en propiedades | ⚠️ Warning | Spectral | Evitar nombres de propiedades que colisionen con palabras reservadas de JavaScript ES2023: `message`, `body`, `payload`, `class`, `default`, `function`, `null`, `true`, `false`, `return`, `var`, `let`, `const`, `new`, `this`, `typeof`, `delete`, `void`, `instanceof`, `in`, `if`, `else`, `for`, `while`, `do`, `break`, `continue`, `switch`, `case`, `throw`, `try`, `catch`, `finally`, `import`, `export`, `extends`, `super`, `yield`, `async`, `await`, `static`, `with`, `debugger`, `eval`, `arguments`, `prototype`, `constructor`. | <pre><code class=\"language-yaml\"># ❌\nproperties: { payload: { type: string } }\n# ✅\nproperties: { payloadHash: { type: string } }</code></pre> |
| Claves sin caracteres Unicode no imprimibles | ⛔ Error | Spectral | Los nombres de propiedades solo deben contener caracteres ASCII imprimibles (U+0020–U+007E). Están prohibidos los caracteres de control (U+0000–U+001F, U+007F), invisibles y que rompen SDKs de generación de código. | <pre><code class=\"language-yaml\"># ❌ (carácter de control invisible)\npropName: { type: string }\n# ✅\npropName: { type: string }</code></pre> |


---

### Formato

| Regla | Severidad | Herramienta | Descripción |
|---|---:|:---:|---|
| Documento YAML/JSON parseable | ⛔ Error | Spectral | El documento **DEBE** ser parseable (YAML/JSON válido). |
| Indentación estándar | ⚠️ Warning | Copilot | YAML **DEBERÍA** usar 2 espacios y no tabs. |
| JSON: comillas dobles | ⚠️ Warning | Copilot | Si el contrato está en JSON, los strings **DEBEN** usar comillas dobles. Aplica solo a contratos `.json`. |

---

### Claridad

| Regla | Severidad | Herramienta | Descripción |
|---|---:|:---:|---|
| Descripción de la API presente | ⚠️ Warning | Spectral | `info.description` **DEBERÍA** explicar propósito y audiencia. |
| Operación con summary | ⚠️ Warning | Spectral | Operaciones **DEBERÍAN** tener `summary` no vacío. |
| Operación con description | ⚠️ Warning | Spectral | Operaciones **DEBERÍAN** tener `description` con contexto funcional del endpoint. |
| Datos personales mínimos (PII) | ⚠️ Warning | Copilot | Evitar campos PII (`rut`, `email`, `phone`, `address`, etc.) sin justificación. Si son inevitables, documentar política de enmascarado en `description`. |
| Códigos de estado coherentes con el método | ⚠️ Warning | Copilot | Los status codes deben ser coherentes con el método HTTP (ej. `GET` no devuelve `201 Created`). |
| GET para filtrar / POST para búsquedas complejas | ⚠️ Warning | Copilot | `GET` para filtros simples vía query params; `POST /.../searches` para búsquedas complejas con múltiples criterios en body. |
| Reuso vía $ref en componentes | ⚠️ Warning | Copilot | Reutilizar modelos en `components` cuando haya duplicación idéntica evidente (≥ 3 ubicaciones). |
| Evitar campos en desuso, vacíos o nulos | ⚠️ Warning | Copilot | No incluir campos con `deprecated: true`, `example: null`, `example: ""` o `description` con referencias a "en desuso"/"deprecated". |
| No duplicar IDs entre path y body | ⛔ Error | Copilot | Si un `{...Id}` del path aparece como `properties.<...Id>` en `requestBody` o `responses` del mismo endpoint, es un error, salvo justificación explícita en `description` (legacy/compat/backward/redundante). |
| Variables de path con semántica de dominio | ⚠️ Warning | Copilot | Las variables `{...}` del path **DEBERÍAN** ser semánticamente coherentes con el recurso que identifican (ej. `{orderId}` bajo `/orders`, no `{entityId}` genérico). | <pre><code class="language-yaml"># ❌&#10;/orders/{entityId}&#10;# ✅&#10;/orders/{orderId}</code></pre> |
| Servers URL estándar ARCHBX (escenarios 1/2/3/4a/4b) | ⛔ Error | Spectral | `servers[].url` **DEBE** ajustarse a ARCHBX. Implementado como **3 reglas Spectral separadas** por escenario: (1) Escenarios 1 y 2 — API Gateway (`api[.<env>].blue.cl`); (2) Escenario 3 — BFF (`int.api[.<env>].blue.cl/bff/`); (3) Escenarios 4a y 4b — DNS interno (`http://<svc>.<ns>/v<M>`) e Ingress externo (`https://<svc>.blueexpress.tech/v<M>`). Prohibido `/v1.2/` y el prefijo `/api/`. |
| Errores estandarizados RFC 9457 (Problem Details) | ⛔ Error | Spectral | Implementado como 4 reglas Spectral: (1) respuestas `4xx/5xx` usan `application/problem+json`; (2) componentes de respuesta de error usan `application/problem+json`; (3) existe `components.schemas.ProblemDetails`; (4) `ProblemDetails` define `required: [type, title, status, detail]`. |
| Booleanos no nulos | ⚠️ Warning | Spectral | Las propiedades `type: boolean` **NO DEBEN** ser `nullable: true` ni tener `default: null`. | <pre><code class="language-yaml"># ❌&#10;active:&#10;  type: boolean&#10;  nullable: true&#10;# ✅&#10;active:&#10;  type: boolean&#10;  description: "Indica si el recurso está activo."</code></pre> |
| Duplicación de datos entre campos | ⚠️ Warning | Copilot | Evitar propiedades en el mismo schema que contengan la misma información semántica bajo nombres distintos. Ante duda razonable, no reportar. | <pre><code class="language-yaml"># ❌&#10;properties:&#10;  date: { type: string }&#10;  creationDate: { type: string }&#10;# ✅&#10;properties:&#10;  createdAt: { type: string, format: date-time }</code></pre> |
| Tipo de dato de `example` consistente con `type` | ⛔ Error | Copilot | El valor de `example` debe ser compatible con el `type` declarado. `type: integer` requiere número sin comillas; `type: boolean` requiere `true`/`false` sin comillas; `type: array` requiere una lista. Un ejemplo mal tipado rompe generación de SDKs y mocks. | <pre><code class=\"language-yaml\"># ❌\nquantity:\n  type: integer\n  example: \"5\"\n# ✅\nquantity:\n  type: integer\n  example: 5</code></pre> |

---

## 7. Ejemplo Práctico (OpenAPI válido)

```yaml
openapi: "3.0.3"
info:
  title: "Orders API - Gestión de órdenes"
  version: "1.0.0"
  x-source-repo: "https://github.com/acme-org/orders-api"
  description: "API para crear, consultar y gestionar órdenes."
servers:
  - url: "https://api.dev.blue.cl/georf/geographical-data/v1"
    description: "Gateway no-prod"
paths:
  /orders:
    get:
      operationId: listOrders
      summary: "Listar órdenes"
      description: "Retorna una colección paginada de órdenes."
      parameters:
        - $ref: "#/components/parameters/TraceparentHeader"
        - $ref: "#/components/parameters/TracestateHeader"
      responses:
        "200":
          description: "OK"
          content:
            application/json:
              schema:
                type: object
                properties:
                  orders:
                    type: array
                    items:
                      $ref: "#/components/schemas/Order"
        "400":
          $ref: "#/components/responses/BadRequest"
components:
  parameters:
    TraceparentHeader:
      name: traceparent
      in: header
      required: true
      schema:
        type: string
    TracestateHeader:
      name: tracestate
      in: header
      required: false
      schema:
        type: string
  schemas:
    Order:
      type: object
      properties:
        orderId:
          type: string
        status:
          type: string
    ProblemDetails:
      type: object
      required: [type, title, status, detail]
      properties:
        type:
          type: string
        title:
          type: string
        status:
          type: integer
        detail:
          type: string
        instance:
          type: string
  responses:
    BadRequest:
      description: "Bad Request"
      content:
        application/problem+json:
          schema:
            $ref: "#/components/schemas/ProblemDetails"
```

---

## 8. Checklist para equipos

- [ ] `openapi: 3.0.x` (recomendado 3.0.3).
- [ ] `info.title` > 10 caracteres y descriptivo.
- [ ] `info.version` en SemVer (`MAJOR.MINOR.PATCH`).
- [ ] `info.x-source-repo` presente y URL absoluta.
- [ ] `info.description` presente (propósito/alcance/audiencia).
- [ ] `servers` presente, con `url` válida y patrón ARCHBX (escenarios 1/2/3/4).
- [ ] `paths` presente, recursos en minúsculas y kebab-case, sin verbos CRUD.
- [ ] Variables de path `{...}` en `lowerCamelCase` y declaradas como parámetros `in: path` requeridos.
- [ ] Todas las operaciones con `operationId` único.
- [ ] Todas las operaciones documentan `traceparent` (y `tracestate` recomendado).
- [ ] Respuestas 4xx/5xx usan `application/problem+json` y `ProblemDetails` (RFC 9457).
- [ ] Componentes reutilizables centralizados en `components.*` y referenciados con `$ref`.
- [ ] Definidos `securitySchemes` + `security` (o `security: []` si la operación es pública).
