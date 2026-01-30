[← Spis treści](README.md)

# Flow (Connect::Flow)

Przepływy danych (pipeline'y) łączące [transformacje](transforms.md) i [konektory](connectors.md).

## API

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/connect/flows.json` | Lista flow |
| GET | `/connect/flows/:id.json` | Szczegóły |
| POST | `/connect/flows.json` | Utworzenie |
| PATCH | `/connect/flows/:id.json` | Aktualizacja |
| DELETE | `/connect/flows/:id.json` | Usunięcie |
| POST | `/connect/flows/:id/start` | Uruchomienie flow |

## Utworzenie Custom Flow

```
POST /connect/flows.json?api_token=TOKEN
Content-Type: application/json

{
  "flow": {
    "name": "Mój przepływ danych",
    "code": "my-data-flow",
    "kind": "noe/custom_flow",
    "active": true,
    "fields": {
      "connector_code": "my-api",
      "program": [
        {
          "id": "step1",
          "action": "connector",
          "in": "json_object",
          "kind": "noe/custom_connector",
          "code": "{connector_code}",
          "method": "get_users",
          "out": "json_array"
        },
        {
          "id": "step2",
          "action": "each",
          "in": "[json_object]",
          "each": [
            {
              "action": "mutation",
              "in": "json_object",
              "mutation": {
                "json_object.processed": true,
                "json_object.timestamp": "'{Time.now}'"
              },
              "out": "json_object"
            }
          ],
          "out": "[json_object]"
        }
      ]
    }
  }
}
```

## Pola flow

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `name` | string | tak | Nazwa flow |
| `code` | string | tak | Unikalny kod (slug) |
| `kind` | string | tak | `noe/custom_flow` dla własnych |
| `active` | boolean | nie | Czy aktywny |
| `fields.program` | array | tak | Tablica instrukcji (InFlow) |
| `fields.*` | any | nie | Zmienne dostępne w programie przez `{nazwa}` |

---

## Instrukcje programu (InFlow)

### Connector - wywołanie konektora

```json
{
  "action": "connector",
  "in": "json_object",
  "kind": "noe/custom_connector",
  "code": "my-api",
  "method": "get_user",
  "out": "json_object"
}
```

### Transform - transformacja danych

```json
{
  "action": "transform",
  "in": "invoice_json",
  "out": "invoice_xml"
}
```

### Mutation - modyfikacja danych

```json
{
  "action": "mutation",
  "in": "json_object",
  "mutation": {
    "json_object.new_field": "'stała wartość'",
    "json_object.copied": "{json_object.original}",
    "json_object.calculated": "{json_object.price} * 1.23"
  },
  "out": "json_object"
}
```

### Each - iteracja po tablicy

```json
{
  "action": "each",
  "in": "[json_object]",
  "each": [
    { "action": "mutation", "in": "json_object", "mutation": {}, "out": "json_object" }
  ],
  "out": "[json_object]"
}
```

### If - warunek

```json
{
  "in": "json_object",
  "out": "json_object",
  "if": {
    "{json_object.status} == 'active'": {
      "then": [
        { "action": "mutation", "in": "json_object", "mutation": {"json_object.processed": true}, "out": "json_object" }
      ],
      "else": [
        { "action": "mutation", "in": "json_object", "mutation": {"json_object.skipped": true}, "out": "json_object" }
      ]
    }
  }
}
```

### Case - wielokrotny warunek

```json
{
  "action": "case",
  "in": "json_object",
  "case": {
    "condition": "{json_object.type}",
    "when": {
      "invoice": [{ "action": "transform", "in": "json_object", "out": "invoice_xml" }],
      "order": [{ "action": "transform", "in": "json_object", "out": "order_xml" }]
    },
    "else": [{ "action": "mutation", "in": "json_object", "mutation": {"json_object.error": "'unknown type'"}, "out": "json_object" }]
  },
  "out": "xml_document"
}
```

---

## Uruchomienie Flow

```
POST /connect/flows/my-data-flow/start?api_token=TOKEN
Content-Type: application/json

{
  "param1": "value1",
  "param2": "value2"
}
```

**Odpowiedź:**
```json
{
  "flow_id": 123,
  "flow_process": {
    "id": "uuid",
    "status": "pending"
  }
}
```

---

## Przykłady

### Przykład 1: Flow pogodowy

```json
{
  "flow": {
    "name": "Get Weather",
    "code": "get-weather",
    "kind": "noe/custom_flow",
    "active": true,
    "fields": {
      "program": [
        {
          "action": "connector",
          "in": "json_object",
          "kind": "noe/custom_connector",
          "code": "open-meteo",
          "method": "get_weather",
          "out": "json_object"
        },
        {
          "action": "mutation",
          "in": "json_object",
          "mutation": {
            "json_object.temperature": "{json_object.current_weather.temperature}",
            "json_object.windspeed": "{json_object.current_weather.windspeed}"
          },
          "out": "json_object"
        }
      ]
    }
  }
}
```

### Przykład 2: Flow eksportu faktur

```json
{
  "flow": {
    "name": "Export Invoice to XML",
    "code": "export-invoice-xml",
    "kind": "noe/custom_flow",
    "active": true,
    "fields": {
      "program": [
        {
          "action": "transform",
          "in": "invoice_json",
          "out": "invoice_xml"
        }
      ]
    }
  }
}
```

### Przykład 3: Flow synchronizacji CRM

```json
{
  "flow": {
    "name": "Sync Contact to CRM",
    "code": "sync-contact-crm",
    "kind": "noe/custom_flow",
    "active": true,
    "fields": {
      "program": [
        {
          "action": "transform",
          "in": "internal_contact",
          "out": "crm_contact"
        },
        {
          "action": "connector",
          "in": "crm_contact",
          "kind": "noe/custom_connector",
          "code": "mycrm",
          "method": "create_contact",
          "out": "json_object"
        }
      ]
    }
  }
}
```

---

## Wskazówki

1. **Zmienne:** Wszystkie pola z `fields` są dostępne przez `{nazwa}` w programie
2. **Debugowanie:** Dodaj `?perform_now=1` do uruchomienia synchronicznego (tylko dev/test)
3. **Polling:** Dla długich operacji sprawdzaj status przez endpoint akcji [aplikacji](apps.md#akcje-aplikacji)
4. **Testowanie transformacji:** Użyj `/noe/transforms/:id/test.json` przed podłączeniem do flow
