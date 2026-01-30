[← Spis treści](README.md)

# Bazy danych (Noe::Db)

Proste bazy danych dla aplikacji Noe z dynamicznym schematem i indeksowanymi polami.

**Uwaga:** API baz danych wymaga autoryzacji (zalogowany użytkownik lub `api_token`). Dla publicznych aplikacji użyj [dostępu publicznego](#rekordy---dostęp-publiczny).

## API

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/noe/dbs.json` | Lista baz |
| GET | `/noe/dbs/:id.json` | Szczegóły bazy |
| POST | `/noe/dbs.json` | Utworzenie bazy |
| PATCH | `/noe/dbs/:id.json` | Aktualizacja |
| DELETE | `/noe/dbs/:id.json` | Usunięcie |

## Utworzenie bazy danych

```
POST /noe/dbs.json?api_token=TOKEN
Content-Type: application/json

{
  "db": {
    "name": "Kontakty",
    "code": "contacts",
    "app_id": "uuid-aplikacji-opcjonalne",
    "schema": {
      "fields": {
        "email": { "type": "string", "required": true },
        "name": { "type": "string" },
        "age": { "type": "integer" },
        "joined_at": { "type": "datetime" }
      },
      "indexes": {
        "idx_str_1": "email",
        "idx_str_2": "name",
        "idx_int_1": "age",
        "idx_time_1": "joined_at"
      }
    }
  }
}
```

## Pola bazy danych

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `name` | string | tak | Nazwa bazy |
| `code` | string | nie | Unikalny kod (slug) do użycia w URL |
| `app_id` | uuid | nie | ID aplikacji Noe, do której należy baza |
| `schema` | object | nie | Definicja pól i indeksów |
| `public_methods` | object | nie | Konfiguracja publicznych metod (bez autoryzacji) |

## Schema - struktura

Schema definiuje strukturę danych i mapowanie na indeksy:

```json
{
  "fields": {
    "email": { "type": "string", "required": true },
    "name": { "type": "string" },
    "age": { "type": "integer" },
    "joined_at": { "type": "datetime" }
  },
  "indexes": {
    "idx_str_1": "email",
    "idx_str_2": "name",
    "idx_int_1": "age",
    "idx_time_1": "joined_at"
  }
}
```

### Dostępne indeksy

Każdy rekord ma 7 kolumn indeksowanych do szybkiego wyszukiwania:

| Indeks | Typ | Opis |
|--------|-----|------|
| `idx_str_1` | string | Pole tekstowe 1 (max 255 znaków) |
| `idx_str_2` | string | Pole tekstowe 2 |
| `idx_str_3` | string | Pole tekstowe 3 |
| `idx_int_1` | bigint | Pole liczbowe 1 |
| `idx_int_2` | bigint | Pole liczbowe 2 |
| `idx_time_1` | datetime | Pole daty/czasu 1 |
| `idx_time_2` | datetime | Pole daty/czasu 2 |

### Typy pól

| Typ | Opis | Przykład | Indeks |
|-----|------|----------|--------|
| `string` | Tekst (max 255 znaków dla indeksu) | `"Jan Kowalski"` | `idx_str_*` |
| `text` | Długi tekst | `"Długi opis..."` | `idx_str_*` |
| `integer` | Liczba całkowita | `42` | `idx_int_*` |
| `number` | Liczba (alias dla integer) | `3.14` | `idx_int_*` |
| `boolean` | Wartość logiczna | `true`, `false` | - |
| `datetime` | Data i czas | `"2024-06-15T14:30:00Z"` | `idx_time_*` |
| `date` | Tylko data | `"2024-06-15"` | `idx_time_*` |
| `time` | Tylko czas | `"14:30:00"` | `idx_time_*` |

---

## Rekordy - dostęp publiczny

Pole `public_methods` pozwala udostępnić wybrane operacje **bez autoryzacji**. Idealne dla publicznych aplikacji.

**URL:** `POST /noe/db/:db_code/:method_name`

### Dostępne operacje

| Operacja | Opis | Parametry |
|----------|------|-----------|
| `list` | Lista rekordów | `limit`, `offset` |
| `get` | Pojedynczy rekord | `id` |
| `create` | Utworzenie rekordu | `{ data: {...} }` |
| `update` | Aktualizacja | `{ id, data: {...} }` |
| `delete` | Usunięcie | `{ id }` |
| `search` | Wyszukiwanie | `q`, `limit`, `offset`, `sort`, `filter[field]` |

### Opcje konfiguracji

| Opcja | Opis |
|-------|------|
| `true` | Metoda dostępna bez ograniczeń |
| `allowed_fields` | Tylko te pola mogą być przekazane (reszta ignorowana) |
| `required_fields` | Wymagane pola - błąd jeśli brakuje |
| `default_values` | Wartości domyślne dodawane automatycznie |
| `allowed_params` | Dozwolone parametry (dla search) |
| `auto_fields` | Pola ustawiane automatycznie, np. `{ "user_ip": "{ip}" }` |
| `scope_by` | Filtrowanie wyników po polu (np. po IP użytkownika) |

### Zmienne w auto_fields

- `{ip}` - adres IP użytkownika
- `{cookie_key}` - unikalny token per przeglądarka (cookie z miesięcznym czasem życia, 32 znaki)

**Jak działa scope_by:** Automatycznie wykrywa zmienną z `auto_fields` dla danego pola. Jeśli `scope_by: "user_ip"` i w `auto_fields` jest `"user_ip": "{ip}"`, to filtrowanie użyje IP użytkownika. Analogicznie dla `{cookie_key}`.

### Przykład - baza z publicznymi metodami

```json
{
  "db": {
    "name": "Todo",
    "code": "todo",
    "schema": {
      "fields": {
        "title": { "type": "string", "required": true },
        "done": { "type": "integer" },
        "user_ip": { "type": "string" }
      },
      "indexes": {
        "idx_str_1": "title",
        "idx_str_2": "user_ip",
        "idx_int_1": "done"
      }
    },
    "public_methods": {
      "list": { "scope_by": "user_ip" },
      "create": {
        "allowed_fields": ["title", "done"],
        "required_fields": ["title"],
        "default_values": { "done": 0 },
        "auto_fields": { "user_ip": "{ip}" }
      },
      "update": {
        "allowed_fields": ["title", "done"],
        "scope_by": "user_ip"
      },
      "delete": { "scope_by": "user_ip" }
    }
  }
}
```

### Wywołanie z frontendu

```javascript
const DB = "todo";

// Lista (filtrowana po IP użytkownika)
const tasks = await fetch(`/noe/db/${DB}/list`, { method: "POST" }).then(r => r.json());

// Utworzenie (user_ip ustawiane automatycznie)
await fetch(`/noe/db/${DB}/create`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ data: { title: "Nowe zadanie" } })
});

// Aktualizacja (tylko własne rekordy)
await fetch(`/noe/db/${DB}/update`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ id: "uuid", data: { done: 1 } })
});

// Usunięcie (tylko własne rekordy)
await fetch(`/noe/db/${DB}/delete`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ id: "uuid" })
});
```

---

## Rekordy - dostęp prywatny

Operacje na rekordach wymagające autoryzacji (`api_token` lub zalogowany użytkownik).

**URL bazowy:** `/noe/dbs/:db_code/records`

| Metoda HTTP | Ścieżka | Opis |
|-------------|---------|------|
| GET | `/noe/dbs/:db/records.json` | Lista rekordów |
| GET | `/noe/dbs/:db/records/:id.json` | Pojedynczy rekord |
| POST | `/noe/dbs/:db/records.json` | Utworzenie rekordu |
| PATCH | `/noe/dbs/:db/records/:id.json` | Aktualizacja rekordu |
| DELETE | `/noe/dbs/:db/records/:id.json` | Usunięcie rekordu |
| GET | `/noe/dbs/:db/search.json` | Wyszukiwanie rekordów |

### Utworzenie rekordu

```
POST /noe/dbs/contacts/records.json?api_token=TOKEN
Content-Type: application/json

{
  "record": {
    "data": {
      "email": "jan@example.com",
      "name": "Jan Kowalski",
      "age": 35
    }
  }
}
```

### Aktualizacja rekordu

```
PATCH /noe/dbs/contacts/records/123.json?api_token=TOKEN
Content-Type: application/json

{
  "record": {
    "data": {
      "email": "jan.nowy@example.com",
      "age": 36
    }
  }
}
```

---

## Search API

Wyszukiwanie rekordów w bazie danych.

**Endpoint:** `GET /noe/dbs/:db_id/search.json`

### Parametry

| Parametr | Typ | Opis |
|----------|-----|------|
| `q` | string | Zapytanie wyszukiwania. Obsługuje `*` jako wildcard. |
| `table_name` | string | Nazwa tabeli (dla multi-table schema) |
| `search_in` | string | Pola do przeszukania (domyślnie indexed_fields). Format: `name,email` |
| `fields` | string | Pola do zwrócenia (domyślnie wszystkie). Format: `name,email` |
| `filter[field]` | string | Filtr na polu. Obsługuje `*` jako wildcard. |
| `sort` | string | Pole sortowania. Prefiks `-` dla DESC, np. `-created_at` |
| `limit` | integer | Limit wyników (1-100, domyślnie 20) |
| `offset` | integer | Offset wyników (domyślnie 0) |

### Przykłady

```bash
# Proste wyszukiwanie
GET /noe/dbs/contacts/search.json?q=jan

# Wyszukiwanie z wildcard
GET /noe/dbs/contacts/search.json?q=jan*

# Filtrowanie po polu
GET /noe/dbs/contacts/search.json?filter[status]=active

# Filtrowanie z wildcard
GET /noe/dbs/contacts/search.json?filter[email]=*@example.com

# Wiele filtrów + sortowanie
GET /noe/dbs/contacts/search.json?filter[status]=active&filter[age]=30&sort=-created_at

# Wybór pól w odpowiedzi
GET /noe/dbs/contacts/search.json?fields=id,email,name&limit=10
```

### Różnica: Search API vs Records API

**WAŻNE:** Search API i Records API zwracają dane w **różnych formatach**!

| API | Endpoint | Struktura danych |
|-----|----------|------------------|
| Records API | `/noe/dbs/:db/records.json` | Zagnieżdżona: `record.data.field` |
| Search API | `/noe/dbs/:db/search.json` | Płaska: `record.field` |

**Records API** - dane zagnieżdżone w `data`:
```json
{ "id": "uuid", "data": { "title": "Zadanie", "done": false }, "created_at": "..." }
```

**Search API** - dane płaskie:
```json
{ "id": "uuid", "title": "Zadanie", "done": false, "created_at": "..." }
```

**Tworzenie/aktualizacja** - zawsze używaj zagnieżdżonej struktury:
```javascript
await fetch(`/noe/dbs/${DB}/records.json`, {
  method: 'POST',
  body: JSON.stringify({ record: { data: { title: "Nowe zadanie" } } })
});
```

---

## Przykład: Aplikacja Todo

Kompletny przykład publicznej aplikacji Todo używającej **bazy danych z public_methods**.

### 1. Baza danych

```json
POST /noe/dbs.json?api_token=TOKEN
{
  "db": {
    "name": "Todo Tasks",
    "code": "todo-tasks",
    "schema": {
      "fields": {
        "title": { "type": "string", "required": true },
        "done": { "type": "integer" }
      },
      "indexes": {
        "idx_str_1": "title",
        "idx_int_1": "done"
      }
    },
    "public_methods": {
      "list": true,
      "create": {
        "allowed_fields": ["title", "done"],
        "required_fields": ["title"],
        "default_values": { "done": 0 }
      },
      "update": { "allowed_fields": ["title", "done"] },
      "delete": true
    }
  }
}
```

### 2. Aplikacja Svelte

```svelte
<script>
  let tasks = $state([]);
  let newTask = $state("");
  let loading = $state(false);

  const DB = "todo-tasks";

  async function callDb(method, body = null) {
    const options = { method: "POST", headers: { "Content-Type": "application/json" } };
    if (body) options.body = JSON.stringify(body);
    return await fetch(`/noe/db/${DB}/${method}`, options).then(r => r.json());
  }

  async function loadTasks() {
    loading = true;
    tasks = await callDb("list");
    loading = false;
  }

  async function addTask() {
    if (!newTask.trim()) return;
    await callDb("create", { data: { title: newTask } });
    newTask = "";
    await loadTasks();
  }

  async function toggleTask(task) {
    await callDb("update", { id: task.id, data: { done: task.data.done ? 0 : 1 } });
    await loadTasks();
  }

  async function deleteTask(task) {
    await callDb("delete", { id: task.id });
    await loadTasks();
  }

  $effect(() => { loadTasks(); });
</script>

<div class="max-w-md mx-auto p-6">
  <h1 class="text-2xl font-bold mb-4">Lista zadań</h1>

  <div class="flex gap-2 mb-4">
    <input bind:value={newTask} onkeydown={(e) => e.key === "Enter" && addTask()}
      placeholder="Nowe zadanie..." class="flex-1 border rounded px-3 py-2" />
    <button onclick={addTask} class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">Dodaj</button>
  </div>

  {#if loading}
    <p class="text-gray-500">Ładowanie...</p>
  {:else if tasks.length === 0}
    <p class="text-gray-500">Brak zadań</p>
  {:else}
    <ul class="space-y-2">
      {#each tasks as task}
        <li class="flex items-center gap-2 p-2 border rounded {task.data.done ? 'bg-gray-100' : ''}">
          <input type="checkbox" checked={task.data.done === 1} onchange={() => toggleTask(task)} class="w-5 h-5" />
          <span class="flex-1 {task.data.done ? 'line-through text-gray-500' : ''}">{task.data.title}</span>
          <button onclick={() => deleteTask(task)} class="text-red-500 hover:text-red-700">✕</button>
        </li>
      {/each}
    </ul>
  {/if}
</div>
```

---

## Przykład: Todo per IP

Aplikacja gdzie każdy użytkownik (IP) widzi tylko swoje zadania.

```json
{
  "db": {
    "name": "Kosmiczne Todo",
    "code": "kosmiczne-todo",
    "schema": {
      "fields": {
        "title": { "type": "string", "required": true },
        "done": { "type": "integer" },
        "user_ip": { "type": "string" }
      },
      "indexes": { "idx_str_1": "title", "idx_str_2": "user_ip", "idx_int_1": "done" }
    },
    "public_methods": {
      "list": { "scope_by": "user_ip" },
      "create": {
        "allowed_fields": ["title", "done"],
        "required_fields": ["title"],
        "default_values": { "done": 0 },
        "auto_fields": { "user_ip": "{ip}" }
      },
      "update": { "allowed_fields": ["title", "done"], "scope_by": "user_ip" },
      "delete": { "scope_by": "user_ip" }
    }
  }
}
```

**Alternatywa z cookie** (trwały identyfikator, lepsze dla mobile):
```json
"auto_fields": { "browser_id": "{cookie_key}" },
"scope_by": "browser_id"
```
