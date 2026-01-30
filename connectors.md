[← Spis treści](README.md)

# Konektory (Connect::Connector)

Połączenia z zewnętrznymi API.

## API

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/connect/connectors.json` | Lista konektorów |
| GET | `/connect/connectors/:id.json` | Szczegóły |
| POST | `/connect/connectors.json` | Utworzenie |
| PATCH | `/connect/connectors/:id.json` | Aktualizacja |
| DELETE | `/connect/connectors/:id.json` | Usunięcie |
| POST | `/noe/connectors/:id/:method_name` | Wykonanie metody |

## Utworzenie Custom Connector

Custom Connector (`noe/custom_connector`) pozwala definiować własne metody HTTP.

```
POST /connect/connectors.json?api_token=TOKEN
Content-Type: application/json

{
  "connector": {
    "name": "Moje API",
    "code": "my-api",
    "kind": "noe/custom_connector",
    "url": "https://api.example.com",
    "secret_token": "API_KEY",
    "active": true,
    "methods": [
      {
        "method": "get_users",
        "object_in": "nil",
        "object_out": "json_array",
        "http_method": "GET",
        "path": "/users",
        "description": "Pobiera listę użytkowników"
      },
      {
        "method": "get_user",
        "object_in": "number",
        "object_out": "json_object",
        "http_method": "GET",
        "path": "/users/{number}",
        "description": "Pobiera użytkownika po ID"
      },
      {
        "method": "create_user",
        "object_in": "json_object",
        "object_out": "json_object",
        "http_method": "POST",
        "path": "/users",
        "description": "Tworzy nowego użytkownika"
      },
      {
        "method": "search",
        "object_in": "json_object",
        "object_out": "json_array",
        "http_method": "GET",
        "path": "/search?q={query}",
        "description": "Wyszukiwanie"
      }
    ]
  }
}
```

## Pola konektora

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `name` | string | tak | Nazwa konektora |
| `code` | string | tak | Unikalny kod (slug) |
| `kind` | string | tak | `noe/custom_connector` dla własnych |
| `url` | string | tak | Bazowy URL API |
| `secret_token` | string | nie | Token autoryzacji (Bearer) |
| `active` | boolean | nie | Czy aktywny |
| `methods` | array | tak | Definicje metod (kolumna jsonb) |

## Definicja metody

| Pole | Typ | Opis |
|------|-----|------|
| `method` | string | Nazwa metody |
| `object_in` | string | Typ wejścia: `nil`, `number`, `json_object`, `json_array` |
| `object_out` | string | Typ wyjścia |
| `http_method` | string | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| `path` | string | Ścieżka URL (może zawierać `{variable}`) |
| `params` | object | Template body dla POST/PUT (z interpolacją `{number}`) |
| `required_status` | int/array | Oczekiwany status HTTP |
| `public` | boolean | Czy metoda jest publicznie dostępna (patrz [Security](#publiczne-metody)) |
| `description` | string | Opis metody |

---

## Interpolacja w ścieżce (path)

Dla `object_in: "json_object"` - dowolne nazwy pól:
- `{id}` - pole `id` z inputu
- `{userId}` - pole `userId` z inputu
- `{any_field}` - dowolne pole z inputu

Dla `object_in: "number"`:
- `{number}` - wartość liczbowa

**Przykład z wieloma parametrami:**
```json
{
  "method": "get_user_post",
  "path": "/users/{userId}/posts/{postId}",
  "object_in": "json_object"
}
```
Wywołanie z `{"userId": 5, "postId": 42}` → GET `/users/5/posts/42`

## Parametry w body (params)

Pole `params` buduje body requestu (POST/PUT). Używa `{number}`, `{input}`, `{value}`:

```json
{
  "method": "send_notification",
  "http_method": "POST",
  "path": "/notifications",
  "object_in": "number",
  "params": {"email_id": "{number}", "action": "notify"}
}
```
Wywołanie z `123` → POST body: `{"email_id": "123", "action": "notify"}`

## JSON object jako body (bez params)

Dla `object_in: "json_object"` bez `params` - cały input jest wysyłany jako body:

```json
{
  "method": "create_user",
  "http_method": "POST",
  "path": "/users",
  "object_in": "json_object"
}
```
Wywołanie z `{"name": "Jan", "email": "jan@test.com"}` → POST body: `{"name": "Jan", "email": "jan@test.com"}`

---

## Publiczne metody

Metody oznaczone jako `public: true` są dostępne **bez autoryzacji** przez URL `/noe/connectors/:code/:method`.

**Uwaga:** Używaj ostrożnie - każdy znający URL może wywołać taką metodę!

### Connector jako proxy dla bazy danych

Przydatne gdy publiczna aplikacja potrzebuje dostępu do [bazy danych](dbs.md):

```json
{
  "connector": {
    "name": "Tasks DB Proxy",
    "code": "tasks-db-proxy",
    "kind": "noe/custom_connector",
    "url": "https://TWOJA_DOMENA",
    "secret_token": "TWOJ_API_TOKEN",
    "active": true,
    "methods": [
      {
        "method": "list_tasks",
        "object_in": "nil",
        "object_out": "json_array",
        "http_method": "GET",
        "path": "/noe/dbs/tasks/records.json",
        "public": true
      },
      {
        "method": "add_task",
        "object_in": "json_object",
        "object_out": "json_object",
        "http_method": "POST",
        "path": "/noe/dbs/tasks/records.json",
        "public": true
      },
      {
        "method": "admin_clear_all",
        "object_in": "nil",
        "object_out": "json_object",
        "http_method": "DELETE",
        "path": "/noe/dbs/tasks.json",
        "description": "Tylko dla zalogowanych"
      }
    ]
  }
}
```

**Wywołanie z frontendu:**

```javascript
// Bezpieczne - api_token jest w secret_token konektora (na serwerze)
const tasks = await fetch('/noe/connectors/tasks-db-proxy/list_tasks', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' }
}).then(r => r.json());
```

**Jak to działa:**
- `secret_token` konektora autoryzuje requesty do API - nigdy nie trafia do przeglądarki
- Metody z `public: true` dostępne bez autoryzacji
- Metody bez `public` wymagają zalogowania lub `api_token`

---

## Przykład: Connector do API pogody

```json
{
  "connector": {
    "name": "Open-Meteo API",
    "code": "open-meteo",
    "kind": "noe/custom_connector",
    "url": "https://api.open-meteo.com/v1",
    "active": true,
    "methods": [
      {
        "method": "get_weather",
        "object_in": "json_object",
        "object_out": "json_object",
        "http_method": "GET",
        "path": "/forecast?latitude={latitude}&longitude={longitude}&current_weather=true",
        "description": "Pobiera aktualną pogodę"
      }
    ]
  }
}
```

## Przykład: Connector do CRM

```json
{
  "connector": {
    "name": "MyCRM",
    "code": "mycrm",
    "kind": "noe/custom_connector",
    "url": "https://api.mycrm.com/v1",
    "secret_token": "crm-api-key",
    "active": true,
    "methods": [
      {
        "method": "list_contacts",
        "object_in": "nil",
        "object_out": "json_array",
        "http_method": "GET",
        "path": "/contacts"
      },
      {
        "method": "create_contact",
        "object_in": "json_object",
        "object_out": "json_object",
        "http_method": "POST",
        "path": "/contacts"
      },
      {
        "method": "update_contact",
        "object_in": "json_object",
        "object_out": "json_object",
        "http_method": "PUT",
        "path": "/contacts/{id}"
      }
    ]
  }
}
```
