# Noe API

Dokumentacja API dla [noe.ai](https://noe.ai) - platforma do tworzenia aplikacji webowych, integracji danych i zarządzania treścią bez pisania kodu backendowego.

## Spis treści

### Noe (No codE)

- [Aplikacje](apps.md) - interaktywne aplikacje webowe (Svelte 5, Vue 3, React 19, JS), security, przykłady
- [Bazy danych](dbs.md) - proste bazy danych z dynamicznym schematem, dostęp publiczny i prywatny
- [Transformacje](transforms.md) - konwersje danych między formatami (JSON, XML, CSV, TXT)
- [Konektory](connectors.md) - połączenia z zewnętrznymi API (REST, SOAP, GraphQL)
- [Flow](flows.md) - przepływy danych łączące transformacje i konektory
- [Bazy wektorowe](vector_dbs.md) - wyszukiwanie semantyczne i RAG (Retrieval-Augmented Generation)

### CMS

- [CMS](cms.md) - system zarządzania treścią: layouts, strony, paragrafy, assety, artykuły, Liquid templates
- [CMS - wytyczne responsywności](cms_guidelines.md) - responsywność, menu mobilne, checklist przed publikacją

## Autoryzacja

Wszystkie requesty API wymagają `api_token`:
- jako parametr GET: `?api_token=TOKEN`
- lub header: `Authorization: Bearer TOKEN`

## Moduły

| Moduł | Opis | Dokumentacja |
|-------|------|--------------|
| **Aplikacje** | Interaktywne aplikacje webowe (Svelte, Vue, React, JS) | [apps.md](apps.md) |
| **Bazy danych** | Proste bazy danych z dynamicznym schematem | [dbs.md](dbs.md) |
| **Transformacje** | Konwersje danych między formatami | [transforms.md](transforms.md) |
| **Konektory** | Połączenia z zewnętrznymi API | [connectors.md](connectors.md) |
| **Flow** | Przepływy danych (pipeline'y) | [flows.md](flows.md) |
| **Bazy wektorowe** | Wyszukiwanie semantyczne i RAG | [vector_dbs.md](vector_dbs.md) |
| **CMS** | System zarządzania treścią stron WWW | [cms.md](cms.md) |
