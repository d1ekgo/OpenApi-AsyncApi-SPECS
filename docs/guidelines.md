# 📜 Guías de Diseño de API Corporativas (OAS & AAS)

Este documento define las reglas estrictas que deben cumplir todas las especificaciones de API (OpenAPI 3.x) y Eventos (AsyncAPI 2.x/3.x) dentro de la organización.

## 1. 🌐 Reglas Generales (Aplican a TODO)

### G-001: Versionado Semántico
- El campo `info.version` debe seguir estrictamente el formato **Semantic Versioning** (X.Y.Z).
- Ejemplo válido: `1.0.0`, `2.1.5`.
- Ejemplo inválido: `v1`, `1.0`.

### G-002: Contacto Obligatorio
- El objeto `info.contact` es obligatorio.
- Debe incluir `email` y este debe pertenecer al dominio corporativo `@miempresa.com`.

### G-003: Naming Conventions (Casing)
- **Schemas/Modelos:** Deben usar **PascalCase** (Ej: `CustomerAddress`, `OrderResponse`).
- **Propiedades/Campos:** Deben usar **camelCase** (Ej: `firstName`, `orderId`).

---

## 2. 🔌 Reglas para OpenAPI (REST)

### REST-001: URLs Limpias (Resource Oriented)
- Los `paths` (endpoints) deben ser **sustantivos** en plural y en **kebab-case**.
- ❌ Prohibido usar verbos en la URL.
- **Incorrecto:** `/getUsers`, `/save-client`, `/books/create`
- **Correcto:** `/users`, `/clients`, `/books`

### REST-002: Métodos HTTP Correctos
- Usar `GET` para leer, `POST` para crear, `PUT/PATCH` para actualizar, `DELETE` para borrar.
- ❌ Prohibido usar `POST` para obtener datos (search excluded).

### REST-003: Respuestas de Error
- Cada operación debe definir explícitamente al menos una respuesta de error (400, 401, 403, 404 o 500).
- No se permite definir únicamente la respuesta `200 OK`.

### REST-004: Identificadores de Operación
- Cada path debe tener un `operationId` único y descriptivo en **camelCase**.
- Ejemplo: `getUserById`, `createOrder`.

---

## 3. 📨 Reglas para AsyncAPI (Eventos/Mensajería)

### EVENT-001: Estructura de Canales (Topics)
- Los nombres de los canales (`channels`) deben seguir la jerarquía: `dominio.entidad.evento`.
- Deben estar en **kebab-case**.
- Ejemplo: `pagos.factura.creada`, `logistica.envio.actualizado`.

### EVENT-002: Documentación de Mensajes
- Cada `message` debe tener un campo `summary` y `description` explicando qué dispara el evento.

### EVENT-003: Trazabilidad (Headers)
- Todos los mensajes deben incluir una definición de `headers` que contenga obligatoriamente:
  - `correlationId` (tipo string, uuid).
  - `timestamp` (tipo string, date-time).

---

## 4. 🔒 Seguridad y Tipos de Datos

### SEC-001: Definición de Seguridad
- El archivo debe contener el objeto `components.securitySchemes`.
- No se permiten endpoints públicos sin seguridad definida (excepto `/health` o `/status`).

### DATA-001: Formato de Fechas
- Nunca usar `string` genérico para fechas.
- Usar siempre:
  - `type: string, format: date` (para YYYY-MM-DD).
  - `type: string, format: date-time` (para ISO 8601).

### DATA-002: Listas Paginadas (Solo OpenAPI)
- Si un endpoint retorna un array de objetos (`type: array`), la respuesta debe incluir metadatos de paginación (`page`, `limit`, `total`).
