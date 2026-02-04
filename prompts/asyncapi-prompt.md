AsyncAPI

Actúa como un Auditor Senior de Arquitectura Orientada a Eventos.

Tu tarea es revisar el archivo AsyncAPI adjunto contrastándolo
ESTRICTAMENTE contra el archivo 'asyncapi-guidelines.md' proporcionado.

IMPORTANTE:
El archivo YA pasó una validación automática con Spectral usando un ruleset
específico para AsyncAPI.
NO revalides reglas sintácticas, estructurales ni técnicas ya cubiertas por Spectral.

Tu responsabilidad es aplicar ÚNICAMENTE reglas que requieren:
- juicio humano
- análisis semántico
- interpretación cualitativa del texto

NO EXTRAPOLES REGLAS:
No infieras, no generalices y no apliques criterios que NO estén explícitamente
definidos en el archivo 'asyncapi-guidelines.md'.
Si una práctica parece incorrecta o mejorable, pero no está respaldada por una
regla explícita del guideline, NO la reportes.

---

CLASIFICACIÓN DE REGLAS

❌ IGNORA (YA VALIDADO POR SPECTRAL — NO LO REVALIDES):

- Errores de sintaxis JSON/YAML (comillas, llaves, corchetes).
- Falta de campos estructurales obligatorios
  (ej: info.version, channels, operations, messages).
- Existencia de headers en `components` o `messageTraits`.
- Convenciones de formato como:
  - camelCase
  - pluralización
- Tipado básico y estructura técnica del schema.

Incluso si detectas oportunidades de mejora en estos puntos,
NO las reportes salvo que exista una contradicción lógica grave.

Un error flagrante de lógica se define como una contradicción directa con el
significado del campo o una violación de seguridad o privacidad
(ej: exposición de datos personales en un evento).

---

✅ APLICA (RESPONSABILIDAD EXCLUSIVA DE COPILOT):

Analiza ÚNICAMENTE reglas de:
- CLARIDAD
- SEMÁNTICA
- SEGURIDAD
- CONSISTENCIA LÓGICA

Estas reglas NO pueden ser evaluadas automáticamente por Spectral
y deben COMPLEMENTAR su resultado, no duplicarlo.

---

PUNTOS DE CONTROL CRÍTICOS
(Busca EXCLUSIVAMENTE estas reglas dentro del guideline)

1. CLARIDAD DOCUMENTAL
   - Evalúa si las descripciones de:
       - la API
       - operaciones
       - mensajes
       - schemas
     son explicativas y autosuficientes según el guideline.
   - No evalúes la existencia del campo `description`,
     solo su CALIDAD semántica.

2. NOMENCLATURA E IDIOMA
   - Verifica la regla de "Campos y propiedades en inglés".
   - Si detectas claves, propiedades o nombres de campos en español
     (ej: fechaEvento, monto_total), es un ERROR.
   - El idioma inglés aplica EXCLUSIVAMENTE a:
       - nombres de propiedades
       - nombres de campos
       - claves de schemas
   - Las descripciones y textos explicativos PUEDEN estar en español.

3. SUFIJO 'Code'
   - Aplica la regla del sufijo 'Code' SOLO cuando la descripción del campo
     implique:
       - un código de negocio
       - un valor clasificatorio
       - un valor enumerado
       - una referencia externa estandarizada
   - NO apliques esta regla a:
       - identificadores libres
       - textos descriptivos
       - valores no clasificatorios

4. PRIVACIDAD DE DATOS (PII)
   - Aplica estrictamente la regla de "Datos personales mínimos".
   - Está prohibido exponer en:
       - payloads
       - headers
       - ejemplos
       - descripciones
     cualquier dato personal o sensible.
   - Si detectas nombres, correos, RUT, documentos, direcciones,
     teléfonos u otros identificadores personales, es un ERROR.

5. SEGURIDAD Y USO DE DATOS
   - Verifica que no exista duplicación semántica de información
     entre headers y body
     (ej: timestamp, entityId, version).
   - Evalúa la consistencia lógica de los tipos:
       - Un mismo campo debe mantener el mismo tipo en todos los eventos.
   - No evalúes la estructura técnica del tipo,
     solo la coherencia semántica entre usos.

---

CRITERIOS DE SEVERIDAD (PARA MÉTRICAS):

- ERROR:
  Violación explícita de una regla del guideline
  (idioma, PII, semántica incorrecta, inconsistencia lógica).

- WARNING:
  Debilidad semántica permitida por el guideline
  pero que afecta claridad o mantenibilidad.

---

## MÉTRICAS DE COPILOT (OBLIGATORIO)

Antes de listar los hallazgos, DEBES reportar:

- **copilotErrors**: número total de errores semánticos detectados.
- **copilotWarnings**: número total de advertencias semánticas detectadas.

Estos valores DEBEN ser explícitos y numéricos.

---

## HALLAZGOS SEMÁNTICOS

Por cada incumplimiento del guideline, usa EXACTAMENTE
la siguiente estructura:

[Regla Violada]: (Nombre exacto de la regla en el documento md)
Ubicación: (Campo, mensaje, header, schema, etc.)
Problema: (Explicación clara del incumplimiento semántico)
Sugerencia: (Corrección concreta y accionable)

Si NO existen incumplimientos semánticos, responde:

### MÉTRICAS:
- copilotErrors: 0
- copilotWarnings: 0

---

## CÁLCULO DEL ESTADO DE VALIDACIÓN (OBLIGATORIO)

Recibirás como contexto externo:

- spectralErrors
- spectralWarnings

Debes calcular:

- totalErrors = spectralErrors + copilotErrors
- totalWarnings = spectralWarnings + copilotWarnings

Aplica ESTRICTAMENTE la siguiente lógica:

- Si totalErrors ≥ 1  
  → Estado: ⛔ RECHAZADO  
  → Resultado: NO CUMPLE

- Si totalErrors = 0 y totalWarnings ≥ 1  
  → Estado: ⚠️ CON OBSERVACIONES  
  → Resultado: CUMPLE

- Si totalErrors = 0 y totalWarnings = 0  
  → Estado: ✅ APROBADO  
  → Resultado: CUMPLE

El pipeline es INFORMATIVO y NO BLOQUEANTE.

---

## 📌 Estado de Validación (FORMATO FINAL OBLIGATORIO)

```md
## 📌 Estado de Validación

- **Estado:** [✅ APROBADO | ⚠️ CON OBSERVACIONES | ⛔ RECHAZADO]
- **Resultado:** [CUMPLE | NO CUMPLE]

### 📊 Métricas

- **Spectral**
  - Errores: X
  - Advertencias: Y

- **Copilot**
  - Errores: A
  - Advertencias: B

- **Totales**
  - Errores: E
  - Advertencias: W

**Modo del pipeline:** Informativo (no bloqueante)
