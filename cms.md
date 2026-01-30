[← Spis treści](README.md)

# CMS - Dokumentacja API

System zarządzania treścią stron internetowych.

**Autoryzacja:** Wszystkie requesty wymagają `api_token` (parametr GET)

**Ważne:** Zamiast `*_id` można używać `*_code` (np. `site_code` zamiast `site_id`)

**Zobacz też:** [Wytyczne tworzenia stron (responsywność, menu mobilne)](cms_guidelines.md)

---

## API Endpoints

### Layouts (Szablony)

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/cms/layouts.json` | Lista szablonów |
| GET | `/cms/layouts/:code.json` | Szczegóły (po code lub id) |
| POST | `/cms/layouts.json` | Utworzenie |
| PATCH | `/cms/layouts/:code.json` | Aktualizacja |
| DELETE | `/cms/layouts/:code.json` | Usunięcie |

### Sites (Strony WWW)

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/cms/sites.json` | Lista site'ów |
| GET | `/cms/sites/:code.json` | Szczegóły |
| POST | `/cms/sites.json` | Utworzenie |
| PATCH | `/cms/sites/:code.json` | Aktualizacja |

### Pages (Strony)

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/cms/pages.json` | Lista stron |
| GET | `/cms/pages/:code.json` | Szczegóły |
| POST | `/cms/pages.json` | Utworzenie |
| PATCH | `/cms/pages/:code.json` | Aktualizacja |
| GET | `/cms/pages/:code/env.json` | Dane strony (env) |

### Paragraphs (Paragrafy)

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/cms/paragraphs.json` | Lista paragrafów |
| GET | `/cms/paragraphs/:code.json` | Szczegóły |
| POST | `/cms/paragraphs.json` | Utworzenie |
| PATCH | `/cms/paragraphs/:code.json` | Aktualizacja |

### Assets (Pliki)

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/cms/assets.json` | Lista assetów |
| GET | `/cms/assets/:id.json` | Szczegóły |
| POST | `/cms/assets.json` | Utworzenie (obsługuje ZIP) |
| PATCH | `/cms/assets/:id.json` | Aktualizacja |
| DELETE | `/cms/assets/:id.json` | Usunięcie |

### Articles (Artykuły)

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/cms/articles.json` | Lista artykułów |
| GET | `/cms/articles/:id.json` | Szczegóły |
| POST | `/cms/articles.json` | Utworzenie |
| PATCH | `/cms/articles/:id.json` | Aktualizacja |
| DELETE | `/cms/articles/:id.json` | Usunięcie |

### Preview (Podgląd)

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/w/:site_code` | Podgląd strony głównej |
| GET | `/w/:site_code/:path` | Podgląd strony |

---

## Pola modeli

### Layout

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `code` | string | tak | Unikalny kod (zalecany format: `{site_code}-layout`) |
| `name` | string | tak | Nazwa |
| `kind` | string | tak | `page` |
| `content` | string | tak | Szablon HTML z `{{ content }}` |
| `example_content` | string | nie | Przykładowa treść do podglądu layoutu |

### Site

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `code` | string | tak | Unikalny kod |
| `name` | string | tak | Nazwa |
| `kind` | string | tak | `www` lub `kb` |
| `layout_code` | string | nie | Code layoutu |

### Page

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `code` | string | nie | Unikalny kod (zalecany format: `{site_code}-{path}`) |
| `name` | string | tak | Nazwa |
| `kind` | string | tak | `text` |
| `path` | string | tak | URL path (np. "" dla głównej, "about") |
| `site_code` | string | tak | Code site'a |
| `layout_code` | string | nie | Code layoutu |
| `content` | string | nie | Treść strony (Liquid) |
| `paragraph_codes` | array | nie | Lista kodów paragrafów |
| `priority` | decimal | nie | Priorytet sortowania |
| `in_menu` | boolean | nie | Czy strona widoczna w menu (domyślnie: true) |
| `menu_code` | string | nie | Kod grupy menu (np. "top_menu", "footer_menu") |
| `redirect_to` | string | nie | URL przekierowania |

### Paragraph

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `code` | string | tak | Unikalny kod (zalecany format: `{site_code}-{nazwa}`) |
| `name` | string | tak | Nazwa |
| `kind` | string | tak | `text` |
| `content` | string | nie | Treść |
| `site_code` | string | nie | Code site'a |
| `page_code` | string | nie | Code strony (podłącza paragraf do strony) |
| `box_code` | string | nie | Kod boxa (do grupowania paragrafów, np. "sidebar", "footer") |
| `priority` | decimal | nie | Priorytet sortowania |

### Asset

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `name` | string | tak | Nazwa pliku (unikalna w ramach konta) |
| `kind` | string | nie | `image`, `css`, `js`, `zip` (auto-detect z rozszerzenia) |
| `file` | file | nie | Plik do wgrania |
| `folder_id` | integer | nie | ID folderu |
| `site_id` | integer | nie | ID site'a |
| `layout_id` | integer | nie | ID layoutu |

**Import ZIP:** Wgranie pliku ZIP z `kind: "zip"` automatycznie rozpakuje archiwum zachowując strukturę folderów.

**URL pliku:** Po wgraniu asset dostępny jest pod `s3_url` (bezpośredni link S3).

### Article

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `title` | string | tak | Tytuł artykułu |
| `path` | string | auto | Ścieżka URL (auto-generowana z tytułu jeśli pusta) |
| `category_code` | string | nie | Kod kategorii (do grupowania artykułów) |
| `author` | string | nie | Autor |
| `abstract` | text | nie | Streszczenie/lead |
| `content` | text | nie | Pełna treść |
| `image_url` | string | nie | URL obrazka głównego |
| `published_at` | datetime | tak | Data publikacji (domyślnie: teraz) |
| `publish_to` | datetime | nie | Data zakończenia publikacji |
| `accepted` | boolean | nie | Zaakceptowany (domyślnie: true) |
| `published` | boolean | nie | Opublikowany (domyślnie: true) |
| `site_id` | integer | nie | ID site'a (opcjonalne powiązanie) |
| `html_title` | string | nie | Tytuł HTML (meta title) |
| `html_description` | text | nie | Opis HTML (meta description) |
| `html_keywords` | string | nie | Słowa kluczowe (meta keywords) |

**Logika publikacji:**
- Artykuł jest widoczny gdy: `published = true` AND `published_at <= teraz` AND (`publish_to` jest null LUB `publish_to >= teraz`)
- Gdy `accepted = false`, automatycznie ustawiane jest `published = false`

**Konwencja nazewnictwa:** Aby uniknąć konfliktów między site'ami, używaj prefiksu `{site_code}-` w kodach:
- Layout: `strona1-layout`
- Page: `strona1-home`, `strona1-about`
- Paragraph: `strona1-intro`, `strona1-footer`

---

## Liquid Templates

Strony używają szablonów Liquid. Dostępne zmienne:

| Zmienna | Opis |
|---------|------|
| `{{ name }}` | Nazwa strony |
| `{{ path }}` | Ścieżka strony z `/` (np. `/`, `/about`) |
| `{{ url }}` | Pełny URL strony |
| `{{ content }}` | Treść (w layoucie - miejsce na content strony) |
| `{{ paragraphs }}` | Lista paragrafów przypisanych do strony |
| `{{ pages }}` | Lista wszystkich stron w site (do budowy menu) |
| `{{ page }}` | Obiekt aktualnie wyświetlanej strony |
| `{{ site }}` | Obiekt site'a (`site.name`, `site.code`, `site.description`, `site.url`) |
| `{{ layout }}` | Obiekt layoutu (`layout.name`, `layout.code`, `layout.description`) |
| `{{ year }}` | Aktualny rok |
| `{{ powered_by }}` | "Intum" |
| `{{ preview_mode }}` | `true` jeśli strona jest w trybie podglądu |

### Właściwości obiektów

**Page** (`page`, elementy `pages`):
- `id`, `name`, `title`, `path`, `url`, `site_id`
- `html_title`, `html_description`, `html_keywords`, `html_script`

**Site**: `name`, `code`, `description`, `url`

**Layout**: `name`, `code`, `description`, `content`

### Własne pola (fields)

Każdy obiekt (Page, Site, Layout) może mieć własne pola w `fields`:

```liquid
{{ page.owner }}        → page.fields["owner"]
{{ page.html.title }}   → page.fields["html"]["title"] (zagnieżdżone)
{{ owner }}             → page.fields["owner"] (skrót - tylko dla page)
{{ site.company }}      → site.fields["company"]
{{ layout.version }}    → layout.fields["version"]
```

**Ustawianie fields przez API:**

```
PATCH /cms/pages/strona1-home.json?api_token=TOKEN
{
  "page": {
    "fields": {
      "owner": "Jan Kowalski",
      "html": { "title": "Strona główna" }
    }
  }
}
```

### Tablice w fields (zalecane dla list i cenników)

```json
{
  "page": {
    "fields": {
      "plans": [
        { "name": "Micro", "price": "0", "users": "1", "popular": false },
        { "name": "Start", "price": "49", "users": "2", "popular": true },
        { "name": "Pro", "price": "149", "users": "5", "popular": false }
      ],
      "faq": [
        { "question": "Czy mogę zmienić plan?", "answer": "Tak, w dowolnym momencie." }
      ]
    }
  }
}
```

**Szablon Liquid:**

```liquid
<div class="plans">
{% for plan in plans %}
  <div class="plan {% if plan.popular %}popular{% endif %}">
    <h3>{{ plan.name }}</h3>
    <p>{{ plan.price }} zł/mies.</p>
  </div>
{% endfor %}
</div>

<h2>FAQ</h2>
{% for item in faq %}
  <div class="faq-item">
    <h4>{{ item.question }}</h4>
    <p>{{ item.answer }}</p>
  </div>
{% endfor %}
```

**Ważne:** Pola z `fields` są dostępne bezpośrednio w Liquid (np. `plans`, `faq`), **nie** przez `fields.plans`.

### Iteracja po paragrafach

```liquid
{% for p in paragraphs %}
  <div>
    <h3>{{ p.name }}</h3>
    <p>{{ p.content }}</p>
  </div>
{% endfor %}
```

### Menu nawigacyjne z podświetleniem

```liquid
<nav>
  <ul class="flex gap-4">
    {% for p in pages %}
      {% if p.path == page.path %}
        <li><a href="{{ p.path }}" class="font-bold text-blue-600">{{ p.name }}</a></li>
      {% else %}
        <li><a href="{{ p.path }}" class="text-gray-600 hover:text-blue-600">{{ p.name }}</a></li>
      {% endif %}
    {% endfor %}
  </ul>
</nav>
```

### Menu z grupami (menu_code)

```liquid
{% assign top_menu = pages | where: "menu_code", "top_menu" %}
<nav>
  {% for p in top_menu %}
    <a href="{{ p.path }}">{{ p.name }}</a>
  {% endfor %}
</nav>
```

**Uwaga:** Zmienna `pages` zawiera tylko strony z `in_menu = true`.

---

## Tag CMS box - renderowanie paragrafów

Tag `<cms type="box" id="X">` wstawia wszystkie paragrafy z danego site'a, które mają `box_code = X`. Paragrafy sortowane po `priority` (malejąco).

```html
<cms type="box" id="sidebar"></cms>
<cms type="box" id="banner" max="1"></cms>
```

| Atrybut | Opis |
|---------|------|
| `type` | `box` |
| `id` | Wartość `box_code` paragrafów do wstawienia |
| `max` | Opcjonalnie: maksymalna liczba paragrafów |

**Przykład layoutu:**
```html
<header>
  <cms type="box" id="banner" max="1"></cms>
</header>
<main>{{ content }}</main>
<aside>
  <cms type="box" id="sidebar"></cms>
</aside>
<footer>
  <cms type="box" id="footer"></cms>
</footer>
```

---

## Tag CMS article - lista i widok artykułów

Tag `<cms type="article">` obsługuje wyświetlanie listy artykułów i pojedynczego artykułu. Automatycznie rozpoznaje tryb na podstawie ścieżki URL:
- `/blog` → wyświetla listę artykułów (sekcja `<list>`)
- `/blog/moj-artykul` → wyświetla pojedynczy artykuł (sekcja `<show>`)

```html
<cms type="article" category_code="news" per_page="10">
  <list>
    <div class="news-item">
      <img src="{{ image_url }}" alt="{{ title }}">
      <h3><a href="{{ path }}">{{ title }}</a></h3>
      <span class="author">{{ author }}</span>
      <time>{{ published_at_date }}</time>
      <p>{{ abstract }}</p>
    </div>
  </list>
  <show>
    <article>
      <h1>{{ title }}</h1>
      <span class="author">{{ author }}</span>
      <time>{{ published_at }}</time>
      <div class="content">{{ content }}</div>
      <a href="{{ back_url }}">← Powrót do listy</a>
    </article>
  </show>
</cms>
```

| Atrybut | Opis |
|---------|------|
| `type` | `article` |
| `category_code` | Opcjonalnie: filtruje artykuły po kategorii |
| `per_page` | Opcjonalnie: artykułów na stronie (domyślnie: 10, max: 25) |
| `site` | Domyślnie aktualny site. `site="any"` = wszystkie artykuły |

### Dostępne zmienne w szablonie

| Zmienna | Lista | Show | Opis |
|---------|-------|------|------|
| `{{ id }}` | ✓ | ✓ | ID artykułu |
| `{{ title }}` | ✓ | ✓ | Tytuł |
| `{{ path }}` | ✓ | - | Ścieżka URL (link do artykułu) |
| `{{ author }}` | ✓ | ✓ | Autor |
| `{{ category_code }}` | ✓ | ✓ | Kod kategorii |
| `{{ abstract }}` | ✓ | ✓ | Streszczenie |
| `{{ content }}` | ✓ | ✓ | Pełna treść |
| `{{ image_url }}` | ✓ | ✓ | URL obrazka |
| `{{ published_at }}` | ✓ | ✓ | Data publikacji sformatowana |
| `{{ published_at_date }}` | ✓ | ✓ | Tylko data |
| `{{ back_url }}` | - | ✓ | URL powrotu do listy |

### Paginacja

Gdy artykułów jest więcej niż `per_page`, automatycznie wyświetlana jest paginacja (`?page=N`).

**Klasy CSS paginacji:** `cms-pagination`, `cms-pagination-prev`, `cms-pagination-next`, `cms-pagination-page`, `cms-pagination-current`, `cms-pagination-ellipsis`

### SEO - meta tagi artykułu

Gdy wyświetlany jest widok szczegółowy, zmienne `html_title`, `html_description`, `html_keywords` są automatycznie nadpisywane wartościami z artykułu.

---

## Wielojęzyczne pola (opcjonalne)

Paragrafy obsługują wielojęzyczne pola poprzez zagnieżdżoną strukturę locale.

**Ustawianie locale:**
- Na poziomie **Site** - pole `locale`
- Na poziomie **Page** - pole `locale` (nadpisuje site)
- Parametrem URL - `?locale=en` (nadpisuje wszystko)

**Struktura fields z locale:**

```json
{
  "en": { "position": "CTO", "experience": "Expert in distributed systems." },
  "pl": { "position": "Dyrektor Techniczny", "experience": "Ekspert w systemach rozproszonych." },
  "default_field": "wartość bez locale"
}
```

**Działanie:**
- `I18n.locale = :pl` → `{{ position }}` = "Dyrektor Techniczny"
- `I18n.locale = :en` → `{{ position }}` = "CTO"

---

## Kompletny przykład

### 1. Utwórz Layout

```
POST /cms/layouts.json?api_token=TOKEN
{
  "layout": {
    "code": "strona1-layout",
    "name": "Prosty szablon",
    "kind": "page",
    "content": "<!DOCTYPE html>\n<html>\n<head>\n  <title>{{ name }} - {{ site.name }}</title>\n</head>\n<body>\n  <header><h1>{{ site.name }}</h1></header>\n  <nav>\n    <ul>\n      {% for p in pages %}\n        <li><a href=\"{{ p.path }}\" {% if p.path == page.path %}class=\"active\"{% endif %}>{{ p.name }}</a></li>\n      {% endfor %}\n    </ul>\n  </nav>\n  {{ content }}\n</body>\n</html>"
  }
}
```

### 2. Utwórz Site

```
POST /cms/sites.json?api_token=TOKEN
{
  "site": {
    "code": "strona1",
    "name": "Moja Strona",
    "kind": "www",
    "layout_code": "strona1-layout"
  }
}
```

### 3. Utwórz Stronę

```
POST /cms/pages.json?api_token=TOKEN
{
  "page": {
    "code": "strona1-home",
    "name": "Strona Główna",
    "kind": "text",
    "path": "",
    "site_code": "strona1",
    "layout_code": "strona1-layout",
    "content": "<h2>{{ name }}</h2>\n<p>Witamy na naszej stronie!</p>\n{% for p in paragraphs %}\n  <div class=\"paragraph\">\n    <strong>{{ p.name }}</strong>\n    <p>{{ p.content }}</p>\n  </div>\n{% endfor %}"
  }
}
```

### 4. Utwórz Paragrafy

```
POST /cms/paragraphs.json?api_token=TOKEN
{
  "paragraph": {
    "code": "strona1-intro",
    "name": "Wprowadzenie",
    "kind": "text",
    "content": "Witaj na mojej stronie!",
    "site_code": "strona1",
    "page_code": "strona1-home",
    "priority": 10
  }
}
```

### 5. Podgląd

```
GET /w/strona1
```

---

## Klonowanie Site

### Klonowanie podstawowe (tylko site)

```
POST /cms/sites.json?api_token=TOKEN
{
  "from_site_code": "strona1",
  "site": { "name": "Strona testowa", "code": "strona1-test" }
}
```

### Klonowanie głębokie (deep clone)

```
POST /cms/sites.json?api_token=TOKEN
{
  "from_site_code": "strona1",
  "deep_clone": "1",
  "site": { "name": "Strona testowa", "code": "strona1-test" }
}
```

Kopiuje site wraz z layoutami, stronami i paragrafami.

| Parametr | Typ | Opis |
|----------|-----|------|
| `from_site_code` | string | Code źródłowego site'a |
| `deep_clone` | string | `"1"` aby skopiować layouty, strony i paragrafy |

---

## Wskazówki

1. **Placeholder w layoucie:** Layout musi zawierać `{{ content }}`
2. **Sortowanie paragrafów:** Używaj `priority` (wyższy = pierwszy)
3. **Code vs ID:** Używaj `*_code` dla czytelności i przenośności
4. **Preview bez auth:** Endpoint `/w/` nie wymaga autoryzacji
5. **Liquid syntax:** Strony używają Liquid do dynamicznej treści
6. **Klonowanie do testów:** Użyj `deep_clone` aby stworzyć kopię roboczą przed zmianami
