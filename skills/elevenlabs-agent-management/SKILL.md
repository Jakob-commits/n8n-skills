---
name: elevenlabs-agent-management
description: ElevenLabs ElevenAgents (formerly Conversational AI) management. Use when creating, updating, or configuring voice agents, adding webhooks or tools, changing LLM settings, managing phone numbers, making outbound/batch calls, configuring agent workflows, setting up WhatsApp, versioning/experiments, or integrating with n8n.
---

# ElevenLabs ElevenAgents — Expert Guide

Comprehensive guide for managing ElevenLabs voice agents via MCP tools and REST API.

> **Rebranding:** "Conversational AI" heißt jetzt **ElevenAgents**. Neue Docs: `elevenlabs.io/docs/eleven-agents/`

---

## Standard-Konfiguration

**LLM:** `gpt-4.1` (Standard für alle Agenten)
**TTS:** `eleven_v3_conversational` (75+ Sprachen, Expressive Mode)
**Sprache:** `de` (für deutsche Agenten)
**Temperature:** `0.5` (balanced)

---

## MCP Tools (24 verfügbar)

### Agent-Management

| Tool | Beschreibung |
|------|-------------|
| `mcp__elevenlabs__create_agent` | Agent erstellen |
| `mcp__elevenlabs__get_agent` | Agent-Details abrufen |
| `mcp__elevenlabs__list_agents` | Alle Agenten auflisten |
| `mcp__elevenlabs__add_knowledge_base_to_agent` | Knowledge Base hinzufügen |

### Telefonie & Calls

| Tool | Beschreibung |
|------|-------------|
| `mcp__elevenlabs__make_outbound_call` | Ausgehenden Anruf starten |
| `mcp__elevenlabs__list_phone_numbers` | Telefonnummern auflisten |

### Conversations

| Tool | Beschreibung |
|------|-------------|
| `mcp__elevenlabs__list_conversations` | Gespräche auflisten (mit Filter) |
| `mcp__elevenlabs__get_conversation` | Einzelgespräch mit Transkript abrufen |

### Voice & Audio

| Tool | Beschreibung |
|------|-------------|
| `mcp__elevenlabs__text_to_speech` | Text zu Sprache |
| `mcp__elevenlabs__speech_to_text` | Sprache zu Text |
| `mcp__elevenlabs__speech_to_speech` | Voice Conversion |
| `mcp__elevenlabs__search_voices` | Eigene Voices suchen |
| `mcp__elevenlabs__search_voice_library` | Öffentliche Voice Library |
| `mcp__elevenlabs__get_voice` | Voice-Details abrufen |
| `mcp__elevenlabs__voice_clone` | Voice klonen (Audio-Dateien) |
| `mcp__elevenlabs__text_to_voice` | Voice aus Beschreibung designen |
| `mcp__elevenlabs__create_voice_from_preview` | Voice-Preview speichern |

### Audio-Utilities

| Tool | Beschreibung |
|------|-------------|
| `mcp__elevenlabs__isolate_audio` | Vocals isolieren / Noise Removal |
| `mcp__elevenlabs__play_audio` | Audio lokal abspielen |
| `mcp__elevenlabs__text_to_sound_effects` | Sound-Effekte generieren |
| `mcp__elevenlabs__compose_music` | Musik generieren |
| `mcp__elevenlabs__create_composition_plan` | Kompositionsplan erstellen |

### Account

| Tool | Beschreibung |
|------|-------------|
| `mcp__elevenlabs__check_subscription` | Abo-Status & Nutzung |
| `mcp__elevenlabs__list_models` | Verfügbare TTS-Modelle |

---

## Lücken (NUR via REST API)

| Funktion | Workaround |
|----------|------------|
| **Agent aktualisieren** | `PATCH /v1/convai/agents/{agent_id}` |
| **Agent löschen** | `DELETE /v1/convai/agents/{agent_id}` |
| **Webhooks konfigurieren** | Über Agent-Update-API |
| **Tools hinzufügen/ändern** | Über Agent-Update-API |
| **LLM nach Erstellung ändern** | Über Agent-Update-API |
| **Telefonnummern verwalten** | Nur Lesen via MCP |
| **Batch Calls starten** | REST API |
| **Versioning/Experiments** | REST API oder Dashboard |

**API Base URL:** `https://api.elevenlabs.io`
**Auth Header:** `xi-api-key: YOUR_API_KEY`

---

## Agent erstellen (MCP)

```
mcp__elevenlabs__create_agent({
  name: "Agent Name",
  first_message: "Begrüßung...",
  system_prompt: "System Prompt...",
  llm: "gpt-4.1",
  language: "de",
  temperature: 0.5,
  voice_id: "VOICE_ID",
  model_id: "eleven_v3_conversational",
  stability: 0.4,
  similarity_boost: 0.8,
  optimize_streaming_latency: 3,
  turn_timeout: 7,
  max_duration_seconds: 600,
  record_voice: true
})
```

---

## Agent aktualisieren (REST API)

```
PATCH https://api.elevenlabs.io/v1/convai/agents/{agent_id}
Header: xi-api-key: YOUR_API_KEY
Content-Type: application/json
```

### LLM ändern
```json
{"conversation_config":{"agent":{"prompt":{"llm":"gpt-4.1","temperature":0.5}}}}
```

### System-Prompt ändern
```json
{"conversation_config":{"agent":{"prompt":{"prompt":"Neuer Prompt..."}}}}
```

### Voice ändern
```json
{"conversation_config":{"tts":{"voice_id":"NEUE_ID","model_id":"eleven_v3_conversational"}}}
```

### First Message ändern
```json
{"conversation_config":{"agent":{"first_message":"Neue Begrüßung..."}}}
```

### Turn Eagerness ändern
```json
{"conversation_config":{"conversation":{"turn_eagerness":"patient"}}}
```
Werte: `eager` (Kundenservice), `normal` (Standard), `patient` (Datenerfassung)

### Thinking Budget setzen
```json
{"conversation_config":{"agent":{"prompt":{"reasoning_effort":"medium"}}}}
```
Werte: `none` (Standard für Gespräche), `minimal`, `low`, `medium`, `high`

---

## Verfügbare LLMs

### OpenAI
| Model ID | Empfehlung |
|----------|------------|
| `gpt-5` | Neuestes, höchste Qualität |
| `gpt-4.1` | **STANDARD** — Bestes Instruction-Following |
| `gpt-4.1-mini` | Budget-Option |
| `gpt-4.1-nano` | FAQ-Bots, einfache Aufgaben |
| `gpt-4o` | Gutes Allround-Modell |
| `gpt-4o-mini` | Schnell, günstig |

### Google
| Model ID | Empfehlung |
|----------|------------|
| `gemini-3-pro-preview` | Neuestes Google |
| `gemini-3-flash-preview` | Schnell + stark |
| `gemini-2.5-flash` | Niedrigste Latenz |
| `gemini-2.5-flash-lite` | Ultra-Budget |
| `gemini-2.0-flash` | Schnell |

### Anthropic
| Model ID | Empfehlung |
|----------|------------|
| `claude-sonnet-4-5` | Premium, Nuancen |
| `claude-3-5-sonnet` | Starkes Reasoning |
| `claude-3-haiku` | Budget-Conversational |

### ElevenLabs (gehostet)
| Model ID | Empfehlung |
|----------|------------|
| `GLM-4.5-Air` | ElevenLabs gehostet |
| `Qwen3-30B-A3B` | ElevenLabs gehostet |
| `GPT-OSS-120B` | ElevenLabs gehostet |

### Custom
| Model ID | Empfehlung |
|----------|------------|
| `custom-llm` | Eigener OpenAI-kompatibler Endpoint |

### Modellwahl nach Use Case
| Use Case | Modell | Begründung |
|----------|--------|------------|
| Kundenservice | `gpt-4.1` | Beste Instruction-Treue |
| Sales/Setter | `gpt-4.1` | Gute Gesprächsführung |
| FAQ-Bot | `gpt-4.1-nano` | Schnell, einfache Fragen |
| VIP-Support | `claude-sonnet-4-5` | Höchste Qualität |
| Echtzeit-Hotline | `gemini-2.5-flash` | Niedrigste Latenz |
| Komplex/Reasoning | `gpt-5` | Tiefes Verständnis |

### Temperature-Empfehlungen
| Typ | Wert | Beschreibung |
|-----|------|-------------|
| Support | 0.3 | Faktisch, konsistent |
| Sales | 0.5 | Natürlich, fokussiert |
| Kreativ | 0.7 | Abwechslungsreich |

---

## TTS-Modelle

| Model ID | Empfehlung |
|----------|------------|
| `eleven_v3_conversational` | **Standard** — 75+ Sprachen, Expressive Mode |
| `eleven_turbo_v2_5` | Schnell (32 Sprachen) |
| `eleven_flash_v2_5` | Schnellstes |

### Expressive Mode (NEU)
Nur mit `eleven_v3_conversational`. Kontextbezogene emotionale Sprachausgabe.

**Tags im System Prompt:**
- `[laughs]` — Lachen
- `[whispers]` — Flüstern
- `[sighs]` — Seufzen
- `[slow]` — Langsam sprechen
- `[excited]` — Aufgeregt

Tags wirken auf ca. 4-5 Wörter danach. Preis: $0.08/Minute.

### Voice-Konfiguration
- **Speed:** 0.7x bis 1.2x
- **Stability:** 0.0 bis 1.0 (niedrig = expressiver)
- **Similarity Boost:** 0.0 bis 1.0
- **Multi-Voice:** Verschiedene Stimmen pro Rolle möglich
- **Pronunciation Dictionary:** IPA oder CMU für spezifische Wörter

---

## Tools — Alle 5 Typen

### 1. Webhook (Server Tools)
HTTP-Aufrufe an externe APIs während des Gesprächs.

```json
{
  "type": "webhook",
  "name": "tool_name",
  "description": "Wann und warum nutzen...",
  "response_timeout_secs": 15,
  "disable_interruptions": false,
  "force_pre_tool_speech": false,
  "execution_mode": "immediate",
  "tool_call_sound": "typing",
  "tool_error_handling_mode": "summarized",
  "assignments": [
    {"variable_name": "result", "description": "Was extrahiert wird"}
  ],
  "api_schema": {
    "url": "https://n8n.example.com/webhook/endpoint",
    "method": "POST",
    "content_type": "application/json",
    "request_headers": {
      "X-API-Key": "{{secret__api_key}}"
    },
    "request_body_schema": {
      "type": "object",
      "properties": {
        "param": {"type": "string", "required": true, "description": "..."}
      }
    },
    "response_data": [
      {"name": "field", "description": "Was das Feld enthält", "type": "string"}
    ]
  }
}
```

**Auth-Methoden für Server Tools:**
- OAuth2 Client Credentials (automatischer Token-Flow)
- OAuth2 JWT (JSON Web Token Bearer)
- Basic Authentication
- Bearer Tokens
- Custom Headers mit `{{secret__variable}}`

### 2. System Tools

| Tool | Beschreibung | Parameter |
|------|-------------|-----------|
| `end_call` | Gespräch beenden | `reason` (pflicht), `message` (optional) |
| `language_detection` | Sprache auto-wechseln | `reason`, `language` |
| `transfer_to_agent` | An anderen KI-Agent | `agent_number` (zero-indexed), `reason` |
| `transfer_to_number` | An Telefonnummer | `transfer_number`, `reason`, `client_message`, `agent_message` |
| `skip_turn` | Agent pausiert, wartet | `reason` |
| `play_keypad_touch_tone` | DTMF-Töne senden | `dtmf_tones` (0-9, *, #, w/W für Pausen) |
| `voicemail_detection` | Mailbox erkennen | `reason` |

### 3. Client Tools
Browser/App-seitige Aktionen. Tool- und Parameter-Namen sind **case-sensitive**.

### 4. MCP Tools (NEU)
Model Context Protocol Integration direkt im Agent.
- Transport: SSE oder HTTP streamable
- Approval Modes: Always Ask (empfohlen), Fine-Grained, No Approval
- Nicht verfügbar bei Zero Retention oder HIPAA

### 5. Tool-Optionen (alle Typen)

| Option | Werte | Standard |
|--------|-------|---------|
| `response_timeout_secs` | 5-60 | 20 |
| `execution_mode` | `immediate`, `post_tool_speech`, `async` | `immediate` |
| `tool_call_sound` | `typing`, `elevator1-4` | - |
| `tool_call_sound_behavior` | `auto`, `always` | `auto` |
| `disable_interruptions` | true/false | false |
| `force_pre_tool_speech` | true/false | false |
| `tool_error_handling_mode` | `auto`, `summarized`, `passthrough`, `hide` | `auto` |
| `assignments` | Array | [] |

> **WICHTIG**: Das `tools`-Array wird bei PATCH komplett ersetzt! Immer ALLE Tools mitschicken.

---

## Webhooks konfigurieren

### Post-Call Webhooks (3 Typen)

**1. Transcription** (`post_call_transcription`)
- Vollständige Gesprächsdaten, Transkript, Analyse, Metadaten

**2. Audio** (`post_call_audio`)
- Base64-encoded MP3, minimale Metadaten

**3. Call Initiation Failure** (`call_initiation_failure`) — NEU
- Fehlgeschlagene Anrufe mit Grund (busy, no-answer, unknown)

```json
PATCH /v1/convai/agents/{agent_id}
{
  "platform_settings": {
    "workspace_overrides": {
      "webhooks": {
        "events": ["transcript", "audio"]
      }
    }
  }
}
```

### Conversation Initiation Webhook (Pre-Call)
```json
{
  "platform_settings": {
    "workspace_overrides": {
      "conversation_initiation_client_data_webhook": {
        "url": "https://n8n.example.com/webhook/init",
        "request_headers": {"Authorization": "Bearer TOKEN"}
      }
    },
    "overrides": {
      "conversation_config_override": {
        "agent": {
          "prompt": {"prompt": true},
          "first_message": true
        }
      }
    }
  }
}
```

### Webhook-Sicherheit
- **HMAC Signatur** via `ElevenLabs-Signature` Header
- **IP Whitelisting** nach Region (US, EU, Asia, India)
- Beide Methoden zusammen empfohlen

### Retry-Logik
- HTTP 200 bestätigt Empfang
- Auto-deaktiviert nach 10+ Fehlern wenn letzter Erfolg > 7 Tage

---

## Dynamic Variables

### Syntax
`{{variable_name}}` in Prompts, First Messages und Tool-Configs.

### System-Variablen (automatisch verfügbar)
| Variable | Beschreibung |
|----------|-------------|
| `system__agent_id` | Agent ID |
| `system__conversation_id` | Gesprächs-ID |
| `system__caller_id` | Anrufer-Nummer |
| `system__called_number` | Angerufene Nummer |
| `system__call_duration_secs` | Gesprächsdauer |
| `system__time_utc` | Aktuelle UTC-Zeit |
| `system__time` | Lokale Zeit |
| `system__timezone` | Zeitzone |
| `system__call_sid` | Twilio Call SID |
| `system__agent_turns` | Anzahl Agent-Turns |
| `system__is_text_only` | Nur-Text Modus |
| `system__conversation_history` | Bisheriger Verlauf |

### Secret Variables
Prefix `secret__` — werden NUR in Tool-Headers verwendet, NIE an LLM gesendet.
```json
"request_headers": {
  "Authorization": "Bearer {{secret__api_token}}"
}
```

### Variables aus Tool-Responses updaten
Server Tools können Dynamic Variables aktualisieren via `assignments`:
```json
"assignments": [
  {"variable_name": "customer_name", "description": "Name aus CRM-Antwort"}
]
```
Dot-Notation für verschachtelte Felder: `response.users.0.email`

---

## Conversation Flow Settings

### Turn Timeout
1-30 Sekunden. Kurz (5-10s) für Smalltalk, lang (10-30s) für komplexe Themen.

### Turn Eagerness (NEU)
| Modus | Verhalten | Use Case |
|-------|-----------|----------|
| `eager` | Antwortet schnell | Kundenservice |
| `normal` | Balanced | Standard |
| `patient` | Wartet bis User fertig | Datenerfassung, Formulare |

### Soft Timeout (NEU)
Audio-Filler wenn LLM langsam (0.5-8.0s). Optionen:
- Statisch: "Hhmmmm...ja."
- LLM-generiert: Kontextbezogener Füller

### Interruptions
Aktivieren für Kundenservice, deaktivieren für kritische Infos (Rechtliches, Sicherheit).

---

## Agent Workflows (NEU)

Visueller Workflow-Builder für komplexe Gesprächsflüsse.

### Node-Typen
| Typ | Beschreibung |
|-----|-------------|
| **Subagent** | Eigener Prompt, LLM, Voice, Knowledge Base, Tools |
| **Dispatch Tool** | Tool-Call mit Erfolg/Fehler-Branching |
| **Agent Transfer** | Übergabe an anderen Agent |
| **Transfer to Number** | Eskalation an Mensch |
| **End** | Gespräch beenden |

### Edge-Typen
- **Forward:** Weiter zum nächsten Node
- **Backward:** Loop zurück (Retries)
- **Conditional:** LLM-Condition (natürliche Sprache), Expression (deterministisch), oder Unconditional

---

## Agent Transfer

Multi-Agent Hierarchie mit verschachtelten Transfers.

Konfiguration pro Transfer:
- Ziel-Agent
- Natürlichsprachliche Bedingung
- Verzögerung (ms)
- Transfer-Nachricht (Audio während Übergabe)
- First Message bei Ziel-Agent aktivieren

**Kontext:** Vollständiges Transkript bleibt über alle Agents erhalten.
**Vererbt:** Client Events, TTS/ASR Format, Sprache, Post-Call Webhooks.
**Nicht vererbt:** Prompt, First Message, LLM, Voice, Tools.

---

## Transfer to Number

3 Transfer-Methoden:
| Methode | Beschreibung |
|---------|-------------|
| **Conference** (Standard) | Teilnehmer zur Konferenz hinzufügen, KI entfernen |
| **Blind Transfer** | Direkter Transfer, Caller-ID bleibt. Braucht native Twilio |
| **SIP REFER** | Direkter SIP-Transfer. Braucht SIP Trunk |

---

## Phone Numbers & Telephony

### Twilio Native
- Gekaufte Nummern importieren (Inbound + Outbound)
- Verifizierte Caller IDs (nur Outbound)

### SIP Trunking
- TCP (Port 5060) oder TLS (Port 5061)
- Digest oder ACL Authentication
- URI: `sip:identifier@sip.rtc.elevenlabs.io:5060`
- Kompatibel: Twilio, Vonage, RingCentral, Sinch, Infobip, Telnyx, Plivo

### Batch Calls (NEU)
- CSV/XLS Upload mit Empfängerliste
- Dynamic Variables pro Empfänger via Spalten
- Scheduling: Sofort oder geplant mit Zeitzone
- Override: `language`, `first_message`, `system_prompt`, `voice_id`

---

## WhatsApp Integration (NEU)

- WhatsApp Business Accounts verbinden
- **Inbound:** Text, Audio (auto-transkribiert), Bilder, Dokumente, Standort
- **Outbound:** Braucht vorab erstellte Templates im WhatsApp Manager
- **Voice Calls:** Inbound unterstützt, Outbound braucht User-Permission
- Dynamic Variables: `system__caller_id` (WhatsApp User ID)

---

## Versioning & Experiments (NEU)

### Versioning
- Unveränderliche Snapshots (`agtvrsn_xxxx`)
- Git-ähnliches Branch-Modell mit "Main" Branch
- Traffic-Splitting nach Prozent (deterministisch per Conversation ID)
- **Permanent:** Kann nicht deaktiviert werden

### Experiments (A/B Testing)
- Baut auf Versioning auf
- Teste: Prompts, Workflows, Voice, Tools, Knowledge Base, LLM
- Start mit 5-10% Traffic, hochskalieren
- Messbar: CSAT, Conversion, Handling Time, Latenz, Cost per Resolution

---

## Personalization & Overrides

### 3 Methoden
1. **Dynamic Variables** — Runtime-Werte injizieren (empfohlen)
2. **Overrides** — System Prompt, First Message, Sprache, Voice, LLM, Speed ersetzen
3. **Twilio Personalization** — Webhook-basiert für Inbound Calls

### Override-Felder (10 Stück)
System Prompt, First Message, Language, Voice ID, LLM, Text-only, Stability (0-1), Speed (0.7-1.2), Similarity Boost (0-1). Müssen im Security-Tab explizit aktiviert werden.

---

## Knowledge Base

- **Formate:** PDF, TXT, DOCX, HTML, EPUB (max 21MB/Datei)
- **Limit:** 20MB oder 300k Zeichen (non-Enterprise)
- **Quellen:** Upload, URL Import, manueller Text
- **Best Practice:** Große Dokumente aufteilen, regelmäßig aktualisieren

---

## Security & Auth

### Signed URLs (empfohlen)
Server holt temporären Token (15 Min Gültigkeit), Client nutzt ihn.

### Allowlists
Bis zu 10 Hostnames. Subdomains brauchen separate Einträge.

**Regel:** Nur EINE Methode pro Agent konfigurieren.

---

## CLI Tool (NEU)

```bash
npm install -g @elevenlabs/cli

# Agent initialisieren
elevenlabs agents init

# Agent hinzufügen
elevenlabs agents add

# Agent deployen
elevenlabs agents push
```

Templates: `default`, `minimal`, `voice-only`, `text-only`, `customer-service`, `assistant`

---

## MCP Server Installation

```bash
# Via uvx (empfohlen)
uvx elevenlabs-mcp

# Env Variables
ELEVENLABS_API_KEY=your_key
ELEVENLABS_MCP_BASE_PATH=/output/path
ELEVENLABS_MCP_OUTPUT_MODE=files  # files | resources | both
```

**Version:** v0.9.1 (Stand Januar 2026)

---

## Real-Time Monitoring (Enterprise)

WebSocket: `wss://api.elevenlabs.io/v1/convai/conversations/{conversation_id}/monitor`

**Befehle:** `end_call`, `transfer_to_number`, `contextual_update`, `enable_human_takeover`, `send_human_message`

---

## n8n Integration Patterns

### Outbound Call via n8n
```json
POST https://api.elevenlabs.io/v1/convai/twilio/outbound-call
Header: xi-api-key: YOUR_KEY
{
  "agent_id": "AGENT_ID",
  "agent_phone_number_id": "PHONE_ID",
  "to_number": "+491234567890",
  "conversation_initiation_client_data": {
    "dynamic_variables": {
      "name": "Max Mustermann",
      "lead_id": "12345"
    }
  }
}
```

### Post-Call Webhook in n8n
n8n Webhook empfängt nach jedem Anruf:
- Transkript
- Gesprächsdauer
- Agent/User Turns
- Tool-Calls
- Analyse-Ergebnisse

### Mid-Call Tool (Server Tool → n8n)
Agent ruft während des Gesprächs einen n8n Webhook auf:
- Kalender prüfen
- CRM updaten
- Daten abfragen
