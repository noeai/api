[← Spis treści](README.md)

# Bazy wektorowe

**Bazy wektorowe** umożliwiają wyszukiwanie semantyczne i budowanie aplikacji RAG (Retrieval-Augmented Generation)

## Pojęcia

- **Embedding** - numeryczna reprezentacja tekstu (wektor 1536 wymiarów)
- **Wyszukiwanie semantyczne** - znajdowanie podobnych tekstów po znaczeniu, nie słowach kluczowych
- **RAG** - technika łącząca wyszukiwanie z generowaniem odpowiedzi przez LLM
- **Distance** - odległość między wektorami (0 = identyczne, 1 = różne)

## API - Bazy wektorowe

### Lista baz
```
GET /noe/vector_dbs.json
```

### Tworzenie bazy
```
POST /noe/vector_dbs.json
{
  "vector_db": {
    "name": "KB Produkty",
    "code": "kb-produkty",
    "connector_id": 6,
    "knowledge_base_id": 123,
    "context_limit": 10
  }
}
```

### Szczegóły bazy
```
GET /noe/vector_dbs/{id}.json
```

## API - Wpisy (entries)

### Lista wpisów
```
GET /noe/vector_dbs/{id}/vector_entries.json
```

### Dodawanie wpisu
```
POST /noe/vector_dbs/{id}/vector_entries.json
{
  "vector_entry": {
    "content": "Treść artykułu lub dokumentu do zaindeksowania",
    "kb_entry_id": 456,
    "metadata": {
      "source": "kb_article",
      "article_id": 123,
      "category": "faktury"
    }
  }
}
```
System automatycznie wygeneruje embedding dla treści.

### Aktualizacja wpisu
```
PATCH /noe/vector_dbs/{id}/vector_entries/{entry_id}.json
{
  "vector_entry": {
    "content": "Zaktualizowana treść"
  }
}
```
Embedding zostanie automatycznie przeliczony.

## API - Wyszukiwanie semantyczne

### Wyszukiwanie podobnych
```
GET /noe/vector_dbs/{id}/search.json?q=jak wystawić fakturę&limit=5
```

Odpowiedź:
```json
[
  {
    "content": "Aby wystawić fakturę...",
    "neighbor_distance": 0.23,
    "metadata": {"category": "faktury"}
  }
]
```

## API - Chat RAG

### Pytanie z kontekstem
```
POST /noe/vector_dbs/{id}/chat.json
{
  "q": "Jak poprawić błąd na fakturze?",
  "context_limit": 5
}
```

Odpowiedź:
```json
{
  "query": "Jak poprawić błąd na fakturze?",
  "response": "Aby poprawić błąd, wystawić fakturę korygującą...",
  "context": [
    {"content": "...", "distance": 0.22, "metadata": {}}
  ]
}
```

## Użycie w aplikacjach Noe

### Wyszukiwanie w Svelte
```javascript
async function searchKB(query) {
  const response = await fetch(
    `/noe/vector_dbs/kb-produkty/search.json?q=${encodeURIComponent(query)}&limit=5`
  );
  return await response.json();
}
```

### Chat RAG w Svelte
```javascript
async function askQuestion(question) {
  const response = await fetch('/noe/vector_dbs/kb-produkty/chat.json', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ q: question, context_limit: 5 })
  });
  const data = await response.json();
  return data.response;
}
```

## Dobre praktyki

### Rozmiar wpisów
- Zalecane: 100-300 słów na wpis
- Za długie teksty tracą precyzję wyszukiwania
- Dziel długie dokumenty na mniejsze fragmenty (chunki)

### Metadata
Używaj metadata do:
- Filtrowania wyników (np. po kategorii)
- Śledzenia źródła (article_id, page_url)
- Dodatkowego kontekstu dla RAG

### Indeksowanie
```javascript
for (const article of articles) {
  await fetch(`/noe/vector_dbs/${vectorDbId}/vector_entries.json`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      vector_entry: {
        content: article.content,
        metadata: {
          article_id: article.id,
          title: article.title,
          category: article.category
        }
      }
    })
  });
}
```

## Connector AI

Baza wektorowa wymaga connectora AI (OpenAI, Gemini) do generowania embeddingów.

Konfiguracja connectora:
- `chat_model` - model do RAG chat (domyślnie gpt-5.2)
- `embedding_model` - model do embeddingów (domyślnie text-embedding-3-small)
