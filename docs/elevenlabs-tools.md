# ElevenLabs Voice Agent Tools – Komplettreferenz

> Tool-Konfiguration, API-Schemas und n8n-Integration für ElevenLabs Conversational AI

---

## Tool-Typen Übersicht

| Typ | Ausführung | Verwendung | API-Config |
|-----|-----------|------------|------------|
| **webhook** | Server-seitig via HTTP | DB-Abfragen, Buchungen, CRM-Updates | `api_schema` mit URL, Method, Body |
| **client** | Browser/App-seitig | UI-Updates, Navigation, lokale Aktionen | `parameters` |
| **system** | Built-in | `end_call`, `transfer_to_number`, `transfer_to_agent`, `language_detection`, `skip_turn`, `voicemail_detection` | Minimal |
| **mcp** | MCP Server | Zugriff auf externe MCP-Tools | MCP-Config |

---

## 1. Webhook Tools (Server Tools)

### Vollständiges JSON-Schema

```json
{
  "type": "webhook",
  "name": "tool_name_ohne_leerzeichen",
  "description": "Wann und warum dieses Tool verwendet werden soll. KRITISCH für korrekte Nutzung!",
  "response_timeout_secs": 15,
  "disable_interruptions": false,
  "force_pre_tool_speech": false,
  "execution_mode": "immediate",
  "tool_call_sound": "typing",
  "tool_call_sound_behavior": "auto",
  "tool_error_handling_mode": "summarized",
  "assignments": [
    {
      "variable_name": "extracted_value",
      "description": "Beschreibung was aus dem Response extrahiert wird"
    }
  ],
  "api_schema": {
    "url": "https://n8n.example.com/webhook/tool-endpoint",
    "method": "POST",
    "content_type": "application/json",
    "request_headers": {
      "X-API-Key": "{{secret__api_key}}",
      "Authorization": "Bearer {{secret__token}}"
    },
    "path_params_schema": {},
    "query_params_schema": {},
    "request_body_schema": {
      "type": "object",
      "properties": {
        "param_name": {
          "type": "string",
          "description": "Beschreibung des Parameters mit Beispiel",
          "required": true,
          "enum": ["option1", "option2"]
        }
      }
    },
    "response_data": [
      {
        "name": "field_name",
        "description": "Was dieses Feld enthält",
        "type": "string"
      }
    ]
  }
}
```

### Parameter-Optionen

| Option | Typ | Standard | Beschreibung |
|--------|-----|----------|-------------|
| `response_timeout_secs` | integer | 20 | Max. Wartezeit auf Tool-Response |
| `disable_interruptions` | boolean | false | User kann Agent nicht unterbrechen während Tool läuft |
| `force_pre_tool_speech` | boolean | false | Agent sagt etwas VOR dem Tool-Aufruf ("Einen Moment...") |
| `execution_mode` | enum | "immediate" | `immediate`, `post_tool_speech`, `async` |
| `tool_call_sound` | enum | - | `typing`, `elevator1`, `elevator2`, `elevator3`, `elevator4` |
| `tool_call_sound_behavior` | enum | "auto" | `auto`, `always` |
| `tool_error_handling_mode` | enum | "auto" | `auto`, `summarized`, `passthrough`, `hide` |
| `assignments` | array | [] | Variablen aus Tool-Response extrahieren |

### Parameter-Typen

| Typ | Beschreibung | Beispiel |
|-----|-------------|---------|
| `string` | Text | `"ORD-12345"` |
| `number` | Dezimalzahl | `42.5` |
| `integer` | Ganzzahl | `42` |
| `boolean` | Wahr/Falsch | `true` |
| `array` | Liste | `["a", "b"]` |
| `object` | Verschachteltes Objekt | `{"key": "value"}` |

### Parameter-Eigenschaften

| Eigenschaft | Pflicht | Beschreibung |
|------------|---------|-------------|
| `type` | Ja | Datentyp des Parameters |
| `description` | Ja | Beschreibung (wird dem LLM gezeigt!) |
| `required` | Nein | Ob Parameter erforderlich ist |
| `enum` | Nein | Erlaubte Werte als Array |
| `default` | Nein | Standardwert |

---

## 2. Tool Description Best Practices

Die Description ist **KRITISCH** – sie bestimmt, wann das LLM das Tool aufruft.

### Schlechte Descriptions

```
"Holt Bestellinfos"
"Prüft die Datenbank"
"API-Call für Bestellungen"
```

### Gute Descriptions

```
"Prüft den Status einer Kundenbestellung. Nutze dieses Tool wenn der Kunde:
- Nach 'wo ist meine Bestellung?' fragt
- Den Versandstatus wissen möchte
- Nach dem Lieferdatum fragt
- Eine Bestellnummer erwähnt"
```

### Parameter-Descriptions

**Schlecht:**
```
order_id: "Die Bestell-ID"
```

**Gut:**
```
order_id: "Die Bestellnummer des Kunden im Format ORD-XXXXX oder nur der numerische Teil. Frage den Kunden wenn nicht angegeben."
```

---

## 3. System Tools

Built-in Tools die keine Konfiguration brauchen:

### end_call
```json
{
  "type": "system",
  "name": "end_call",
  "description": "Beendet das aktuelle Gespräch."
}
```

**Im Prompt definieren:**
```markdown
Beende das Gespräch wenn:
- Nutzer sich verabschiedet
- Aufgabe erledigt und Nutzer bestätigt
- Nutzer bittet das Gespräch zu beenden
```

### transfer_to_number
```json
{
  "type": "system",
  "name": "transfer_to_number",
  "description": "Leitet an eine andere Telefonnummer weiter."
}
```

**Im Prompt definieren:**
```markdown
Leite an einen menschlichen Mitarbeiter weiter wenn:
- Nutzer es ausdrücklich verlangt
- Problem außerhalb deiner Fähigkeiten
- Rückerstattung über 500€ angefragt
```

### transfer_to_agent
```json
{
  "type": "system",
  "name": "transfer_to_agent",
  "description": "Leitet an einen anderen ElevenLabs-Agenten weiter."
}
```

### language_detection
```json
{
  "type": "system",
  "name": "language_detection",
  "description": "Erkennt die Sprache des Nutzers automatisch."
}
```

### Weitere System Tools
- `skip_turn` – Überspringt den aktuellen Turn
- `play_keypad_touch_tone` – Spielt Tastentöne ab
- `voicemail_detection` – Erkennt Anrufbeantworter

---

## 4. Tools zu Agenten hinzufügen (API)

### Neues Tool hinzufügen

> **WICHTIG**: Die `tools`-Array wird beim PATCH **komplett ersetzt**! Immer ALLE gewünschten Tools mitschicken.

**Workflow:**
1. Bestehenden Agent abrufen: `GET /v1/convai/agents/{agent_id}`
2. Bestehende Tools aus dem Response extrahieren
3. Neues Tool zum Array hinzufügen
4. Komplettes Tools-Array per PATCH senden

```json
PATCH https://api.elevenlabs.io/v1/convai/agents/{agent_id}
Header: xi-api-key: YOUR_API_KEY

{
  "conversation_config": {
    "agent": {
      "prompt": {
        "tools": [
          { "...bestehende Tools..." },
          {
            "type": "webhook",
            "name": "neues_tool",
            "description": "Beschreibung...",
            "api_schema": {
              "url": "https://n8n.example.com/webhook/neues-tool",
              "method": "POST",
              "content_type": "application/json",
              "request_body_schema": {
                "type": "object",
                "properties": {
                  "param1": { "type": "string", "required": true }
                }
              }
            }
          }
        ]
      }
    }
  }
}
```

### Tool entfernen

Sende das Tools-Array ohne das zu entfernende Tool:

```json
{
  "conversation_config": {
    "agent": {
      "prompt": {
        "tools": []
      }
    }
  }
}
```

---

## 5. Dynamic Variables in Tools

### In Tool-Config (ElevenLabs UI oder API)

Parameter können auf Dynamic Variables referenzieren:
```json
{
  "request_body_schema": {
    "type": "object",
    "properties": {
      "lead_id": {
        "type": "string",
        "description": "Lead-ID aus dynamischen Variablen",
        "required": false
      }
    }
  }
}
```

### In Headers (Secret Variables)

API-Keys die nicht durch das LLM gehen sollen:
```json
"request_headers": {
  "Authorization": "Bearer {{secret__api_token}}",
  "X-API-Key": "{{secret__api_key}}"
}
```

1. Secret in ElevenLabs erstellen: `secret__api_key`
2. In Headers referenzieren: `{{secret__api_key}}`

### Im System Prompt

```markdown
# Tools
## check_order
Dieses Tool erhält automatisch die Lead_ID des Kunden.
Du musst nur nach der Bestellnummer fragen.
```

---

## 6. Tool Chaining (Mehrere Tools nacheinander)

### Im System Prompt definieren

```markdown
# Tools

## verify_identity
Nutze ZUERST vor jeder Konto-Operation.

## get_account_details
Nutze NUR NACHDEM verify_identity erfolgreich war.

## update_account
Nutze NUR NACHDEM:
1. verify_identity erfolgreich war
2. get_account_details den aktuellen Stand bestätigt hat
3. Kunde die Änderung bestätigt hat
```

### In Tool Descriptions

```
Name: process_refund
Description: Verarbeitet eine Rückerstattung. Voraussetzungen:
1. verify_identity muss erfolgreich gewesen sein
2. check_refund_eligibility muss eligible=true zeigen
3. Kunde muss die Rückerstattung mündlich bestätigt haben

NICHT nutzen wenn eine Voraussetzung fehlt.
```

---

## 7. Tool Response Guidelines

### Voice-freundliches Format

**Immer ein `message`-Feld** mit sprechbarem Text zurückgeben:

```javascript
// GUT - enthält sprechbare Nachricht
{
  success: true,
  message: "Ich habe 3 verfügbare Termine diese Woche gefunden. Passt Ihnen Dienstag um 14 Uhr, Mittwoch um 10 Uhr oder Freitag um 15 Uhr?",
  slots: [...]
}

// SCHLECHT - nur Rohdaten
{
  success: true,
  slots: [
    { datetime: "2026-01-28T14:00:00Z" },
    { datetime: "2026-01-29T10:00:00Z" }
  ]
}
```

### Response-Größe

| Größe | Richtlinie |
|-------|-----------|
| < 200 Zeichen | Ideal für einfache Bestätigungen |
| 200-500 Zeichen | Gut für detaillierte Informationen |
| > 500 Zeichen | Kann lange Pausen verursachen – zusammenfassen! |

### Fehler-Responses

```javascript
// GUTER Fehler-Response
{
  success: false,
  error_type: "not_found",
  message: "Ich konnte keine Bestellung mit dieser Nummer finden. Können Sie die Nummer nochmal prüfen? Sie sollte mit ORD- beginnen."
}

// SCHLECHTER Fehler-Response
{
  error: "404 Not Found",
  stack: "Error at line 45..."
}
```

---

## 8. Tool-Pattern Bibliothek

### Daten-Abfrage Tool

```json
{
  "type": "webhook",
  "name": "get_customer_info",
  "description": "Ruft Kundeninformationen ab. Nutze wenn Kunde nach seinem Konto, Mitgliedsstatus oder Identitätsprüfung fragt.",
  "response_timeout_secs": 10,
  "api_schema": {
    "url": "https://n8n.example.com/webhook/customer-info",
    "method": "POST",
    "content_type": "application/json",
    "request_body_schema": {
      "type": "object",
      "properties": {
        "customer_id": {
          "type": "string",
          "description": "Kundennummer oder E-Mail-Adresse",
          "required": true
        }
      }
    }
  }
}
```

### Termin-Buchung Tool

```json
{
  "type": "webhook",
  "name": "book_appointment",
  "description": "Bucht einen Termin. NUR nutzen NACHDEM: 1. Kunde einen Termin bestätigt hat 2. Datum und Uhrzeit feststehen 3. Kontaktdaten vorliegen",
  "response_timeout_secs": 20,
  "tool_call_sound": "typing",
  "force_pre_tool_speech": true,
  "api_schema": {
    "url": "https://n8n.example.com/webhook/book-appointment",
    "method": "POST",
    "content_type": "application/json",
    "request_body_schema": {
      "type": "object",
      "properties": {
        "datetime": {
          "type": "string",
          "description": "Datum und Uhrzeit, z.B. '2026-01-28 14:00'",
          "required": true
        },
        "customer_email": {
          "type": "string",
          "description": "E-Mail für Bestätigung",
          "required": true
        },
        "appointment_type": {
          "type": "string",
          "description": "Art des Termins",
          "required": true,
          "enum": ["beratung", "demo", "support"]
        },
        "notes": {
          "type": "string",
          "description": "Besondere Anforderungen",
          "required": false
        }
      }
    }
  }
}
```

### Identitätsprüfung Tool

```json
{
  "type": "webhook",
  "name": "verify_identity",
  "description": "Prüft die Identität des Kunden bevor sensible Daten geteilt werden. Nutze wenn Kunde Kontoänderungen oder sensible Informationen anfragt.",
  "response_timeout_secs": 10,
  "api_schema": {
    "url": "https://n8n.example.com/webhook/verify-identity",
    "method": "POST",
    "content_type": "application/json",
    "request_body_schema": {
      "type": "object",
      "properties": {
        "phone_last_four": {
          "type": "string",
          "description": "Letzte 4 Ziffern der hinterlegten Telefonnummer",
          "required": true
        },
        "zip_code": {
          "type": "string",
          "description": "Rechnungs-Postleitzahl",
          "required": true
        }
      }
    }
  }
}
```

### Öffnungszeiten Tool

```json
{
  "type": "webhook",
  "name": "check_business_hours",
  "description": "Prüft ob wir gerade geöffnet haben und gibt die heutigen Öffnungszeiten zurück. Nutze wenn Kunde nach Öffnungszeiten fragt.",
  "response_timeout_secs": 5,
  "api_schema": {
    "url": "https://n8n.example.com/webhook/business-hours",
    "method": "GET"
  }
}
```

### Support-Ticket Tool

```json
{
  "type": "webhook",
  "name": "create_ticket",
  "description": "Erstellt ein Support-Ticket für Probleme die nicht sofort gelöst werden können. Nutze wenn: - Problem technische Untersuchung erfordert - Kunde Follow-up wünscht - Spezialist nötig ist",
  "response_timeout_secs": 15,
  "tool_call_sound": "typing",
  "api_schema": {
    "url": "https://n8n.example.com/webhook/create-ticket",
    "method": "POST",
    "content_type": "application/json",
    "request_body_schema": {
      "type": "object",
      "properties": {
        "issue_summary": {
          "type": "string",
          "description": "Kurze Beschreibung des Problems",
          "required": true
        },
        "priority": {
          "type": "string",
          "description": "Priorität basierend auf Dringlichkeit",
          "required": true,
          "enum": ["low", "medium", "high"]
        },
        "customer_email": {
          "type": "string",
          "description": "E-Mail für Ticket-Updates",
          "required": true
        }
      }
    }
  }
}
```

---

## 9. Authentifizierung

### Bearer Token

```json
"request_headers": {
  "Authorization": "Bearer {{secret__api_token}}"
}
```

### API Key Header

```json
"request_headers": {
  "X-API-Key": "{{secret__api_key}}"
}
```

### Basic Auth

```json
"request_headers": {
  "Authorization": "Basic {{secret__basic_auth}}"
}
```

> `secret__basic_auth` ist Base64-encoded `username:password`

### OAuth2 (via n8n)

OAuth in n8n-Credentials handhaben, einfachen Webhook an ElevenLabs exponieren.

---

## 10. n8n Tool Webhook Patterns

### Basic Handler

```
Webhook → Process → Respond (200)
```

### Mit Validation

```
Webhook → Validate Input → Process → Format Response → Respond (200)
```

### Mit Error Handling

```
Webhook → IF: Valid? ─────── YES → Process → Format → Respond (200)
                    └── NO → Error Response → Respond (200)
```

> **WICHTIG**: Auch bei Fehlern immer HTTP 200 zurückgeben! Der Fehler wird im JSON-Body kommuniziert.

### Kompletter n8n Tool Handler (Code Node)

```javascript
// 1. Parameter extrahieren
const params = $input.first().json.body;

// 2. Pflichtfelder validieren
if (!params.order_id) {
  return [{
    json: {
      success: false,
      message: "Ich brauche die Bestellnummer. Wie lautet Ihre Bestellnummer?"
    }
  }];
}

// 3. Ergebnis formatieren (nach DB/API-Abfrage im vorherigen Node)
const result = $input.first().json;

if (!result || result.error) {
  return [{
    json: {
      success: false,
      message: "Ich habe gerade Schwierigkeiten auf die Bestellung zuzugreifen. Lassen Sie mich es nochmal versuchen."
    }
  }];
}

// 4. Voice-freundliche Response
return [{
  json: {
    success: true,
    message: `Ihre Bestellung ${result.id} ist ${result.status}. ${
      result.tracking
        ? `Sie wird über ${result.carrier} versendet und sollte ${result.eta} ankommen.`
        : `Wir bereiten sie gerade vor.`
    }`,
    data: result
  }
}];
```

---

## 11. Debugging

### Häufige Probleme

| Symptom | Wahrscheinliche Ursache | Lösung |
|---------|------------------------|--------|
| Tool wird nie aufgerufen | Schlechte Description | Description spezifischer machen |
| Falsches Tool aufgerufen | Überlappende Descriptions | Use Cases klar differenzieren |
| Fehlende Parameter | Unklare Param-Descriptions | Formatbeispiele hinzufügen |
| Timeout-Fehler | Langsamer n8n-Workflow | Caching, Queries optimieren |
| Falsche Daten extrahiert | Mehrdeutiges Gespräch | Verifizierungsschritt hinzufügen |

### Testing-Checkliste

- [ ] Tool triggert bei erwarteten Phrasen
- [ ] Tool triggert NICHT bei unverwandten Phrasen
- [ ] Parameter werden korrekt extrahiert
- [ ] Fehlerfälle werden graceful behandelt
- [ ] Response ist voice-freundlich
- [ ] Response-Zeit < 10 Sekunden
- [ ] Alle Secret-Variables sind konfiguriert
- [ ] n8n-Webhook ist aktiv und erreichbar

---

## Quellen

- [Server Tools Dokumentation](https://elevenlabs.io/docs/agents-platform/customization/tools/server-tools)
- [Client Tools Dokumentation](https://elevenlabs.io/docs/agents-platform/customization/tools/client-tools)
- [System Tools Dokumentation](https://elevenlabs.io/docs/conversational-ai/customization/tools/system-tools)
- [Update Agent API](https://elevenlabs.io/docs/api-reference/agents/update)
- [Tools Overview](https://elevenlabs.io/docs/conversational-ai/customization/tools)
