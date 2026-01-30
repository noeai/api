[← Spis treści](README.md)

# Aplikacje Noe

Interaktywne aplikacje webowe (Svelte 5, Vue 3, React 19, JS, HTML).

**Autoryzacja:** Wszystkie requesty wymagają `api_token` (parametr GET lub header `Authorization: Bearer TOKEN`)

## Spis treści

1. [API](#api)
2. [Pola aplikacji](#pola-aplikacji)
3. [Silniki aplikacji](#silniki-aplikacji)
4. [Osadzanie w modułach](#osadzanie-aplikacji-w-modułach-show_in_module)
5. [Wstrzykiwanie na strony](#wstrzykiwanie-aplikacji-na-strony)
6. [Kontekst aplikacji](#kontekst-aplikacji-windownoeappcontext)
7. [Akcje aplikacji](#akcje-aplikacji)
8. [Przykłady kodu](#przykład-svelte-5)
9. [Security](#security)
10. [Replace API](#replace-api)
11. [Kompletne przykłady](#przykłady)
12. [Wskazówki](#wskazówki)

---

## API

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/noe/apps.json` | Lista aplikacji |
| GET | `/noe/apps/:id.json` | Szczegóły aplikacji |
| POST | `/noe/apps.json` | Utworzenie aplikacji |
| PATCH | `/noe/apps/:id.json` | Aktualizacja |
| DELETE | `/noe/apps/:id.json` | Usunięcie |
| GET | `/noe/:url_code` | Uruchomienie aplikacji (HTML) |
| POST | `/noe/apps/:id/action/:action_name` | Wywołanie akcji |
| PATCH | `/noe/apps/:id/replace.json` | Częściowa edycja pola ([Replace API](#replace-api)) |

### Utworzenie aplikacji

```
POST /noe/apps.json?api_token=TOKEN
Content-Type: application/json

{
  "app": {
    "name": "Nazwa aplikacji",
    "url_code": "kod-url",
    "description": "Opis",
    "app_engine": "svelte",
    "css_engine": "tailwind_all",
    "public": true,
    "source_code": "KOD_ZRODLOWY",
    "actions": []
  }
}
```

## Pola aplikacji

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `name` | string | tak | Nazwa aplikacji |
| `url_code` | string | nie | Przyjazny URL (np. "pogoda" → `/noe/pogoda`) |
| `db_code` | string | nie | Kod bazy Noe::Db (wiele aplikacji może współdzielić jedną bazę) |
| `description` | string | nie | Opis aplikacji |
| `app_engine` | string | tak | `svelte`, `vue`, `react`, `js`, `html`, `kaboom`, `phaser` |
| `css_engine` | string | tak | `tailwind_all`, `tailwind`, `none` |
| `public` | boolean | nie | `true` = dostępna bez logowania |
| `show_in_module` | string | nie | Osadza aplikację w wybranym module (patrz sekcja poniżej) |
| `source_code` | string | tak | Kod źródłowy |
| `actions` | array | nie | Definicje akcji (do integracji z Flow) |

## Silniki aplikacji

| Silnik | Opis | Przykład |
|--------|------|----------|
| `svelte` | Svelte 5 z $state/$derived | `<script>let count = $state(0);</script>` |
| `vue` | Vue 3 SFC (Composition API) | `<script setup>import { ref } from 'vue';</script>` |
| `react` | React 19 z JSX | `import { useState } from 'react';` |
| `js` | Czysty JavaScript | `document.getElementById("noe-app")` |
| `html` | Czysty HTML (bez kompilacji) | `<div>Hello</div>` |
| `kaboom` | Kaboom.js (gry 2D) | Framework do gier |
| `phaser` | Phaser (gry 2D/3D) | Framework do gier |

### Silniki CSS

| Silnik | Opis |
|--------|------|
| **tailwind_all** | Tailwind CSS z CDN (wszystkie klasy) |
| **tailwind** | Tailwind CSS (kompilowany) |
| **none** | Bez CSS |

---

## Osadzanie aplikacji w modułach (show_in_module)

Pole `show_in_module` pozwala osadzić aplikację Noe wewnątrz wybranego modułu systemu. Aplikacja:
- Pojawia się w menu bocznym modułu
- Wyświetla się z breadcrumbs i layoutem danego modułu
- Jest dostępna pod tym samym URL `/noe/:url_code`

**Dostępne moduły:**

| Wartość | Moduł |
|---------|-------|
| `organize` | Organizacja (zadania, projekty) |
| `mail` | Poczta |
| `crm` | CRM |
| `kb` | Baza wiedzy |
| `cms` | CMS |
| `connect` | Konektory |
| `voip` | VoIP |
| `form` | Formularze |
| `insight` | Analityka |
| `billing` | Rozliczenia |
| `drive` | Dysk |
| `fiskator` | Fiskator |
| `commerce` | E-commerce |
| `account` | Konto |

**Przykład:**

```json
{
  "app": {
    "name": "Kalkulator finansowy",
    "url_code": "kalkulator-finansowy",
    "app_engine": "svelte",
    "css_engine": "tailwind_all",
    "public": true,
    "show_in_module": "cms",
    "source_code": "..."
  }
}
```

**Uwaga:** Jeśli użytkownik nie ma dostępu do modułu, aplikacja wyświetli się w trybie standardowym (bez layoutu modułu).

---

## Wstrzykiwanie aplikacji na strony

Pole `show_in_module` może również zawierać ścieżkę do konkretnej strony (np. `tasks/show`). Wtedy aplikacja zostanie **wstrzyknięta** bezpośrednio w widok tej strony.

| Wartość | Zachowanie |
|---------|------------|
| `organize` | Aplikacja w menu bocznym modułu |
| `tasks/show` | Aplikacja wstrzyknięta na stronie szczegółów zadania |

**Format:** `{controller}/{action}` - np. `tasks/show`, `contacts/index`, `emails/show`

**Przykładowe strony:**

| Wartość | Strona |
|---------|--------|
| `tasks/show` | Szczegóły zadania |
| `tasks/index` | Lista zadań |
| `contacts/show` | Szczegóły kontaktu |
| `contacts/index` | Lista kontaktów |
| `emails/show` | Szczegóły emaila |
| `projects/show` | Szczegóły projektu |

**Uwagi:**
- Wstrzykiwanie wymaga aktywnego modułu Noe na koncie
- Działa na wszystkich stronach systemu (layout tailwind)
- Aplikacja renderuje się inline na stronie (bez osobnego layoutu)
- Można wstrzyknąć wiele aplikacji na tę samą stronę
- Aplikacja ma dostęp do DOM strony i może interagować z jej elementami
- Pole `active` (domyślnie true) kontroluje czy aplikacja jest wstrzykiwana

---

## Kontekst aplikacji (window.NoeAppContext)

Wstrzykiwane aplikacje mają dostęp do `window.NoeAppContext`:

```javascript
{
  user: {
    login: "jan.kowalski",
    email: "jan@example.com",
    name: "Jan Kowalski",
    role: "admin",
    team_ids: [1, 2],
    team_names: ["Sprzedaż", "IT"],
    group_ids: [5],
    group_names: ["Menedżerowie"],
    department_ids: [3],
    department_names: ["Dział IT"]
  }
}
```

### Kontener (id="noe-app")

Wstrzykiwane aplikacje dzielą wspólny kontener `<div id="noe-app">`. Apki powinny **dodawać** (appendChild) do kontenera, nie zastępować jego zawartość:

```javascript
(function() {
  const container = document.getElementById("noe-app");
  const div = document.createElement("div");
  div.innerHTML = `<p>Cześć ${window.NoeAppContext?.user?.name}!</p>`;
  container.appendChild(div);
})();
```

**Ważne:**
- Owijaj kod w IIFE `(function() { ... })();` aby unikać konfliktów zmiennych
- Używaj `appendChild` zamiast `innerHTML` - wiele apek może być na jednej stronie

---

## Akcje aplikacji

Akcje pozwalają na wywołanie [Flow](flows.md) z poziomu frontendu.

```json
{
  "app": {
    "actions": [
      {
        "action": "get_data",
        "flow_code": "my-data-flow",
        "description": "Pobiera dane z API",
        "public": true
      },
      {
        "action": "save_data",
        "flow_code": "save-data-flow",
        "description": "Zapisuje dane"
      }
    ]
  }
}
```

**Wywołanie z frontendu:**

```javascript
const response = await fetch('/noe/apps/APP_ID/action/get_data', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ param1: "value1" })
});
const result = await response.json();
// result = { flow_process_id: "...", status: "pending"|"finished", data: {...} }
```

**Polling statusu (dla długich operacji):**

```javascript
async function waitForResult(flowProcessId) {
  while (true) {
    const response = await fetch(`/noe/apps/APP_ID/action/get_data/status?flow_process_id=${flowProcessId}`);
    const result = await response.json();
    if (result.status === "finished") return result.data;
    if (result.status === "error") throw new Error(result.error);
    await new Promise(r => setTimeout(r, 1000));
  }
}
```

---

## Przykład Svelte 5

```svelte
<script>
  let items = $state([]);
  let newItem = $state("");

  async function fetchData() {
    const response = await fetch('/noe/apps/ID/action/get_data', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ query: "test" })
    });
    const result = await response.json();
    items = result.data || [];
  }

  function addItem() {
    if (newItem.trim()) {
      items = [...items, { name: newItem }];
      newItem = "";
    }
  }
</script>

<div class="p-4">
  <input bind:value={newItem} class="border p-2 rounded" placeholder="Nowy element" />
  <button onclick={addItem} class="bg-blue-500 text-white p-2 rounded ml-2">Dodaj</button>
  <button onclick={fetchData} class="bg-green-500 text-white p-2 rounded ml-2">Pobierz dane</button>

  <ul class="mt-4">
    {#each items as item}
      <li class="p-2 border-b">{item.name}</li>
    {/each}
  </ul>
</div>
```

## Przykład Vue 3

```vue
<script setup>
import { ref } from 'vue';

const count = ref(0);
const items = ref([]);

async function fetchData() {
  const response = await fetch('/noe/apps/ID/action/get_data', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' }
  });
  items.value = (await response.json()).data || [];
}
</script>

<template>
  <div class="p-4">
    <button @click="count++" class="bg-blue-500 text-white p-2 rounded">
      Kliknięto: {{ count }}
    </button>
    <button @click="fetchData" class="bg-green-500 text-white p-2 rounded ml-2">
      Pobierz dane
    </button>
    <ul>
      <li v-for="item in items" :key="item.id">{{ item.name }}</li>
    </ul>
  </div>
</template>
```

## Przykład React 19

```jsx
import { useState } from 'react';
import { createRoot } from 'react-dom/client';

function App() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);

  async function fetchData() {
    const response = await fetch('/noe/apps/ID/action/get_data', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' }
    });
    const result = await response.json();
    setItems(result.data || []);
  }

  return (
    <div className="p-4">
      <button onClick={() => setCount(c => c + 1)} className="bg-blue-500 text-white p-2 rounded">
        Kliknięto: {count}
      </button>
      <button onClick={fetchData} className="bg-green-500 text-white p-2 rounded ml-2">
        Pobierz dane
      </button>
      <ul>
        {items.map((item, i) => <li key={i}>{item.name}</li>)}
      </ul>
    </div>
  );
}

createRoot(document.getElementById("noe-app")).render(<App />);
```

---

## Security

### Nigdy nie zaszywaj api_token w kodzie aplikacji Noe

**KRYTYCZNE:** Kod źródłowy aplikacji Noe (`source_code`) jest widoczny dla użytkowników. Nigdy nie umieszczaj w nim:
- `api_token` do API Noe
- Tokenów autoryzacji do zewnętrznych serwisów
- Kluczy API
- Haseł lub sekretów

```javascript
// ŹLE - token widoczny w kodzie aplikacji!
const response = await fetch('/noe/dbs/tasks/records.json?api_token=SEKRET');

// DOBRZE - użyj publicznej metody bazy danych lub konektora
const response = await fetch('/noe/db/tasks/list', { method: 'POST' });
```

Dla **publicznych aplikacji** (bez logowania) są dwa rozwiązania:

1. **Publiczne metody w bazie danych** (zalecane) - pole `public_methods` w [Noe::Db](dbs.md#rekordy---dostęp-publiczny)
2. **Connector jako proxy** - konektor z metodami `public: true` w [Connectors](connectors.md)

---

## Replace API

Endpoint do częściowej edycji pól tekstowych bez przesyłania całej zawartości.

### Dostępne endpointy

| Metoda | Ścieżka | Dozwolone pola |
|--------|---------|----------------|
| PATCH | `/noe/apps/:id/replace.json` | `source_code`, `fields`, `prompt` |
| PATCH | `/noe/dbs/:id/replace.json` | `fields`, `schema`, `public_methods` |
| PATCH | `/noe/dbs/:db_id/records/:id/replace.json` | `data` |
| PATCH | `/noe/transforms/:id/replace.json` | `transform_code`, `example_object_in` |
| PATCH | `/noe/vector_dbs/:id/replace.json` | `system_prompt` |
| PATCH | `/noe/vector_dbs/:vdb_id/vector_entries/:id/replace.json` | `content` |
| PATCH | `/cms/sites/:id/replace.json` | `content`, `fields` |
| PATCH | `/cms/pages/:id/replace.json` | `content`, `fields` |
| PATCH | `/cms/layouts/:id/replace.json` | `content`, `fields` |
| PATCH | `/cms/paragraphs/:id/replace.json` | `content`, `fields` |
| PATCH | `/cms/articles/:id/replace.json` | `content`, `fields` |

### Parametry

| Parametr | Typ | Wymagane | Opis |
|----------|-----|----------|------|
| `field` | string | tak | Nazwa pola do edycji (musi być na liście dozwolonych) |
| `old_string` | string | tak | Fragment tekstu do zamiany |
| `new_string` | string | nie | Nowy tekst (domyślnie pusty = usunięcie fragmentu) |
| `replace_all` | boolean | nie | `true` = zamień wszystkie wystąpienia, domyślnie `false` |

### Przykład

```
PATCH /noe/apps/123/replace.json?api_token=TOKEN
Content-Type: application/json

{
  "field": "source_code",
  "old_string": "<h1>Stary tytuł</h1>",
  "new_string": "<h1>Nowy tytuł</h1>"
}
```

### Obsługa błędów

| Kod | Błąd | Kiedy |
|-----|------|-------|
| 422 | `field must be one of: ...` | Pole nie na liście dozwolonych |
| 422 | `old_string is required` | Brak `old_string` |
| 422 | `old_string not found in ...` | Fragment nie istnieje w polu |
| 422 | `old_string is not unique, use replace_all: true` | Wiele wystąpień bez `replace_all` |

---

## Przykłady

### Przykład 1: Aplikacja pogodowa

**1. Utwórz [connector](connectors.md) do API pogody:**

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
        "path": "/forecast?latitude={latitude}&longitude={longitude}&current_weather=true"
      }
    ]
  }
}
```

**2. Utwórz [flow](flows.md) do pobierania pogody:**

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

**3. Utwórz aplikację Svelte:**

```json
{
  "app": {
    "name": "Pogoda",
    "url_code": "pogoda",
    "app_engine": "svelte",
    "css_engine": "tailwind_all",
    "public": true,
    "actions": [
      { "action": "get_weather", "flow_code": "get-weather", "public": true }
    ],
    "source_code": "..."
  }
}
```

### Przykład 2: Integracja z zewnętrznym CRM

Pełny przykład: [connector](connectors.md) + [transformacja](transforms.md) + [flow](flows.md) do synchronizacji kontaktów.

### Przykład 3: Aplikacja Todo z bazą danych

Pełny przykład publicznej aplikacji z bazą danych: [dbs.md - Przykład Todo](dbs.md#przykład-aplikacja-todo)

---

## Wskazówki

1. **Publiczne API:**
   - [Baza danych](dbs.md): pole `public_methods` - dostępne przez `/noe/db/:code/:method` (zalecane)
   - [Konektory](connectors.md): `public: true` w `methods` - dostępne przez `/noe/connectors/:code/:method`
   - Akcje aplikacji: `public: true` w `actions` - endpoint bez autoryzacji
2. **Debugowanie flow:** Dodaj `?perform_now=1` do uruchomienia synchronicznego (tylko dev/test)
3. **Polling:** Dla długich operacji używaj `action_status` do sprawdzania postępu
4. **Tailwind CSS:** Używaj `css_engine: tailwind_all` dla pełnego CDN Tailwind
5. **Dane per użytkownik:** Użyj `auto_fields` z `{ip}` + `scope_by` do separacji danych per IP
