# ElevenLabs Conversational AI – Komplettanleitung

> Agent-Management, Webhooks, Tools, Outbound Calls und n8n-Integration

---

## Übersicht

ElevenLabs Conversational AI bietet eine vollständige Plattform für Voice-Agenten. Diese Anleitung deckt ab:

| Bereich | Was du tun kannst |
|---------|-------------------|
| **Agent Management** | Erstellen, lesen, aktualisieren (LLM, Prompt, Voice, Tools, Webhooks) |
| **Webhooks** | Post-Call Reports, Conversation Initiation, dynamische Variablen |
| **Server Tools** | Mid-Call Webhooks, Datenabfragen, Aktionen während des Gesprächs |
| **Outbound Calls** | Automatische ausgehende Anrufe via Twilio/SIP |
| **n8n-Integration** | Workflow-Patterns für alle Integrationstypen |

---

## 1. ElevenLabs MCP Tools (24 Tools)

### Agent-Management

| MCP Tool | Beschreibung | Kosten |
|----------|-------------|--------|
| `mcp__elevenlabs__create_agent` | Agent erstellen (Name, Prompt, Voice, LLM) | Ja |
| `mcp__elevenlabs__get_agent` | Agent-Details abrufen | Nein |
| `mcp__elevenlabs__list_agents` | Alle Agenten auflisten | Nein |
| `mcp__elevenlabs__add_knowledge_base_to_agent` | Knowledge Base hinzufügen | Ja |

### Telefonie & Calls

| MCP Tool | Beschreibung | Kosten |
|----------|-------------|--------|
| `mcp__elevenlabs__list_phone_numbers` | Telefonnummern auflisten | Nein |
| `mcp__elevenlabs__make_outbound_call` | Ausgehenden Anruf starten | Ja |

### Conversations

| MCP Tool | Beschreibung | Kosten |
|----------|-------------|--------|
| `mcp__elevenlabs__list_conversations` | Gespräche auflisten (mit Filter) | Nein |
| `mcp__elevenlabs__get_conversation` | Einzelgespräch mit Transkript | Nein |

### Voice & Audio

| MCP Tool | Beschreibung | Kosten |
|----------|-------------|--------|
| `mcp__elevenlabs__text_to_speech` | Text → Sprache | Ja |
| `mcp__elevenlabs__speech_to_text` | Sprache → Text | Ja |
| `mcp__elevenlabs__speech_to_speech` | Voice Conversion | Ja |
| `mcp__elevenlabs__search_voices` | Eigene Voices durchsuchen | Nein |
| `mcp__elevenlabs__search_voice_library` | Öffentliche Voice-Library | Nein |
| `mcp__elevenlabs__get_voice` | Voice-Details | Nein |
| `mcp__elevenlabs__voice_clone` | Voice klonen (Audio-Dateien) | Ja |
| `mcp__elevenlabs__text_to_voice` | Voice aus Beschreibung designen | Ja |
| `mcp__elevenlabs__create_voice_from_preview` | Voice-Preview speichern | Ja |

### Audio-Utilities

| MCP Tool | Beschreibung | Kosten |
|----------|-------------|--------|
| `mcp__elevenlabs__isolate_audio` | Vocals isolieren / Noise Removal | Ja |
| `mcp__elevenlabs__play_audio` | Audio lokal abspielen | Nein |
| `mcp__elevenlabs__text_to_sound_effects` | Sound-Effekte generieren | Ja |
| `mcp__elevenlabs__compose_music` | Musik generieren | Ja |
| `mcp__elevenlabs__create_composition_plan` | Kompositionsplan erstellen | Nein |

### Account

| MCP Tool | Beschreibung | Kosten |
|----------|-------------|--------|
| `mcp__elevenlabs__check_subscription` | Abo-Status & Nutzung | Nein |
| `mcp__elevenlabs__list_models` | Verfügbare TTS-Modelle | Nein |

### Lücken (nur via REST API)

| Funktion | Workaround |
|----------|------------|
| **Agent aktualisieren** | `PATCH /v1/convai/agents/{id}` via HTTP Request |
| **Agent löschen** | `DELETE /v1/convai/agents/{id}` via HTTP Request |
| **Webhooks konfigurieren** | Über Agent-Update-API |
| **Tools hinzufügen/ändern** | Über Agent-Update-API |
| **LLM nach Erstellung ändern** | Über Agent-Update-API |
| **Telefonnummern verwalten** | Nur Lesen via MCP |

---

## 2. Agent erstellen

### Via MCP Tool (empfohlen für schnelle Erstellung)

```
mcp__elevenlabs__create_agent({
  name: "Sophie - Kundenservice",
  first_message: "Hallo, hier ist Sophie. Wie kann ich Ihnen helfen?",
  system_prompt: "Du bist Sophie, eine freundliche Kundenservice-Agentin...",
  llm: "gpt-4.1",
  language: "de",
  temperature: 0.5,
  voice_id: "cGDgTTxzmC34oVBGkcEm",
  model_id: "eleven_v3_conversational",
  stability: 0.4,
  similarity_boost: 0.8,
  optimize_streaming_latency: 3,
  turn_timeout: 7,
  max_duration_seconds: 600,
  record_voice: true,
  retention_days: 730
})
```

### Via REST API (vollständige Kontrolle)

```
POST https://api.elevenlabs.io/v1/convai/agents/create
Header: xi-api-key: YOUR_API_KEY
```

```json
{
  "name": "Sophie - Kundenservice",
  "conversation_config": {
    "agent": {
      "first_message": "Hallo, hier ist Sophie. Wie kann ich Ihnen helfen?",
      "language": "de",
      "prompt": {
        "prompt": "Du bist Sophie, eine freundliche Kundenservice-Agentin...",
        "llm": "gpt-4.1",
        "temperature": 0.5,
        "max_tokens": -1,
        "tools": [],
        "knowledge_base": []
      }
    },
    "asr": {
      "quality": "high",
      "provider": "elevenlabs"
    },
    "tts": {
      "model_id": "eleven_v3_conversational",
      "voice_id": "cGDgTTxzmC34oVBGkcEm",
      "optimize_streaming_latency": 3,
      "stability": 0.4,
      "similarity_boost": 0.8,
      "expressive_mode": true,
      "agent_output_audio_format": "pcm_16000"
    },
    "turn": {
      "turn_timeout": 7,
      "turn_eagerness": "normal"
    },
    "conversation": {
      "max_duration_seconds": 600,
      "client_events": ["audio", "agent_response", "user_transcript"]
    }
  },
  "platform_settings": {
    "privacy": {
      "record_voice": true,
      "retention_days": 730
    },
    "auth": {
      "enable_auth": false
    }
  }
}
```

---

## 3. Agent aktualisieren (PATCH API)

> **WICHTIG**: Es gibt KEIN MCP-Tool zum Aktualisieren! Immer HTTP Request verwenden.

```
PATCH https://api.elevenlabs.io/v1/convai/agents/{agent_id}
Header: xi-api-key: YOUR_API_KEY
Content-Type: application/json
```

### 3.1 LLM ändern

```json
{
  "conversation_config": {
    "agent": {
      "prompt": {
        "llm": "gpt-4.1",
        "temperature": 0.5
      }
    }
  }
}
```

### 3.2 System-Prompt aktualisieren

```json
{
  "conversation_config": {
    "agent": {
      "prompt": {
        "prompt": "Neuer System-Prompt hier..."
      }
    }
  }
}
```

### 3.3 Voice ändern

```json
{
  "conversation_config": {
    "tts": {
      "voice_id": "neue_voice_id",
      "model_id": "eleven_v3_conversational"
    }
  }
}
```

### 3.4 First Message ändern

```json
{
  "conversation_config": {
    "agent": {
      "first_message": "Neue Begrüßung hier..."
    }
  }
}
```

### 3.5 Turn-Einstellungen anpassen

```json
{
  "conversation_config": {
    "turn": {
      "turn_timeout": 10,
      "turn_eagerness": "patient",
      "silence_end_call_timeout": 30
    }
  }
}
```

---

## 4. Webhooks konfigurieren

### 4.1 Post-Call Webhook einrichten

Feuert **nach jedem Gespräch** mit Transkript, Analyse und Metadaten.

```json
PATCH /v1/convai/agents/{agent_id}

{
  "platform_settings": {
    "workspace_overrides": {
      "webhooks": {
        "events": ["transcript", "audio", "call_initiation_failure"],
        "send_audio": false
      }
    }
  }
}
```

> **Hinweis**: Post-Call-Webhooks werden über `post_call_webhook_id` oder über die ElevenLabs-UI konfiguriert. Die Webhook-URL wird in den Workspace-Settings der ElevenLabs-Plattform definiert.

#### Webhook-Payload (wird an deine URL gesendet)

```json
{
  "type": "post_call_transcription",
  "event_timestamp": 1706234567890,
  "data": {
    "agent_id": "agent_abc123",
    "conversation_id": "conv_xyz789",
    "status": "done",
    "transcript": [
      {
        "role": "agent",
        "message": "Hallo, wie kann ich Ihnen helfen?",
        "time_in_call_secs": 0.5
      },
      {
        "role": "user",
        "message": "Ich möchte meinen Bestellstatus prüfen.",
        "time_in_call_secs": 2.1
      }
    ],
    "metadata": {
      "call_duration_secs": 145,
      "start_time_unix_secs": 1706234400,
      "cost": 150,
      "termination_reason": "user_ended_call"
    },
    "analysis": {
      "call_successful": "success",
      "transcript_summary": "Kunde hat nach Bestellung #12345 gefragt...",
      "evaluation_criteria_results": {},
      "data_collection_results": {
        "order_id": "12345",
        "customer_sentiment": "satisfied"
      }
    },
    "conversation_initiation_client_data": {
      "dynamic_variables": {
        "user_name": "John",
        "account_id": "ACC-001"
      }
    }
  }
}
```

#### Webhook-Typen

| Typ | Event | Inhalt |
|-----|-------|--------|
| `post_call_transcription` | Anruf endet | Transkript, Analyse, Metadaten |
| `post_call_audio` | Anruf endet | Base64-encoded MP3 Audio |
| `call_initiation_failure` | Anruf fehlgeschlagen | Fehlergrund, Telefonie-Metadaten |

### 4.2 Conversation Initiation Webhook einrichten

Feuert **vor Gesprächsbeginn** – erlaubt Personalisierung basierend auf Anrufer-Info.

```json
PATCH /v1/convai/agents/{agent_id}

{
  "platform_settings": {
    "workspace_overrides": {
      "conversation_initiation_client_data_webhook": {
        "url": "https://n8n.example.com/webhook/elevenlabs-init",
        "request_headers": {
          "Authorization": "Bearer your-secret-token"
        }
      }
    },
    "overrides": {
      "conversation_config_override": {
        "agent": {
          "prompt": { "prompt": true },
          "first_message": true,
          "language": true
        },
        "tts": { "voice_id": true }
      }
    }
  }
}
```

> **WICHTIG**: Die `overrides` müssen auf `true` gesetzt werden, damit der Webhook Felder überschreiben darf!

#### Request von ElevenLabs an dein n8n-Webhook

```json
{
  "type": "conversation_initiation",
  "agent_id": "agent_abc123",
  "conversation_id": "conv_xyz789",
  "caller_id": "+15551234567",
  "called_number": "+15559876543",
  "call_sid": "CA1234567890abcdef"
}
```

#### Erforderliche Antwort von n8n

```json
{
  "dynamic_variables": {
    "user_name": "Max Mustermann",
    "account_status": "Premium",
    "last_order": "ORD-12345"
  },
  "conversation_config_override": {
    "agent": {
      "prompt": {
        "prompt": "Du hilfst {{user_name}}, einem {{account_status}} Mitglied..."
      },
      "first_message": "Hallo {{user_name}}! Wie kann ich Ihnen helfen?"
    }
  }
}
```

#### Wichtige Regeln

- **Alle Variablen müssen zurückgegeben werden**: Wenn der System-Prompt `{{variable}}` nutzt, MUSS diese im Response sein
- **Timeout**: Antwort muss innerhalb von ~5 Sekunden kommen
- **Variablen sind Strings**: Zahlen zu Strings konvertieren

---

## 5. Server Tools (Mid-Call Webhooks) zu Agenten hinzufügen

### 5.1 Tool via Agent-Update-API hinzufügen

```json
PATCH /v1/convai/agents/{agent_id}

{
  "conversation_config": {
    "agent": {
      "prompt": {
        "tools": [
          {
            "type": "webhook",
            "name": "check_order_status",
            "description": "Prüft den Status einer Kundenbestellung. Nutze dieses Tool wenn der Kunde nach seiner Bestellung, Lieferung oder Versand fragt.",
            "response_timeout_secs": 15,
            "api_schema": {
              "url": "https://n8n.example.com/webhook/check-order",
              "method": "POST",
              "content_type": "application/json",
              "request_headers": {
                "X-API-Key": "{{secret__api_key}}"
              },
              "request_body_schema": {
                "type": "object",
                "properties": {
                  "order_id": {
                    "type": "string",
                    "description": "Die Bestellnummer im Format ORD-XXXXX",
                    "required": true
                  },
                  "lead_id": {
                    "type": "string",
                    "description": "Die Lead-ID aus den dynamischen Variablen",
                    "required": false
                  }
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

### 5.2 Mehrere Tools hinzufügen

> **WICHTIG**: Die `tools`-Array wird komplett ersetzt! Immer ALLE gewünschten Tools mitschicken.

```json
{
  "conversation_config": {
    "agent": {
      "prompt": {
        "tools": [
          {
            "type": "webhook",
            "name": "check_order",
            "description": "Prüft den Bestellstatus...",
            "response_timeout_secs": 15,
            "api_schema": {
              "url": "https://n8n.example.com/webhook/check-order",
              "method": "POST",
              "request_body_schema": {
                "type": "object",
                "properties": {
                  "order_id": { "type": "string", "required": true }
                }
              }
            }
          },
          {
            "type": "webhook",
            "name": "book_appointment",
            "description": "Bucht einen Termin für den Kunden...",
            "response_timeout_secs": 20,
            "api_schema": {
              "url": "https://n8n.example.com/webhook/book-appointment",
              "method": "POST",
              "request_body_schema": {
                "type": "object",
                "properties": {
                  "datetime": { "type": "string", "required": true },
                  "customer_email": { "type": "string", "required": true },
                  "notes": { "type": "string", "required": false }
                }
              }
            }
          },
          {
            "type": "system",
            "name": "end_call",
            "description": "Beendet das Gespräch."
          },
          {
            "type": "system",
            "name": "transfer_to_number",
            "description": "Leitet an einen menschlichen Mitarbeiter weiter."
          }
        ]
      }
    }
  }
}
```

### 5.3 Tool-Typen

| Typ | Beschreibung | Verwendung |
|-----|-------------|------------|
| `webhook` | HTTP-Aufruf an deinen Server | DB-Abfragen, Buchungen, CRM-Updates |
| `client` | Läuft im Browser/App des Nutzers | UI-Updates, Navigation |
| `system` | Built-in Funktionen | `end_call`, `transfer_to_number`, `transfer_to_agent`, `language_detection` |
| `mcp` | MCP Server Integration | Zugriff auf MCP-Tools |

### 5.4 Tool-Optionen (Neue Features)

```json
{
  "type": "webhook",
  "name": "tool_name",
  "description": "...",
  "response_timeout_secs": 20,
  "disable_interruptions": true,
  "force_pre_tool_speech": true,
  "execution_mode": "immediate",
  "tool_call_sound": "typing",
  "tool_call_sound_behavior": "auto",
  "tool_error_handling_mode": "summarized",
  "assignments": [
    {
      "variable_name": "order_status",
      "description": "Der Status der Bestellung aus dem Tool-Response"
    }
  ],
  "api_schema": { ... }
}
```

| Option | Werte | Beschreibung |
|--------|-------|-------------|
| `execution_mode` | `immediate`, `post_tool_speech`, `async` | Wann Tool ausgeführt wird |
| `tool_call_sound` | `typing`, `elevator1`-`elevator4` | Audio-Feedback während Tool-Aufruf |
| `tool_error_handling_mode` | `auto`, `summarized`, `passthrough`, `hide` | Fehlerbehandlung |
| `disable_interruptions` | `true/false` | User kann Agent nicht unterbrechen während Tool läuft |
| `assignments` | Array | Variablen aus Tool-Response extrahieren |

---

## 6. n8n Workflow-Patterns

### 6.1 Post-Call Webhook Handler

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Webhook         │────▶│  Validate HMAC   │────▶│  Extract Data   │
│  (POST)          │     │  (Code Node)     │     │  (Code Node)    │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                          │
         ┌────────────────────────────────────────────────┘
         ▼
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Respond 200    │     │  Process/Save    │────▶│  Notifications  │
│  (Required!)    │     │  (DB/Sheets)     │     │  (Email/Slack)  │
└─────────────────┘     └──────────────────┘     └─────────────────┘
```

#### HMAC Signature Validation (Code Node)

```javascript
const crypto = require('crypto');

const signature = $input.first().json.headers['elevenlabs-signature'];
const body = JSON.stringify($input.first().json.body);
const secret = $env.ELEVENLABS_WEBHOOK_SECRET;

// Signatur parsen: t=timestamp,v0=hash
const parts = signature.split(',');
const timestamp = parts[0].split('=')[1];
const hash = parts[1].split('=')[1];

// Timestamp validieren (30 Min Toleranz)
const now = Math.floor(Date.now() / 1000);
if (Math.abs(now - parseInt(timestamp)) > 1800) {
  throw new Error('Webhook timestamp abgelaufen');
}

// HMAC validieren
const expectedHash = crypto
  .createHmac('sha256', secret)
  .update(`${timestamp}.${body}`)
  .digest('hex');

if (hash !== expectedHash) {
  throw new Error('Ungültige Webhook-Signatur');
}

return [{json: $input.first().json.body}];
```

#### Daten-Extraktion (Code Node)

```javascript
const data = $input.first().json.data;

return [{
  json: {
    conversation_id: data.conversation_id,
    agent_id: data.agent_id,
    call_duration_secs: data.metadata ? data.metadata.call_duration_secs : 0,
    termination_reason: data.metadata ? data.metadata.termination_reason : '',
    cost: data.metadata ? data.metadata.cost : 0,
    call_sid: data.metadata
      ? (data.metadata.twilio ? data.metadata.twilio.call_sid : '')
      : '',
    transcript: (data.transcript || [])
      .map(item => `${item.role}: ${item.message}`)
      .join('\n'),
    summary: data.analysis ? data.analysis.transcript_summary : '',
    call_successful: data.analysis ? data.analysis.call_successful : '',
    data_collection: data.analysis ? (data.analysis.data_collection_results || {}) : {},
    lead_id: data.conversation_initiation_client_data
      ? (data.conversation_initiation_client_data.dynamic_variables
        ? data.conversation_initiation_client_data.dynamic_variables.Lead_ID
        : '')
      : ''
  }
}];
```

> **WICHTIG**: Immer HTTP 200 zurückgeben! Nach 10+ aufeinanderfolgenden Fehlern über 7+ Tage deaktiviert ElevenLabs den Webhook automatisch.

### 6.2 Conversation Initiation Webhook Handler

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Webhook         │────▶│  Lookup Customer │────▶│  Build Response │
│  /init           │     │  (DB/CRM/Sheets) │     │  (Code Node)    │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                          │
                                                          ▼
                                               ┌─────────────────┐
                                               │  Respond to     │
                                               │  Webhook (200)  │
                                               └─────────────────┘
```

#### Response Builder (Code Node)

```javascript
const request = $input.first().json.body;
const customer = $('Lookup Customer').first().json;

const response = {
  dynamic_variables: {
    Name: customer ? (customer.name || "Geschätzter Kunde") : "Geschätzter Kunde",
    account_id: customer ? (customer.id || "GUEST") : "GUEST",
    account_status: customer ? (customer.tier || "Standard") : "Standard",
    open_tickets: String(customer ? (customer.open_tickets || 0) : 0),
    last_purchase: customer ? (customer.last_order_date || "N/A") : "N/A"
  }
};

// Optional: System-Prompt für VIP-Kunden überschreiben
if (customer && customer.vip) {
  response.conversation_config_override = {
    agent: {
      prompt: {
        prompt: `Du bist ein VIP-Concierge für ${customer.name}.
                 Priorisiere die Anfragen und biete Premium-Support.`
      },
      first_message: `Hallo ${customer.name}! Als VIP-Kunde haben Sie Vorrang. Wie kann ich helfen?`
    }
  };
}

return [{json: response}];
```

### 6.3 Server Tool Handler (Mid-Call)

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Webhook         │────▶│  Query Database  │────▶│  Format Voice   │
│  /tool-name      │     │  or External API │     │  Response       │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                          │
                                                          ▼
                                               ┌─────────────────┐
                                               │  Respond 200    │
                                               │  mit JSON       │
                                               └─────────────────┘
```

#### Voice-freundliche Response (Code Node)

```javascript
const order = $input.first().json;

const response = {
  success: true,
  order_status: order.status,
  message: `Bestellung ${order.id} ist ${order.status}.
            ${order.status === 'shipped'
              ? `Sie wurde am ${order.ship_date} verschickt und sollte ${order.eta} ankommen.`
              : order.status === 'processing'
              ? `Sie wird gerade vorbereitet und sollte innerhalb von 2 Werktagen verschickt werden.`
              : `Aktueller Status: ${order.status}.`
            }`
};

return [{json: response}];
```

#### Tool Response Best Practices

| Tun | Nicht tun |
|-----|-----------|
| Kurzen, sprechbaren Text zurückgeben | Rohe Datenbank-Records |
| Wichtige Info zuerst | Wichtiges in verschachtelten Objekten |
| Dates natürlich ("Freitag, 26. Januar") | ISO-Dates ("2026-01-26T00:00:00Z") |
| < 500 Zeichen anstreben | Mehrabsatz-Antworten |
| Innerhalb 10-15 Sek. antworten | Langsame API-Calls ohne Timeout |

---

## 7. Outbound Call API

### Via MCP Tool

```
mcp__elevenlabs__make_outbound_call({
  agent_id: "agent_abc123",
  agent_phone_number_id: "phnum_xyz789",
  to_number: "+491511234567"
})
```

### Via REST API (mit Dynamic Variables)

```
POST https://api.elevenlabs.io/v1/convai/twilio/outbound-call
Header: xi-api-key: YOUR_API_KEY
```

```json
{
  "agent_id": "agent_abc123",
  "agent_phone_number_id": "phnum_xyz789",
  "to_number": "+491511234567",
  "dynamic_variables": {
    "Name": "Max Mustermann",
    "Business Name": "Musterfirma GmbH",
    "E-Mail": "max@musterfirma.de",
    "Phone": "+491511234567",
    "Lead_ID": "ID00001",
    "Information": "Interesse an Enterprise-Plan",
    "call_attempt": "1",
    "previous_status": "Neu"
  },
  "conversation_config_override": {
    "agent": {
      "first_message": "Hallo {{Name}}, hier ist Sophie von [Firma]..."
    }
  }
}
```

### n8n HTTP Request Node

```
Method: POST
URL: https://api.elevenlabs.io/v1/convai/twilio/outbound-call
Headers:
  - xi-api-key: {{ $env.ELEVENLABS_API_KEY }}
Body (JSON): { ... wie oben ... }
```

**Response:**
```json
{
  "success": true,
  "conversation_id": "conv_abc123xyz"
}
```

> **WICHTIG**: `conversation_id` speichern, um Post-Call-Webhook-Daten zuzuordnen!

---

## 8. Komplettes Agent-Setup Beispiel

### Agent mit Tools, Webhooks und GPT-4.1 erstellen

**Schritt 1: Agent erstellen via MCP**

```
mcp__elevenlabs__create_agent({
  name: "Sophie - Sales Setter",
  first_message: "Hallo {{Name}}, hier ist Sophie von [Firma]. Haben Sie kurz Zeit?",
  system_prompt: "Du bist Sophie, ein AI Sales Setter...",
  llm: "gpt-4.1",
  language: "de",
  temperature: 0.5,
  voice_id: "cGDgTTxzmC34oVBGkcEm",
  model_id: "eleven_v3_conversational",
  stability: 0.4,
  similarity_boost: 0.8,
  max_duration_seconds: 600
})
```

**Schritt 2: Tools und Webhooks hinzufügen via API**

```json
PATCH /v1/convai/agents/{agent_id}

{
  "conversation_config": {
    "agent": {
      "prompt": {
        "tools": [
          {
            "type": "webhook",
            "name": "check_availability",
            "description": "Prüft verfügbare Termine. Nutze wenn Kunde einen Termin möchte.",
            "response_timeout_secs": 15,
            "tool_call_sound": "typing",
            "api_schema": {
              "url": "https://n8n.example.com/webhook/check-slots",
              "method": "POST",
              "content_type": "application/json",
              "request_headers": {
                "X-API-Key": "{{secret__api_key}}"
              },
              "request_body_schema": {
                "type": "object",
                "properties": {
                  "date": {
                    "type": "string",
                    "description": "Gewünschtes Datum im Format YYYY-MM-DD",
                    "required": true
                  }
                }
              }
            }
          },
          {
            "type": "webhook",
            "name": "book_appointment",
            "description": "Bucht einen Termin. NUR nutzen NACHDEM: 1. check_availability Verfügbarkeit bestätigt hat 2. Kunde den Termin bestätigt hat",
            "response_timeout_secs": 20,
            "api_schema": {
              "url": "https://n8n.example.com/webhook/book-appointment",
              "method": "POST",
              "content_type": "application/json",
              "request_body_schema": {
                "type": "object",
                "properties": {
                  "datetime": { "type": "string", "required": true },
                  "customer_name": { "type": "string", "required": true },
                  "customer_email": { "type": "string", "required": true },
                  "notes": { "type": "string", "required": false }
                }
              }
            }
          },
          {
            "type": "system",
            "name": "end_call",
            "description": "Beendet das Gespräch höflich."
          },
          {
            "type": "system",
            "name": "transfer_to_number",
            "description": "Leitet an menschlichen Mitarbeiter weiter wenn Kunde es verlangt."
          }
        ]
      }
    }
  },
  "platform_settings": {
    "workspace_overrides": {
      "conversation_initiation_client_data_webhook": {
        "url": "https://n8n.example.com/webhook/elevenlabs-init",
        "request_headers": {
          "Authorization": "Bearer your-secret"
        }
      },
      "webhooks": {
        "events": ["transcript"]
      }
    },
    "overrides": {
      "conversation_config_override": {
        "agent": {
          "prompt": { "prompt": true },
          "first_message": true
        }
      }
    }
  }
}
```

---

## 9. Security Best Practices

### HMAC Validation (Post-Call Webhooks)

```
Signatur-Format: t=timestamp,v0=hash
Hash = HMAC-SHA256(timestamp.body, secret)
```

### IP Whitelisting

| Region | IPs |
|--------|-----|
| **US** | `34.67.146.145`, `34.59.11.47` |
| **EU** | `35.204.38.71`, `34.147.113.54` |
| **Asia** | `35.185.187.110`, `35.247.157.189` |

### Tool Authentication

```javascript
// Header-Validation in n8n Code Node
const apiKey = $input.first().json.headers['x-api-key'];
if (apiKey !== $env.EXPECTED_TOOL_API_KEY) {
  throw new Error('Unauthorized');
}
```

### Environment Variables

```
ELEVENLABS_API_KEY=xi-xxxxx
ELEVENLABS_WEBHOOK_SECRET=whsec_xxxxx
EXPECTED_TOOL_API_KEY=your-secret-key
```

---

## 10. API Endpoint Referenz

| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| `POST` | `/v1/convai/agents/create` | Agent erstellen |
| `GET` | `/v1/convai/agents/{agent_id}` | Agent abrufen |
| `GET` | `/v1/convai/agents` | Alle Agenten auflisten |
| `PATCH` | `/v1/convai/agents/{agent_id}` | Agent aktualisieren |
| `DELETE` | `/v1/convai/agents/{agent_id}` | Agent löschen |
| `POST` | `/v1/convai/agents/{agent_id}/add-knowledge-base` | Knowledge Base hinzufügen |
| `GET` | `/v1/convai/conversations` | Gespräche auflisten |
| `GET` | `/v1/convai/conversations/{conv_id}` | Gespräch abrufen |
| `DELETE` | `/v1/convai/conversations/{conv_id}` | Gespräch löschen |
| `GET` | `/v1/convai/phone-numbers` | Telefonnummern auflisten |
| `POST` | `/v1/convai/phone-numbers/create` | Telefonnummer importieren |
| `PATCH` | `/v1/convai/phone-numbers/{id}` | Telefonnummer aktualisieren |
| `DELETE` | `/v1/convai/phone-numbers/{id}` | Telefonnummer löschen |
| `POST` | `/v1/convai/twilio/outbound-call` | Outbound Call (Twilio) |
| `POST` | `/v1/convai/sip-trunk/outbound-call` | Outbound Call (SIP) |
| `GET` | `/v1/convai/knowledge-base` | Knowledge Bases auflisten |
| `DELETE` | `/v1/convai/knowledge-base/{id}` | Knowledge Base löschen |

**Base URL:** `https://api.elevenlabs.io`
**Auth Header:** `xi-api-key: YOUR_API_KEY`

---

## 11. Fehlerbehandlung

### Häufige Probleme & Lösungen

| Problem | Ursache | Lösung |
|---------|---------|--------|
| Webhook wird nicht ausgelöst | Falsche HTTP-Methode | Sicherstellen: POST in beiden Systemen |
| Variablen undefined | Fehlen im Response | ALLE definierten Variablen zurückgeben |
| Tool wird nicht aufgerufen | Schlechte Description | Description spezifischer machen |
| Roboterhafte Sprache | Rohe Daten zurückgegeben | Als natürlichen Text formatieren |
| Webhook deaktiviert | 10+ Fehler | Logs prüfen, fixen, re-aktivieren |
| Lead nicht zugeordnet | Falsche Con_ID | conversation_id Speicherung prüfen |

### Timeout-Richtlinien

| Integration | Max. Zeit |
|-------------|-----------|
| Initiation Webhook | ~5 Sekunden |
| Server Tools | 10-15 Sekunden (max 20) |
| Post-Call Webhook | Kein striktes Limit (200 schnell zurückgeben) |

---

## 12. Komplette Architektur

```
                    ┌────────────────────────────────────┐
                    │         ElevenLabs Agent           │
                    │  ┌──────────────────────────────┐  │
                    │  │    System Prompt mit          │  │
                    │  │    {{dynamic_variables}}      │  │
                    │  │    LLM: gpt-4.1              │  │
                    │  │    Tools: webhook + system    │  │
                    │  └──────────────────────────────┘  │
                    └──────────────┬─────────────────────┘
                                   │
     ┌─────────────────────────────┼─────────────────────────────┐
     │                             │                             │
     ▼                             ▼                             ▼
┌─────────────┐            ┌─────────────┐            ┌─────────────┐
│ CALL START  │            │ DURING CALL │            │  CALL END   │
│             │            │             │            │             │
│ Initiation  │            │ Server      │            │ Post-Call   │
│ Webhook     │            │ Tools       │            │ Webhook     │
└──────┬──────┘            └──────┬──────┘            └──────┬──────┘
       │                          │                          │
       ▼                          ▼                          ▼
┌─────────────┐            ┌─────────────┐            ┌─────────────┐
│   n8n       │            │   n8n       │            │   n8n       │
│  /init      │            │  /tools/*   │            │  /report    │
│             │            │             │            │             │
│ • CRM lookup│            │ • DB queries│            │ • Save call │
│ • Variables │            │ • Booking   │            │ • Analytics │
│ • Overrides │            │ • Actions   │            │ • Notify    │
└─────────────┘            └─────────────┘            └─────────────┘
```

---

## Quellen

- [Update Agent API](https://elevenlabs.io/docs/api-reference/agents/update)
- [Post-call Webhooks](https://elevenlabs.io/docs/agents-platform/workflows/post-call-webhooks)
- [Dynamic Variables](https://elevenlabs.io/docs/agents-platform/customization/personalization/dynamic-variables)
- [Server Tools](https://elevenlabs.io/docs/agents-platform/customization/tools/server-tools)
- [GPT-4.1 Support (Changelog April 2025)](https://elevenlabs.io/docs/changelog/2025/4/21)
- [LLM Models](https://elevenlabs.io/docs/agents-platform/customization/llm)
