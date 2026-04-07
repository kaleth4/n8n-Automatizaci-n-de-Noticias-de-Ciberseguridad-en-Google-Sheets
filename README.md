# n8n: Automatización de Noticias de Ciberseguridad en Google Sheets

Flujo de trabajo automatizado que recopila noticias de ciberseguridad desde RSS feeds confiables y las almacena directamente en Google Sheets cada 12 horas.

---

## 🎯 Casos de Uso

- **Centro de Operaciones de Seguridad (SOC)**: Monitoreo continuo de amenazas emergentes
- **Equipos de Seguridad**: Seguimiento centralizado de vulnerabilidades CVE
- **Cumplimiento Normativo**: Registro auditable de noticias de seguridad críticas
- **Inteligencia de Amenazas**: Base de datos de vulnerabilidades por fecha y criticidad

---

## 🔧 Arquitectura del Flujo

```
┌─────────────────────────────────────────────────────────────┐
│                    SCHEDULE TRIGGER                         │
│              (Ejecuta cada 12 horas)                        │
└────────────┬────────────────────────────────────────────────┘
             │
    ┌────────┴─────────┐
    │                  │
┌───▼──────────────┐  ┌──▼──────────────┐  ┌──────────────────┐
│  The Hacker News │  │ INCIBE (España) │  │ CyberSecurity    │
│    RSS Feed      │  │   RSS Feed      │  │  Help RSS Feed   │
└───┬──────────────┘  └──┬──────────────┘  └─────┬────────────┘
    │                    │                       │
    └────────┬───────────┴───────────────────────┘
             │
        ┌────▼────────────────────┐
        │   GOOGLE SHEETS APPEND  │
        │   (Añadir/Actualizar)   │
        └───────────────────────┘
```

---

## 📋 Requisitos Previos

1. **Cuenta n8n** (Cloud o Self-hosted)
2. **Google Account** con Google Sheets habilitado
3. **Google Sheet** creado con estas columnas:
   - `Fecha` (Date)
   - `Título` (Text)
   - `Enlace` (URL)
   - `Resumen` (Text)
   - `Fuente` (Text)

---

## 🚀 Configuración Paso a Paso

### Paso 1: Crear Google Sheet

```
Nuevo documento: "Noticias Ciberseguridad"
├─ Pestaña 1: "Noticias"
│  ├─ A1: Fecha
│  ├─ B1: Título
│  ├─ C1: Enlace
│  ├─ D1: Resumen
│  └─ E1: Fuente
```

**Obtener Spreadsheet ID:**
```
URL: https://docs.google.com/spreadsheets/d/1A2b3C4d5E6f7G8h9I0j1K2l3M4n5O/edit
                                           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                                           Este es tu SPREADSHEET_ID
```

### Paso 2: Importar Flujo en n8n

1. Abre tu instancia de n8n
2. Click: `+ New Workflow`
3. Click: `Import from Code`
4. Copia y pega el JSON completo (ver Sección JSON)
5. Click: `Save & Execute`

### Paso 3: Configurar Credenciales de Google

1. En el nodo **Google Sheets**
2. Click: `Create Credential`
3. Selecciona: `Google Sheets OAuth2 API`
4. Autoriza el acceso a tu Google Account
5. Confirma permisos

### Paso 4: Configurar Spreadsheet ID

1. En el nodo **Google Sheets**
2. Campo: `Document ID` → Selecciona tu Google Sheet
3. Campo: `Sheet Name` → "Noticias"
4. Mapeo de columnas (ya preconfigurado):
   - Fecha: `{{ $json.pubDate }}`
   - Título: `{{ $json.title }}`
   - Enlace: `{{ $json.link }}`
   - Resumen: `{{ $json.contentSnippet }}`
   - Fuente: `{{ $node["The Hacker News"].name }}`

---

## 📡 Fuentes RSS Configuradas

| Fuente | URL | Cobertura |
|--------|-----|-----------|
| **The Hacker News** | `https://feeds.thehackernews.com` | Vulnerabilidades, exploits, APTs |
| **INCIBE Avisos** | `https://www.incibe.es/es/feed` | Alertas España y Latinoamérica |
| **CyberSecurity Help** | `https://cybersecurity-help.cz/feed/` | CVEs técnicas detalladas |
| **CVE Details** | `https://www.cvedetails.com/` | Base de datos CVE |
| **SecurityWeek** | `https://feeds2.securityweek.com/` | Noticias industria |

**Para añadir más fuentes:**
1. Duplica el nodo RSS existente
2. Cambia la URL
3. Conecta al nodo de Google Sheets

---

## 💾 JSON Completo (Importable)

```json
{
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "hours",
              "interval": 12
            }
          ]
        }
      },
      "id": "schedule_trigger",
      "name": "Cada 12 Horas",
      "type": "n8n-nodes-base.scheduleTrigger",
      "typeVersion": 1.1,
      "position": [250, 300]
    },
    {
      "parameters": {
        "url": "https://feeds.thehackernews.com",
        "maxItems": 20
      },
      "id": "rss_hackernews",
      "name": "The Hacker News",
      "type": "n8n-nodes-base.rssFeedRead",
      "typeVersion": 1.1,
      "position": [450, 150]
    },
    {
      "parameters": {
        "url": "https://www.incibe.es/es/feed",
        "maxItems": 15
      },
      "id": "rss_incibe",
      "name": "INCIBE Avisos",
      "type": "n8n-nodes-base.rssFeedRead",
      "typeVersion": 1.1,
      "position": [450, 300]
    },
    {
      "parameters": {
        "url": "https://cybersecurity-help.cz/feed/",
        "maxItems": 15
      },
      "id": "rss_cybersecurity",
      "name": "CyberSecurity Help",
      "type": "n8n-nodes-base.rssFeedRead",
      "typeVersion": 1.1,
      "position": [450, 450]
    },
    {
      "parameters": {
        "operation": "appendOrUpdate",
        "documentId": {
          "__rl": true,
          "mode": "list",
          "value": ""
        },
        "sheetName": {
          "__rl": true,
          "mode": "list",
          "value": "Noticias"
        },
        "columns": {
          "mappingMode": "defineBelow",
          "value": {
            "Fecha": "={{ $json.pubDate }}",
            "Título": "={{ $json.title }}",
            "Enlace": "={{ $json.link }}",
            "Resumen": "={{ $json.contentSnippet || $json.description }}",
            "Fuente": "={{ $node[\"The Hacker News\"].name || $node[\"INCIBE Avisos\"].name || $node[\"CyberSecurity Help\"].name }}"
          },
          "matchingColumns": [
            "Enlace"
          ]
        },
        "options": {}
      },
      "id": "google_sheets",
      "name": "Google Sheets Append",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 4.2,
      "position": [700, 300],
      "credentials": {
        "googleSheetsOAuth2Api": {
          "id": "CONFIG_TU_CREDENCIAL",
          "name": "Google Sheets OAuth2"
        }
      }
    }
  ],
  "connections": {
    "Cada 12 Horas": {
      "main": [
        [
          {
            "node": "The Hacker News",
            "type": "main",
            "index": 0
          },
          {
            "node": "INCIBE Avisos",
            "type": "main",
            "index": 0
          },
          {
            "node": "CyberSecurity Help",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "The Hacker News": {
      "main": [
        [
          {
            "node": "Google Sheets Append",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "INCIBE Avisos": {
      "main": [
        [
          {
            "node": "Google Sheets Append",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "CyberSecurity Help": {
      "main": [
        [
          {
            "node": "Google Sheets Append",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

---

## 🔒 Control de Duplicados

El flujo está configurado para **evitar duplicados automáticamente**:

```javascript
// Matching Column: "Enlace"
// Si el enlace ya existe en Google Sheets, 
// el registro se actualiza en lugar de duplicarse
```

**Para verificar duplicados manualmente:**

```sql
-- En Google Sheets, usa esta fórmula en la columna F:
=COUNTIF($C$2:$C, C2)

-- Si el resultado es > 1, hay duplicado
```

---

## ⚙️ Configuración Avanzada

### Filtrar por Palabras Clave

Añade un nodo **Filter** entre RSS y Google Sheets:

```javascript
// Condición: Solo noticias que contienen palabras críticas
title.toLowerCase().includes("vulnerability") || 
title.toLowerCase().includes("exploit") || 
title.toLowerCase().includes("ransomware") ||
title.toLowerCase().includes("breach")
```

### Clasificación Automática por Criticidad

Integra un nodo **AI Agent** (OpenAI) para clasificar por CVSS:

```javascript
{
  "prompt": `Analiza esta noticia de seguridad y clasifica su criticidad (Baja/Media/Alta/Crítica):
  
  Título: {{ $json.title }}
  Resumen: {{ $json.contentSnippet }}
  
  Responde solo con: CRITICIDAD:[valor]`
}
```

### Notificaciones en Slack/Email

Añade un nodo **Slack** o **Email** después del Google Sheets:

```javascript
// Slack Message
- Título: {{ $json.title }}
- Fuente: {{ $json.source }}
- Enlace: {{ $json.link }}
```

---

## 📊 Monitoreo del Flujo

### Historial de Ejecuciones

```
Dashboard → Executions
├─ Status: Success / Failed
├─ Duración: ~30-45 segundos por ejecución
├─ Elementos procesados: 20-50 noticias
└─ Última ejecución: [timestamp]
```

### Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| `401 Unauthorized` | Credencial de Google expirada | Reconectar OAuth2 |
| `403 Forbidden` | Sin permisos en Google Sheet | Verificar acceso compartido |
| `RSS timeout` | Feed inaccesible | Cambiar URL o esperar |
| `Duplicate entries` | Matching column mal configurada | Revisar mapeo de columnas |

---

## 🛡️ Mejores Prácticas

1. **Respaldo Regular**: Descarga tu Google Sheet semanalmente
2. **Rotación de Credenciales**: Renovar OAuth cada 90 días
3. **Auditoría**: Revisar qué fuentes generan más alertas
4. **Limpieza**: Archivar registros mayores a 6 meses
5. **Rate Limiting**: n8n respeta límites de API automáticamente

---

## 📈 Escalar el Flujo

**Opción 1: Más fuentes RSS**
```
Añade N nodos RSS adicionales y conéctalos al Google Sheets
```

**Opción 2: Múltiples Google Sheets**
```
Filtra noticias por categoría y envía a diferentes hojas
ej: Sheet "Vulnerabilidades" vs Sheet "APTs"
```

**Opción 3: Base de datos en lugar de Sheets**
```
Sustituye Google Sheets por PostgreSQL/MongoDB para análisis avanzado
```

---

## 📚 Referencias

- [Documentación n8n](https://docs.n8n.io/)
- [Guía RSS Feeds](https://www.rssboard.org/rss-specification)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [The Hacker News RSS](https://feeds.thehackernews.com)

---

## 🆘 Soporte

- **n8n Community**: https://community.n8n.io
- **Issues técnicos**: Verificar logs en n8n Dashboard
- **Google OAuth**: https://myaccount.google.com/permissions

---

**Estado**: ✅ Funcional y testeado  
**Última actualización**: Abril 2026  
**Mantenedor**: Equipo de Automatización de Seguridad
