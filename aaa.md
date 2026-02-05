function doPost(e) {
  // # Bloqueo para evitar que dos requests escriban a la vez (concurrencia)
  var lock = LockService.getScriptLock();
  lock.tryLock(10000); // # Espera hasta 10s por el lock

  try {
    // # Abre el Spreadsheet donde está instalado este Apps Script
    var doc = SpreadsheetApp.getActiveSpreadsheet();

    // # Busca la hoja llamada "Metrics"; si no existe, usa la hoja activa
    // # (Esto es donde se guardarán las filas enviadas desde GitHub Actions)
    var sheet = doc.getSheetByName('Metrics') || doc.getActiveSheet();

    // # Lee el cuerpo crudo del POST recibido por el Web App (viene desde tu workflow)
    // # En tu workflow lo mandas con curl -d '{ ... }'
    var rawData = e.postData.contents;

    // # Convierte el JSON string en objeto JavaScript
    var jsonData = JSON.parse(rawData);

    // # Construye una nueva fila con el orden exacto de columnas que quieres en Google Sheets
    // # Cada campo jsonData.<campo> corresponde a una clave enviada desde tu GitHub workflow
    var newRow = [
      new Date(),                         // Columna A: timestamp local (cuando llegó el request)

      // # Datos del PR (enviados desde GitHub Actions)
      jsonData.repo,                      // Columna B: repo (ej: "owner/repo")
      jsonData.pr_number,                 // Columna C: número de PR
      jsonData.pr_author,                 // Columna D: autor del PR

      // # Tipo de especificación detectada (openapi o asyncapi)
      jsonData.api_type,                  // Columna E: api_type (sirve para filtros/segmentación)

      // # Métricas de Spectral (enviadas desde el step "Run Spectral")
      jsonData.spectral_errors,           // Columna F: cantidad de errores Spectral
      jsonData.spectral_warnings,         // Columna G: cantidad de warnings Spectral

      // # Métricas de Copilot (enviadas desde el step "Run Copilot semantic audit")
      jsonData.copilot_errors,            // Columna H: cantidad de errores Copilot
      jsonData.copilot_warnings,          // Columna I: cantidad de warnings Copilot

      // # Totales calculados en el workflow (Spectral + Copilot)
      jsonData.total_errors,              // Columna J: total_errors
      jsonData.total_warnings,            // Columna K: total_warnings

      // # Estado final del workflow (PASS / FAIL)
      jsonData.status,                    // Columna L: status

      // # Lista de reglas disparadas (para análisis posterior)
      jsonData.spectral_rules_list,       // Columna M: reglas Spectral separadas por comas
      jsonData.copilot_rules_list         // Columna N: reglas Copilot separadas por comas
    ];

    // # Inserta la fila al final de la hoja
    sheet.appendRow(newRow);

    // # Respuesta OK (200) en formato JSON
    return ContentService
      .createTextOutput(JSON.stringify({ "result": "success", "row": newRow }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    // # Si algo falla (JSON malformado, hoja no existe, etc.), responde con error
    return ContentService
      .createTextOutput(JSON.stringify({ "result": "error", "message": error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);

  } finally {
    // # Libera el lock pase lo que pase
    lock.releaseLock();
  }
}
