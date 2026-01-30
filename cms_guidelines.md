[← CMS](cms.md)

# Wytyczne tworzenia stron CMS

## Responsywność (Mobile-First)

**WAŻNE:** Każda tworzona strona MUSI działać poprawnie na urządzeniach mobilnych (telefony, tablety).

### Zasady CSS Grid

Nigdy nie używaj stałej liczby kolumn bez breakpointów responsywnych:

```html
<!-- ŹLE - przewija się w boki na telefonie -->
<div style="display:grid; grid-template-columns:repeat(3,1fr)">

<!-- DOBRZE - responsywne z Tailwind -->
<div class="grid grid-cols-1 md:grid-cols-3 gap-6">
```

### Breakpointy Tailwind

| Prefix | Min-width | Urządzenie |
|--------|-----------|------------|
| (brak) | 0px | Telefon |
| `sm:` | 640px | Duży telefon |
| `md:` | 768px | Tablet |
| `lg:` | 1024px | Laptop |
| `xl:` | 1280px | Desktop |

### Typowe wzorce responsywne

**Grid 2-kolumnowy (tekst + obraz):**
```html
<div class="grid grid-cols-1 md:grid-cols-2 gap-8 items-center">
  <div>Treść tekstowa</div>
  <div><img src="..." class="w-full"></div>
</div>
```

**Grid 3-kolumnowy (karty):**
```html
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
  <div class="card">...</div>
</div>
```

**Grid 4-kolumnowy (galeria):**
```html
<div class="grid grid-cols-2 md:grid-cols-4 gap-4">
  <!-- na telefonie 2 kolumny, na tablecie+ 4 kolumny -->
</div>
```

### Kolejność elementów (order)

```html
<!-- ŹLE - order bez breakpointa -->
<div style="order:2">

<!-- DOBRZE - order tylko na desktop -->
<div class="md:order-2">
```

### Szerokość kontenerów

```html
<!-- ŹLE - może wychodzić poza ekran -->
<div style="width:1200px">

<!-- DOBRZE - max-width z marginesami -->
<div class="max-w-6xl mx-auto px-4">
```

### Obrazy responsywne

```html
<!-- ŹLE - stała szerokość -->
<img style="width:500px" src="...">

<!-- DOBRZE - skaluje się do kontenera -->
<img class="w-full h-auto" src="...">
```

---

## Menu mobilne (hamburger)

**WYMAGANE:** Każda strona z menu nawigacyjnym MUSI mieć menu mobilne.

### Wzorzec menu z hamburgerem

```html
<header class="bg-white shadow">
  <div class="max-w-6xl mx-auto px-4">
    <div class="flex justify-between items-center py-4">
      <!-- Logo -->
      <a href="/" class="text-xl font-bold">{{ site.name }}</a>

      <!-- Menu desktop (ukryte na mobile) -->
      <nav class="hidden md:flex gap-6">
        {% for p in pages %}
          <a href="{{ p.path }}" class="hover:text-blue-600">{{ p.name }}</a>
        {% endfor %}
      </nav>

      <!-- Przycisk hamburger (widoczny tylko na mobile) -->
      <button class="md:hidden" onclick="document.getElementById('mobile-menu').classList.toggle('hidden')">
        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"/>
        </svg>
      </button>
    </div>

    <!-- Menu mobilne (domyślnie ukryte) -->
    <nav id="mobile-menu" class="hidden md:hidden pb-4">
      {% for p in pages %}
        <a href="{{ p.path }}" class="block py-2 hover:text-blue-600">{{ p.name }}</a>
      {% endfor %}
    </nav>
  </div>
</header>
```

### Kluczowe elementy:

1. **`hidden md:flex`** - menu desktop ukryte na mobile, widoczne od tabletu
2. **`md:hidden`** - przycisk hamburger widoczny tylko na mobile
3. **`hidden md:hidden`** - menu mobilne domyślnie ukryte, toggle przez JS
4. **`block py-2`** - linki w menu mobilnym jako bloki (łatwiejsze kliknięcie)

---

## Checklist przed publikacją

- [ ] Strona nie przewija się w boki na telefonie
- [ ] Menu jest dostępne na urządzeniach mobilnych
- [ ] Tekst jest czytelny bez zoomowania (min. 16px)
- [ ] Przyciski/linki są wystarczająco duże do kliknięcia palcem (min. 44x44px)
- [ ] Obrazy skalują się do szerokości ekranu
- [ ] Formularze działają na urządzeniach dotykowych
