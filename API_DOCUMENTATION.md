# API/MCP Documentation - noe.ai

## Spis treści / Table of Contents

1. [Wprowadzenie / Introduction](#wprowadzenie--introduction)
2. [Uwierzytelnianie / Authentication](#uwierzytelnianie--authentication)
3. [Punkty końcowe API / API Endpoints](#punkty-końcowe-api--api-endpoints)
4. [Model Context Protocol (MCP)](#model-context-protocol-mcp)
5. [Przykłady użycia / Usage Examples](#przykłady-użycia--usage-examples)
6. [Kody błędów / Error Codes](#kody-błędów--error-codes)
7. [Limity i ograniczenia / Limits and Restrictions](#limity-i-ograniczenia--limits-and-restrictions)

---

## Wprowadzenie / Introduction

API noe.ai umożliwia integrację z platformą poprzez standardowe protokoły REST oraz Model Context Protocol (MCP). Niniejsza dokumentacja opisuje dostępne punkty końcowe, metody uwierzytelniania oraz przykłady użycia.

**Base URL:** `https://api.noe.ai/v1`

---

## Uwierzytelnianie / Authentication

### API Key Authentication

Wszystkie żądania wymagają klucza API przekazywanego w nagłówku:

```http
Authorization: Bearer YOUR_API_KEY
```

### Przykład / Example

```bash
curl -H "Authorization: Bearer sk-noeai-abc123xyz..." \
     https://api.noe.ai/v1/status
```

---

## Punkty końcowe API / API Endpoints

### 1. Status API

**Endpoint:** `GET /status`

**Opis:** Sprawdza status API i jego dostępność.

**Przykład żądania:**
```bash
curl -X GET https://api.noe.ai/v1/status \
     -H "Authorization: Bearer YOUR_API_KEY"
```

**Przykład odpowiedzi:**
```json
{
  "status": "operational",
  "version": "1.0.0",
  "timestamp": "2026-01-30T05:48:00Z"
}
```

---

### 2. Zapytania do modelu / Model Queries

**Endpoint:** `POST /chat/completions`

**Opis:** Wysyła zapytanie do modelu AI i otrzymuje odpowiedź.

**Parametry:**
- `model` (string, required) - Nazwa modelu (np. "gpt-4", "claude-3")
- `messages` (array, required) - Tablica wiadomości w konwersacji
- `temperature` (float, optional) - Temperatura generowania (0.0-2.0)
- `max_tokens` (integer, optional) - Maksymalna liczba tokenów w odpowiedzi

**Przykład żądania:**
```bash
curl -X POST https://api.noe.ai/v1/chat/completions \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "model": "gpt-4",
       "messages": [
         {"role": "system", "content": "Jesteś pomocnym asystentem AI."},
         {"role": "user", "content": "Wyjaśnij czym jest machine learning."}
       ],
       "temperature": 0.7,
       "max_tokens": 500
     }'
```

**Przykład odpowiedzi:**
```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1706587680,
  "model": "gpt-4",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Machine learning to dziedzina sztucznej inteligencji..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 150,
    "total_tokens": 175
  }
}
```

---

### 3. Embedding

**Endpoint:** `POST /embeddings`

**Opis:** Generuje wektory embedding dla podanego tekstu.

**Parametry:**
- `model` (string, required) - Model embedding (np. "text-embedding-ada-002")
- `input` (string lub array, required) - Tekst do zakodowania

**Przykład żądania:**
```bash
curl -X POST https://api.noe.ai/v1/embeddings \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "model": "text-embedding-ada-002",
       "input": "Przykładowy tekst do zakodowania"
     }'
```

**Przykład odpowiedzi:**
```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "index": 0,
      "embedding": [0.0023, -0.0091, 0.0045, ...]
    }
  ],
  "model": "text-embedding-ada-002",
  "usage": {
    "prompt_tokens": 8,
    "total_tokens": 8
  }
}
```

---

## Model Context Protocol (MCP)

### Czym jest MCP?

Model Context Protocol (MCP) to standaryzowany protokół komunikacji między modelami AI a kontekstem aplikacji. Umożliwia:
- Udostępnianie kontekstu aplikacji modelom AI
- Zarządzanie historią konwersacji
- Integrację z narzędziami zewnętrznymi

### Konfiguracja MCP

**Endpoint:** `POST /mcp/initialize`

**Przykład inicjalizacji sesji MCP:**
```bash
curl -X POST https://api.noe.ai/v1/mcp/initialize \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "client_info": {
         "name": "my-app",
         "version": "1.0.0"
       },
       "capabilities": {
         "tools": ["web_search", "code_execution"],
         "context_windows": 8192
       }
     }'
```

**Przykład odpowiedzi:**
```json
{
  "session_id": "mcp_sess_abc123xyz",
  "server_info": {
    "name": "noe.ai MCP Server",
    "version": "1.0.0"
  },
  "capabilities": {
    "supported_tools": ["web_search", "code_execution", "file_operations"],
    "max_context_window": 128000
  }
}
```

### Wysyłanie żądań MCP

**Endpoint:** `POST /mcp/send`

**Przykład:**
```bash
curl -X POST https://api.noe.ai/v1/mcp/send \
     -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{
       "session_id": "mcp_sess_abc123xyz",
       "method": "tools/call",
       "params": {
         "name": "web_search",
         "arguments": {
           "query": "najnowsze wiadomości AI"
         }
       }
     }'
```

---

## Przykłady użycia / Usage Examples

### Python

```python
import requests
import json

API_KEY = "sk-noeai-your-api-key"
BASE_URL = "https://api.noe.ai/v1"

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# Przykład: Chat completion
def chat_completion(message):
    payload = {
        "model": "gpt-4",
        "messages": [
            {"role": "user", "content": message}
        ],
        "temperature": 0.7
    }
    
    response = requests.post(
        f"{BASE_URL}/chat/completions",
        headers=headers,
        json=payload
    )
    
    return response.json()

# Użycie
result = chat_completion("Napisz krótki wiersz o AI")
print(result["choices"][0]["message"]["content"])
```

### JavaScript/Node.js

```javascript
const axios = require('axios');

const API_KEY = 'sk-noeai-your-api-key';
const BASE_URL = 'https://api.noe.ai/v1';

const headers = {
  'Authorization': `Bearer ${API_KEY}`,
  'Content-Type': 'application/json'
};

// Przykład: Chat completion
async function chatCompletion(message) {
  const payload = {
    model: 'gpt-4',
    messages: [
      { role: 'user', content: message }
    ],
    temperature: 0.7
  };
  
  try {
    const response = await axios.post(
      `${BASE_URL}/chat/completions`,
      payload,
      { headers }
    );
    
    return response.data;
  } catch (error) {
    console.error('Error:', error.response?.data || error.message);
    throw error;
  }
}

// Użycie
chatCompletion('Napisz krótki wiersz o AI')
  .then(result => {
    console.log(result.choices[0].message.content);
  });
```

### cURL

```bash
# Podstawowe zapytanie
curl -X POST https://api.noe.ai/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "messages": [
      {"role": "user", "content": "Hello, world!"}
    ]
  }'

# Streaming response
curl -X POST https://api.noe.ai/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "messages": [{"role": "user", "content": "Count to 10"}],
    "stream": true
  }'
```

---

## Kody błędów / Error Codes

| Kod | Znaczenie | Opis |
|-----|-----------|------|
| 200 | OK | Żądanie zakończone sukcesem |
| 400 | Bad Request | Nieprawidłowe parametry żądania |
| 401 | Unauthorized | Brak lub nieprawidłowy klucz API |
| 403 | Forbidden | Brak uprawnień do zasobu |
| 404 | Not Found | Zasób nie został znaleziony |
| 429 | Too Many Requests | Przekroczono limit żądań |
| 500 | Internal Server Error | Błąd serwera |
| 503 | Service Unavailable | Usługa tymczasowo niedostępna |

### Przykład odpowiedzi błędu:

```json
{
  "error": {
    "message": "Invalid API key provided",
    "type": "invalid_request_error",
    "code": "invalid_api_key"
  }
}
```

---

## Limity i ograniczenia / Limits and Restrictions

### Rate Limiting

- **Darmowy plan:** 60 żądań/minutę
- **Pro plan:** 600 żądań/minutę
- **Enterprise:** Bez limitów

### Limity tokenów

- **Maksymalna długość zapytania:** 128,000 tokenów (zależnie od modelu)
- **Maksymalna długość odpowiedzi:** Konfigurowalna poprzez parametr `max_tokens`

### Nagłówki odpowiedzi

Każda odpowiedź zawiera informacje o limitach:

```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1706587800
```

---

## Wsparcie / Support

- **Email:** support@noe.ai
- **Dokumentacja:** https://docs.noe.ai
- **Status:** https://status.noe.ai
- **GitHub:** https://github.com/noeai/api

---

## Changelog

### v1.0.0 (2026-01-30)
- Początkowa wersja dokumentacji API
- Wsparcie dla MCP (Model Context Protocol)
- Podstawowe endpointy: chat completions, embeddings
- Przykłady w Python, JavaScript, cURL

---

## Licencja / License

Dokumentacja dostępna na licencji MIT. API podlega warunkom użytkowania noe.ai.
