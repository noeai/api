[← Spis treści](README.md)

# Transformacje (Noe::Transform)

Konwersje danych między formatami.

## Funkcjonalność

- [x] Transformacje JSON ↔ XML (odwracalne)
- [x] Transformacje JSON → TXT
- [x] Transformacje TXT → JSON
- [x] Transformacje CSV → JSON
- [x] Transformacje XSL (XSLT)
- [x] Transformacje Hash (mapping JSON)
- [x] Testowanie transformacji z przykładowymi danymi
- [x] Klonowanie transformacji

## API

| Metoda | Ścieżka | Opis |
|--------|---------|------|
| GET | `/noe/transforms.json` | Lista transformacji |
| GET | `/noe/transforms/:id.json` | Szczegóły |
| POST | `/noe/transforms.json` | Utworzenie |
| PATCH | `/noe/transforms/:id.json` | Aktualizacja |
| DELETE | `/noe/transforms/:id.json` | Usunięcie |
| POST | `/noe/transforms/:id/test.json` | Testowanie |
| PATCH | `/noe/transforms/:id/replace.json` | Częściowa edycja pola ([Replace API](apps.md#replace-api)) |

## Utworzenie transformacji

```
POST /noe/transforms.json?api_token=TOKEN
Content-Type: application/json

{
  "transform": {
    "name": "Order JSON to XML",
    "code": "order-json-to-xml",
    "from": "order_json",
    "to": "order_xml",
    "kind": "json_to_xml",
    "reversible": true,
    "description": "Konwersja zamówień",
    "transform_options": {},
    "example_object_in": "{\"order\": {\"id\": 1}}"
  }
}
```

## Pola transformacji

| Pole | Typ | Wymagane | Opis |
|------|-----|----------|------|
| `name` | string | tak | Nazwa transformacji |
| `code` | string | tak | Unikalny kod (slug) |
| `from` | string | tak | Typ wejściowy (np. "order_json") |
| `to` | string | tak | Typ wyjściowy (np. "order_xml") |
| `kind` | string | tak | Rodzaj transformacji |
| `reversible` | boolean | nie | Czy transformacja jest odwracalna |
| `description` | string | nie | Opis |
| `transform_code` | string | nie | Kod transformacji (dla hash_transform, xsl_transform) |
| `transform_options` | object | nie | Opcje transformacji |
| `example_object_in` | string | nie | Przykładowe dane wejściowe (do walidacji) |

## Rodzaje transformacji (kind)

| Kind | Opis | transform_code |
|------|------|----------------|
| `json_to_xml` | JSON → XML | nie wymagane |
| `xml_to_json` | XML → JSON | nie wymagane |
| `csv_to_json` | CSV → JSON | nie wymagane |
| `txt_to_json` | TXT → JSON | nie wymagane |
| `json_to_txt` | JSON → TXT | nie wymagane |
| `hash_transform` | Mapping JSON z interpolacją | JSON z mapowaniem |
| `xsl_transform` | Transformacja XSLT | Kod XSL |

## Przykład: hash_transform

```json
{
  "transform": {
    "name": "Format API Response",
    "code": "format-api-response",
    "from": "rest_country",
    "to": "formatted_country",
    "kind": "hash_transform",
    "transform_code": "{\"name\": \"{name.common}\", \"capital\": \"{capital[0]}\", \"flag\": \"{flags.svg}\"}",
    "example_object_in": "{\"name\": {\"common\": \"Poland\"}, \"capital\": [\"Warsaw\"], \"flags\": {\"svg\": \"url\"}}"
  }
}
```

**Składnia interpolacji:**
- `{field}` - proste pole
- `{nested.field}` - zagnieżdżone pole
- `{array[0]}` - element tablicy
- `{array[0].field}` - pole w elemencie tablicy

## Przykład: json_to_xml (odwracalna)

```json
{
  "transform": {
    "name": "Invoice JSON to XML",
    "code": "invoice-json-to-xml",
    "from": "invoice_json",
    "to": "invoice_xml",
    "kind": "json_to_xml",
    "reversible": true,
    "example_object_in": "{\"invoice\": {\"number\": \"FV/001\", \"total\": 100}}"
  }
}
```

## Użycie w Flow

Transformacje są automatycznie dostępne w [Flow](flows.md) przez akcję `transform`:

```json
{
  "action": "transform",
  "in": "invoice_json",
  "out": "invoice_xml"
}
```

## Testowanie

Użyj endpointu `/noe/transforms/:id/test.json` przed podłączeniem do flow.
