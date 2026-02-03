---
name: skelet-next-development
description: >-
  Vývoj nových projektů ze Skelet Next. Aktivuje se při vytváření nových modulů,
  entit, routin, oprávnění a API dokumentace. Skelet Next je základní projekt pro API,
  ze kterého vycházejí další projekty šité přímo pro zákazníky.
license: MIT
metadata:
  author: internal
  version: '1.0.0'
---

# Skelet Next Development

Komplexní pravidla a best practices pro vývoj projektů ze Skelet Next. Obsahuje pravidla pro architekturu, kódování, testování a dokumentaci.

## When to Apply

Aktivuj tento skill když:

- Začínáš nový projekt ze Skelet Next
- Přizpůsobuješ základní moduly (eshop, obsah, admin) zákazníkovi
- Vytváříš nový modul/kontroler/service pro konkrétního klienta
- Pracuješ na databázových migracích
- Tvořís oprávnění a policies
- Píšeš API dokumentaci
- Pracuješ na testování

## Co je Skelet Next?

**Skelet Next** je základní projekt pro API, ze kterého vycházejí další projekty šité přímo pro zákazníky. Obsahuje:

- ✅ **Eshop modul** - produkty, kategorie, objednávky, košík
- ✅ **Obsahová část** - články, galerie, SEO, tagy
- ✅ **Administrace** - kompletní admin API s autorizací
- ✅ **Autentizace** - Laravel Sanctum, role a oprávnění (Spatie)
- ✅ **Vícejazyčnost** - astrotomic/laravel-translatable
- ✅ **Vyhledávání** - Laravel Scout + Elasticsearch

## Tech Stack

- **PHP 8.4+** - strict types, nullsafe, readonly, enums, match
- **Laravel 12** - upgraded z v10 (zachována původní struktura)
- **MariaDB 10.11** - JSON validation, foreign keys
- **Docker** - kontejnerizace

## Pravidla dle priority

| Priorita | Kategorie | Dopad | Prefix |
|----------|-----------|-------|--------|
| 1 | Architektura | KRITICKÝ | `architecture-` |
| 2 | Code Style | VYSOKÝ | `code-style-` |
| 3 | Validace | VYSOKÝ | `validation-` |
| 4 | Oprávnění | VYSOKÝ | `permissions-` |
| 5 | Migrace & Databáze | STŘEDNÍ | `migration-` |
| 6 | Modely | STŘEDNÍ | `model-` |
| 7 | Services | STŘEDNÍ | `service-` |
| 8 | Testování | STŘEDNÍ | `testing-` |
| 9 | Dokumentace | STŘEDNÍ | `docs-` |
| 10 | Nástroje | NÍZKÝ | `tools-` |

## Quick Reference

### 1. Architektura (KRITICKÝ)

**Poznámka:** Architektura je pokryta přímo v Controller, Service, Resource a Policy pravidlech níže.

### 2. Code Style (VYSOKÝ)

- `code-style` - PHP 8.4+ best practices (strict types, readonly, final, match, enums, PHPDoc)

### 3. Validace (VYSOKÝ)

- `validation-structure` - Validation třídy s createOrReplace() a update()
- `validation-foreign-keys` - Validace cizích klíčů

**Poznámka:** Další validační pravidla jsou popsána v `rules/validation-*.md`.

### 4. Oprávnění (VYSOKÝ)

- `permissions-enum-naming` - PascalCase názvy + camelCase hodnoty formát
- `permissions-api-mappings` - Lidsky čitelné české názvy
- `permissions-descriptions` - Detailní popisy oprávnění (bez .own)
- `permissions-groups` - Skupiny oprávnění pro organizaci v UI
- `permissions-artisan-command` - Spuštění app:create-permissions
- `policy-structure` - Policy struktura s autorizací a Forbidden exception

### 5. Testování (STŘEDNÍ)

**Poznámka:** Testovací pravidla jsou popsána v Laravel Boost guidelines (PHPUnit, factories, feature tests).

### 6. Modely (VYSOKÝ)

- `elasticsearch-implementation` - Scout + Explorer pro full-text vyhledávání
- `model-structure` - Struktura modelů
- `model-translatable` - Vícejazyčné modely s astrotomic/laravel-translatable
- `model-casts` - Castování atributů
- `model-relationships` - Eloquent vztahy
- `model-activity-log` - Activity log pro sledování změn

### 7. Services (VYSOKÝ)

- `service-structure` - Struktura service vrstvy
- `service-upsert` - Upsert operace
- `service-builder` - Query builder metody

### 8. Migrace & Databáze (VYSOKÝ)

- `migration-naming` - Naming conventions pro migrace
- `migration-required-columns` - Povinné sloupce (NOT NULL)
- `migration-optional-columns` - Nepovinné sloupce (NULL)
- `migration-data-types` - Datové typy a validace
- `migration-collation` - Collation pro varchar
- `migration-foreign-keys` - Foreign keys konvence
- `migration-json-validation` - JSON validace v MariaDB
- `migration-reversible` - Reversible migrace
- `migration-translations` - Překladové tabulky
- `migration-pivot-tables` - Pivot tabulky
- `migration-indexes` - Indexy
- `migration-phpdoc-update` - Aktualizace PHPDoc u modelů

### 9. Dokumentace (STŘEDNÍ)

- `api-documentation` - Kompletní API dokumentace (struktura, běžné chyby, MCP registrace)
- `hoppscotch-collections` - Hoppscotch v10 kolekce pro API testování

### 10. Nástroje (NÍZKÝ)

- `tools-docker-commands` - Docker compose příkazy
- `tools-artisan-scaffolding` - Artisan make commands
- `tools-mcp-tools` - Laravel Boost, Context7, API Server, httpProxy

### 11. Controllers, Validations & Resources (VYSOKÝ)

- `controller-structure` - Struktura controlleru s readonly service a middleware
- `controller-rest-methods` - REST metody (index, show, store, update, destroy, restore, actions)
- `validation-structure` - Validation třídy s createOrReplace() a update()
- `resource-structure` - Resource, ListResource, EnumResource s camelCase

### 12. Router (VYSOKÝ)

- `router-basics` - Aliasy, middleware, model binding
- `router-admin-rest` - Admin REST struktura s withTrashed
- `router-public-rest` - Public REST struktura s preview a cache
- `router-constraints` - RouteParamType constraints
- `enum-controller` - EnumController pro výčtové seznamy

## Běžné příkazy

```bash
# Docker
docker-compose up                         # Start services
docker-compose down --remove-orphans      # Stop services
docker exec -it {container-name} bash     # Vstup do kontejneru

# Artisan (přes docker compose)
docker compose run php php artisan migrate
docker compose run php php artisan test --compact
docker compose run php php artisan app:create-permissions

# Scaffolding
docker compose run php php artisan make:controller Admin/Products/ProductController
docker compose run php php artisan make:model Product -mfs
docker compose run php php artisan make:policy ProductPolicy
```

## Core Architecture Overview

```
app/
├── Http/Controllers/
│   ├── Admin/{Modul}/          # Admin API
│   ├── Public/{Modul}/         # Public API
│   └── {Název}Controller       # Společné
├── Models/                     # Eloquent modely
└── Policies/                   # Authorization

domain/                         # Business logika
├── {Modul}/
│   └── {Entity}Service         # Servisní vrstva

frame/                          # Framework extensions
├── Controllers/
├── Permissions/
├── Serializers/
└── Validation/
```

## Detailní pravidla

Všechna detailní pravidla jsou v adresáři `rules/`:

**Vytvořené skills:**

- **Code Style (1)**: `code-style`
- **Oprávnění (5)**: `permissions-enum-naming`, `permissions-api-mappings`, `permissions-descriptions`, `permissions-groups`, `permissions-artisan-command`
- **Controllers/Validations/Resources (4)**: `controller-structure`, `controller-rest-methods`, `validation-structure`, `resource-structure`
- **Router (5)**: `router-basics`, `router-admin-rest`, `router-public-rest`, `router-constraints`, `enum-controller`
- **Policy (1)**: `policy-structure`
- **Elasticsearch (1)**: `elasticsearch-implementation`
- **Modely (5)**: `model-structure`, `model-translatable`, `model-casts`, `model-relationships`, `model-activity-log`
- **Services (3)**: `service-structure`, `service-upsert`, `service-builder`
- **Migrace (12)**: `migration-naming`, `migration-required-columns`, `migration-optional-columns`, `migration-data-types`, `migration-collation`, `migration-foreign-keys`, `migration-json-validation`, `migration-reversible`, `migration-translations`, `migration-pivot-tables`, `migration-indexes`, `migration-phpdoc-update`
- **Validace (2)**: `validation-structure`, `validation-foreign-keys`
- **Dokumentace (2)**: `api-documentation`, `hoppscotch-collections`
- **Nástroje (3)**: `tools-docker-commands`, `tools-artisan-scaffolding`, `tools-mcp-tools`

**Celkem: 44 skills**

---

**📘 Pro podrobnosti viz jednotlivé soubory v `rules/` adresáři.**
