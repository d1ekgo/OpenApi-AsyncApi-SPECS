function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    var doc = SpreadsheetApp.getActiveSpreadsheet();
    // Asegúrate de que este nombre coincida con tu hoja ('Metricas' o 'Hoja 1')
    var sheet = doc.getSheetByName('Metrics') || doc.getActiveSheet();

    var rawData = e.postData.contents;
    var jsonData = JSON.parse(rawData);

    var newRow = [
      new Date(),                         // Columna A
      jsonData.repo,                      // Columna B
      jsonData.pr_number,                 // Columna C
      jsonData.pr_author,                 // Columna D
      jsonData.api_type,                  // Columna E (Filtro clave)
      jsonData.spectral_errors,           // Columna F
      jsonData.spectral_warnings,         // Columna G
      jsonData.copilot_errors,            // Columna H
      jsonData.copilot_warnings,          // Columna I
      jsonData.total_errors,              // Columna J
      jsonData.total_warnings,            // Columna K
      jsonData.status,                    // Columna L
      jsonData.spectral_rules_list,       // Columna M (NUEVO: Reglas Spectral)
      jsonData.copilot_rules_list         // Columna N (NUEVO: Reglas Copilot)
    ];

    sheet.appendRow(newRow);

    return ContentService.createTextOutput(JSON.stringify({ "result": "success", "row": newRow }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService.createTextOutput(JSON.stringify({ "result": "error", "message": error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);

  } finally {
    lock.releaseLock();
  }
}
