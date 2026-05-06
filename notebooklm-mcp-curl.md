# NotebookLM MCP - Comandos curl

Este documento lista todos los comandos disponibles para usar el MCP de NotebookLM via HTTP.

---

## Iniciar el servidor

```bash
cd D:\.mcp\notebooklm-mcp
npm run start:http
```

El servidor escucha en `http://127.0.0.1:3000`

---

## 1. Health - Verificar estado

Verifica si el servidor está corriendo y si tenés login hecho.

```bash
curl http://127.0.0.1:3000/health
```

**Respuesta esperada:**

```json
{"success":true,"data":{"status":"ok","authenticated":true,...}}
```

**PowerShell:**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/health -Method GET
```

**PowerShell (completo):**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/health -Method GET | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- `success`: true/false
- `data.status`: "ok" o "error"
- `data.authenticated`: true si tenés login hecho
- `data.active_sessions`: cantidad de sesiones abiertas

---

## 2. Listar notebooks

Lista todos los notebooks que tenés agregados en la biblioteca.

```bash
curl http://127.0.0.1:3000/notebooks
```

**PowerShell:**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/notebooks -Method GET
```

**PowerShell (completo):**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/notebooks -Method GET | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- Array de notebooks con: id, url, name, description, topics, content_types, use_cases, added_at, last_used, use_count, tags

Ejemplo de respuesta:

```json
{
  "success": true,
  "data": {
    "notebooks": [
      {
        "id": "ai-architect",
        "url": "https://notebooklm.google.com/notebook/...",
        "name": "ai-architect",
        "description": "Arquitectura de sistemas de AI...",
        "topics": ["ai architecture", "mlops", ...],
        ...
      }
    ]
  }
}
```

---

## 3. Seleccionar notebook

Hace que un notebook sea el activo para preguntar.

```bash
curl -X PUT http://127.0.0.1:3000/notebooks/{id}/activate
```

Ejemplo:

```bash
curl -X PUT http://127.0.0.1:3000/notebooks/ai-architect/activate
```

**PowerShell:**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/notebooks/ai-architect/activate -Method PUT
```

**PowerShell (completo):**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/notebooks/ai-architect/activate -Method PUT | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- `success`: true
- `data.notebook`: el notebook seleccionado

---

## 4. Obtener un notebook

Sacá información de un notebook específico.

```bash
curl http://127.0.0.1:3000/notebooks/{id}
```

Ejemplo:

```bash
curl http://127.0.0.1:3000/notebooks/ai-architect
```

**PowerShell:**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/notebooks/ai-architect -Method GET
```

**PowerShell (completo):**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/notebooks/ai-architect -Method GET | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- El objeto completo del notebook (mismos campos que en la lista)

---

## 5. Chatear con notebook

**El principal - para hacer preguntas.**

```bash
curl -X POST http://127.0.0.1:3000/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "TU PREGUNTA", "session_id": "nombre-opcional"}'
```

Ejemplo:

```bash
curl -X POST http://127.0.0.1:3000/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "Qué sabes sobre vector databases?", "session_id": "test"}'
```

**Parámetros:**

- `question` (required): La pregunta en texto
- `notebook_id` (optional): ID del notebook a usar
- `notebook_url` (optional): URL directa del notebook
- `session_id` (optional): Si no lo pasás, crea uno nuevo. Si repetís el mismo, sigue la conversación.
- `show_browser` (optional): true/false para mostrar el navegador
- `source_format` (optional): Formato de citación (none, inline, footnotes, json, expanded)

**Alternativa con show_browser:**

```bash
curl -X POST http://127.0.0.1:3000/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "...", "show_browser": true}'
```

**PowerShell:**

```powershell
$body = @{question="TU PREGUNTA"; session_id="nombre-opcional"} | ConvertTo-Json
Invoke-WebRequest http://127.0.0.1:3000/ask -Method POST -Body $body -ContentType "application/json"
```

**PowerShell (completo):**

```powershell
$body = @{question="TU PREGUNTA"; session_id="nombre-opcional"} | ConvertTo-Json
Invoke-WebRequest http://127.0.0.1:3000/ask -Method POST -Body $body -ContentType "application/json" | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- La respuesta del notebook (texto con fuentes citing documentos)
- Estados de la sesión

---

## 6. Listar sesiones

Ver las sesiones activas (para seguir usando la misma o cerrarlas).

```bash
curl http://127.0.0.1:3000/sessions
```

**PowerShell:**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/sessions -Method GET
```

**PowerShell (completo):**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/sessions -Method GET | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- Array de sesiones activas con: session_id, notebook_id, created_at, last_activity

---

## 7. Cerrar sesión

Cerrá una sesión específica.

```bash
curl -X DELETE http://127.0.0.1:3000/sessions/{session_id}
```

Ejemplo:

```bash
curl -X DELETE http://127.0.0.1:3000/sessions/test
```

**PowerShell:**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/sessions/test -Method DELETE
```

**PowerShell (completo):**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/sessions/test -Method DELETE | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- `success`: true
- `data.closed`: true

---

## 8. Setup auth

Abre el navegador para hacer login en Google/NotebookLM (si no está autenticado).

```bash
curl -X POST http://127.0.0.1:3000/setup-auth \
  -H "Content-Type: application/json" \
  -d '{"show_browser": true}'
```

**PowerShell:**

```powershell
$body = @{show_browser=$true} | ConvertTo-Json
Invoke-WebRequest http://127.0.0.1:3000/setup-auth -Method POST -Body $body -ContentType "application/json"
```

**PowerShell (completo):**

```powershell
$body = @{show_browser=$true} | ConvertTo-Json
Invoke-WebRequest http://127.0.0.1:3000/setup-auth -Method POST -Body $body -ContentType "application/json" | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- `success`: true
- `data.authenticated`: true
- `data.duration_seconds`: tiempo que tardó

### De-auth - Cerrar sesión

Cierra la sesión actual manteniendo la biblioteca.

```bash
curl -X POST http://127.0.0.1:3000/de-auth
```

**PowerShell:**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/de-auth -Method POST
```

**PowerShell (completo):**

```powershell
Invoke-WebRequest http://127.0.0.1:3000/de-auth -Method POST | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

### Re-auth - Re-autenticar

Cambiar de cuenta o re-autenticar después de cerrar sesión.

```bash
curl -X POST http://127.0.0.1:3000/re-auth \
  -H "Content-Type: application/json" \
  -d '{"show_browser": true}'
```

**PowerShell:**

```powershell
$body = @{show_browser=$true} | ConvertTo-Json
Invoke-WebRequest http://127.0.0.1:3000/re-auth -Method POST -Body $body -ContentType "application/json"
```

**PowerShell (completo):**

```powershell
$body = @{show_browser=$true} | ConvertTo-Json
Invoke-WebRequest http://127.0.0.1:3000/re-auth -Method POST -Body $body -ContentType "application/json" | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

### Cleanup Data - Limpiar datos

Limpia todos los datos del MCP. Requiere parámetro `confirm`.

```bash
# Preview (sin ejecutar)
curl -X POST http://127.0.0.1:3000/cleanup-data \
  -H "Content-Type: application/json" \
  -d '{"confirm": false, "preserve_library": true}'

# Ejecutar limpieza
curl -X POST http://127.0.0.1:3000/cleanup-data \
  -H "Content-Type: application/json" \
  -d '{"confirm": true, "preserve_library": true}'
```

**PowerShell:**

```powershell
$body = @{confirm=$false; preserve_library=$true} | ConvertTo-Json
Invoke-WebRequest http://127.0.0.1:3000/cleanup-data -Method POST -Body $body -ContentType "application/json"
```

**PowerShell (completo):**

```powershell
$body = @{confirm=$true; preserve_library=$true} | ConvertTo-Json
Invoke-WebRequest http://127.0.0.1:3000/cleanup-data -Method POST -Body $body -ContentType "application/json" | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- `success`: true/false
- `data.files_deleted`: cantidad de archivos eliminados
- `data.space_freed_mb`: espacio liberado en MB

---

## 10. Buscar notebooks

Busca notebooks por nombre/descripción/topics.

```bash
curl "http://127.0.0.1:3000/notebooks/search?query=busqueda"
```

**PowerShell:**

```powershell
Invoke-WebRequest "http://127.0.0.1:3000/notebooks/search?query=busqueda" -Method GET
```

**PowerShell (completo):**

```powershell
Invoke-WebRequest "http://127.0.0.1:3000/notebooks/search?query=busqueda" -Method GET | Select-Object -ExpandProperty Content | Out-File -FilePath result.json -Encoding utf8
Get-Content result.json -Raw
```

**Qué devuelve:**

- Array de notebooks que matchean la query (busca en name, description, topics, tags)

---

## Ejemplo completo de uso

```bash
# 1. Arrancar servidor
cd D:\.mcp\notebooklm-mcp && npm run start:http

# 2. Verificar estado
curl http://127.0.0.1:3000/health

# 3. Listar notebooks
curl http://127.0.0.1:3000/notebooks

# 4. Seleccionar uno (activar)
curl -X PUT http://127.0.0.1:3000/notebooks/ai-architect/activate

# 5. Preguntar
curl -X POST http://127.0.0.1:3000/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "Qué sabes sobre pgvector?"}'
```

---

## Notes

- Todos los endpoints retornan JSON
- El servidor debe estar corriendo primero
- Default port: 3000
- Para cambiar el port, editá `D:\.mcp\notebooklm-mcp\.env` y poner `HTTP_PORT=4000`
- En Windows, si curl no funciona, usá PowerShell
- **Las versiones "(completo)" guardan en archivo o muestran todo el JSON**
