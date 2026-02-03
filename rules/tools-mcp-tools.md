---
title: MCP nástroje
impact: LOW
impactDescription: MCP nástroje pro debugging a vývoj
tags: mcp, laravel-boost, context7, debugging, testing
---

## MCP nástroje

**Impact: LOW**

Projekt používá MCP (Model Context Protocol) nástroje pro debugging a vývoj.

## Dostupné MCP servery

### 1. Laravel Boost

**Server pro Laravel aplikace** - debugging, testing, dokumentace

**Hlavní funkce:**
- Čtení konfigurace (`get-config`, `list-available-config-keys`)
- Databázové dotazy (`database-query`, `database-schema`)
- Tinker (`tinker` - spouštění PHP kódu)
- Logy (`read-log-entries`, `browser-logs`, `last-error`)
- Route listing (`list-routes`)
- Artisan commands (`list-artisan-commands`)
- Application info (`application-info`)
- Dokumentace (`search-docs`)

**Příklady použití:**

```typescript
// Spuštění PHP kódu
await tinker({
  code: "App\\Models\\Product::count()",
  timeout: 180
});

// Dotaz do DB
await database_query({
  query: "SELECT * FROM products WHERE published = 1 LIMIT 10"
});

// Hledání v dokumentaci
await search_docs({
  queries: ["scout indexing", "elasticsearch"]
});

// Aplikační info
await application_info({});
```

### 2. Context7

**Server pro aktuální dokumentaci** knihoven a frameworků

**Hlavní funkce:**
- `resolve-library-id` - Najde správný library ID
- `query-docs` - Hledání v dokumentaci

**Podporované knihovny:**
- Laravel Framework
- Laravel Scout
- Spatie Laravel Permission
- Laravel Sanctum
- atd.

**Příklady použití:**

```typescript
// Najít library ID
await resolve_library_id({
  query: "How to use scout with elasticsearch",
  libraryName: "laravel/scout"
});

// Hledat v dokumentaci
await query_docs({
  libraryId: "/laravel/scout",
  query: "How to create custom search index"
});
```

### 3. API Server (custom)

**Server pro vlastní API dokumentaci**

**Hlavní funkce:**
- `search-documentation-tool` - Hledání v API dokumentaci
- `add-numbers-tool` - Testovací nástroj

**Moduly dokumentace:**
- `admin` - Administrační API
- `public` - Veřejné API
- `shared` - Sdílená dokumentace

**Příklady použití:**

```typescript
// Hledat v admin dokumentaci
await search_documentation_tool({
  module: "admin",
  keyword: "produkty"
});

// Načíst konkrétní dokument podle landmark
await search_documentation_tool({
  module: "admin",
  landmark: "products"
});

// Načíst podle cesty
await search_documentation_tool({
  module: "admin",
  path: "modules/products.md"
});
```

### 4. httpProxy

**Server pro API testování**

**Hlavní funkce:**
- `api-call` - Volání nakonfigurovaných API
- `list-apis` - Seznam dostupných API

**Nakonfigurované API:**
- `jsonplaceholder` - Testovací API
- `uam` - UAM API
- `gs` - GS API

**Příklady použití:**

```typescript
// Seznam API
await list_apis({});

// API call
await api_call({
  apiId: "uam",
  method: "GET",
  endpoint: "/api/admin/products",
  headers: {
    "Authorization": "Bearer token",
    "X-Lang": "cs_CZ"
  }
});
```

## Laravel Boost - detailní použití

### Tinker

Spouštění PHP kódu v kontextu aplikace:

```php
// Počet produktů
App\Models\Product::count()

// Najít produkt
App\Models\Product::find(1)

// Vytvořit záznam
$product = new App\Models\Product();
$product->title = "Test";
$product->save();

// Získat config
config('app.name')

// Test service
app(Domain\Products\ProductService::class)->find(...)
```

**⚠️ Nepoužívej pro:**
- Vytváření modelů (raději factories v testech)
- Komplexní logiku (raději unit testy)
- Produkční změny

### Database Query

**Read-only** SQL dotazy:

```sql
-- Seznam produktů
SELECT * FROM products WHERE published = 1 LIMIT 10;

-- Počet záznamů
SELECT COUNT(*) FROM products;

-- Joiny
SELECT p.*, c.name as category_name 
FROM products p 
LEFT JOIN categories c ON p.category_id = c.id
LIMIT 10;

-- SHOW příkazy
SHOW TABLES;
SHOW COLUMNS FROM products;
```

**⚠️ Pouze SELECT, SHOW, EXPLAIN, DESCRIBE** - žádné INSERT/UPDATE/DELETE!

### Database Schema

Vrací kompletní strukturu databáze:

```typescript
// Celá schema
await database_schema({});

// Filtrované tabulky
await database_schema({
  filter: "products"
});
```

**Vrací:**
- Názvy tabulek
- Sloupce (název, typ, nullable, default)
- Indexy
- Foreign keys
- Constraints

### Logy

```typescript
// Laravel log (posledních 50 záznamů)
await read_log_entries({
  entries: 50
});

// Browser log (JS chyby)
await browser_logs({
  entries: 20
});

// Poslední error
await last_error({});
```

### Routes

```typescript
// Všechny routy
await list_routes({});

// Filtr podle cesty
await list_routes({
  path: "admin/products"
});

// Filtr podle metody
await list_routes({
  method: "POST"
});

// Filtr podle action
await list_routes({
  action: "ProductController"
});
```

## Context7 - Best practices

### Kdy použít

- Hledáš aktuální syntaxi pro Laravel 12
- Potřebuješ příklady použití Scout
- Chceš ověřit správné použití Spatie Permission
- Kontroluješ breaking changes mezi verzemi

### Workflow

1. **Resolve library ID:**
```typescript
await resolve_library_id({
  query: "scout elasticsearch indexing",
  libraryName: "laravel/scout"
});
// Vrátí: /laravel/scout
```

2. **Query dokumentace:**
```typescript
await query_docs({
  libraryId: "/laravel/scout",
  query: "How to create custom index"
});
```

### Limity

- **Max 3 calls** per question
- Pokud nenajdeš po 3 calls, použij best effort
- Používej specifické queries

## API Server - Dokumentace

### Struktura

Dokumentace je v `doc/api/`:
- `admin/modules/` - Admin moduly
- `public/modules/` - Public moduly
- `shared/modules/` - Shared (preview, thumbnails)

### Registrace v MCP

**Po vytvoření dokumentace MUSÍŠ zaregistrovat v `config/mcp-docs.php`:**

```php
'landmarks' => [
    'products' => [
        'path' => 'modules/products.md',
        'title' => 'Products - Produkty',
        'module' => 'admin',
        'keywords' => ['product', 'produkty', 'goods', 'zboží'],
    ],
],
```

Viz [api-documentation.md](api-documentation.md)

### Hledání

```typescript
// Seznam dokumentů v modulu
await search_documentation_tool({
  module: "admin"
});

// Hledat podle klíčových slov
await search_documentation_tool({
  module: "admin",
  keyword: "produkt"
});

// Načíst konkrétní dokument
await search_documentation_tool({
  module: "admin",
  landmark: "products",
  content_only: true
});
```

## Debugging workflow

### 1. Chyba v aplikaci

```typescript
// 1. Zjisti poslední error
await last_error({});

// 2. Prohlédni logy
await read_log_entries({ entries: 20 });

// 3. Zkontroluj browser log (pokud je to frontend chyba)
await browser_logs({ entries: 10 });

// 4. Testuj v tinkeru
await tinker({
  code: "App\\Models\\Product::find(1)"
});
```

### 2. Databázový problém

```typescript
// 1. Schema tabulky
await database_schema({ filter: "products" });

// 2. Testovací dotaz
await database_query({
  query: "SELECT * FROM products LIMIT 5"
});

// 3. Zkontroluj foreign keys
await database_query({
  query: "SHOW CREATE TABLE products"
});
```

### 3. Route problém

```typescript
// 1. Seznam rout
await list_routes({ path: "admin/products" });

// 2. Zkontroluj middleware
await list_routes({ action: "ProductController" });

// 3. Testuj v tinkeru
await tinker({
  code: "route('admin.products.index')"
});
```

## Checklist

- [ ] Používáš Laravel Boost pro debugging
- [ ] Kontroluješ Context7 pro aktuální syntaxi
- [ ] Dokumentaci registruješ v `config/mcp-docs.php`
- [ ] Tinker jen pro quick tests, ne pro produkční změny
- [ ] Database query jen READ-only
- [ ] Max 3 Context7 calls per question

**⚠️ Důležité:**
- **Tinker** - quick tests only
- **Database query** - READ-only!
- **Context7** - max 3 calls
- **API Server** - registrace dokumentace povinná
- **Logy** - browser-logs pro JS chyby

Reference: [API Documentation](api-documentation.md), [Tools Docker Commands](tools-docker-commands.md)
