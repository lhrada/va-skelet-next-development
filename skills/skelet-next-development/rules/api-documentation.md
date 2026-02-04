---
title: API dokumentace generování
impact: HIGH
impactDescription: API dokumentace je kritická pro integraci a používání API
tags: documentation, api, markdown, openapi, mcp
---

## API dokumentace generování

**Impact: HIGH**

Podle tohoto návodu vytváříš dokumentaci pro API moduly ve formátu Markdown a OpenAPI YAML.

**⚠️ Na začátku tvorby** napiš: "Dokumentaci vytvářím podle pravidel v `.github/instructions/doc.instructions.md`"

## Umístění souborů

### Markdown dokumentace

- **Admin**: `doc/api/admin/modules/{module-name}.md`
- **Public**: `doc/api/public/modules/{module-name}.md`
- **Shared**: `doc/api/shared/modules/{module-name}.md`

### OpenAPI specifikace

- `doc/api/openapi/{module-name}.openapi.yml`

**Příklady:**
- `doc/api/admin/modules/products.md` (admin)
- `doc/api/public/modules/articles.md` (public)
- `doc/api/shared/modules/preview.md` (shared - preview, thumbnails)

## Kriticky důležité - Přesnost dat

**NESMÍŠ SI VYMÝŠLET** strukturu dat!

**VŠECHNO MUSÍŠ ZJISTIT Z KÓDU:**

### Kde hledat data:

1. **Validátory** (`*Validation.php`)
   - Přesné názvy parametrů
   - Validační pravidla
   - Povinnost polí

2. **Resources** (`*Resource.php`, `*ListResource.php`)
   - Struktura výstupních dat
   - Názvy polí v responses

3. **GenericSerializer** (`frame/Serializers/GenericSerializer.php`)
   - `published()` - struktura publikování
   - `seo()` - SEO metadata
   - `toDateTime()` - časové značky
   - `getLinkObject()` - link objekty

4. **Common validátor** (`frame/Validation/Common.php`)
   - `address()` - struktura adresy
   - `seo()` - SEO validace
   - `publication()` - publikační údaje

5. **Modely**
   - Vztahy (relationships)
   - Vlastnosti (properties)

### Příklady správných struktur:

**❌ ŠPATNĚ:**
```json
"address": {
  "street": "Ulice 123"
}
```

**✅ SPRÁVNĚ:**
```json
"address": {
  "address1": "Radnická",
  "address2": "",
  "address3": "",
  "houseNumber": "8",
  "orientationNumber": ""
}
```

**❌ ŠPATNĚ:**
```json
"published": {
  "value": true,
  "locale": true
}
```

**✅ SPRÁVNĚ:**
```json
"published": {
  "published": true,
  "locales": {
    "cs_CZ": true,
    "en_US": false
  },
  "from": {
    "iso": "2024-01-01T00:00:00+01:00",
    "unix": 1704063600,
    "timezone": "Europe/Prague"
  },
  "to": null
}
```

## Běžné chyby

### 1. Kopíruj strukturu z existujících modulů

**✅ Vzorové dokumenty:**
- `doc/api/admin/modules/products.md`
- `doc/api/public/modules/articles.md`

**MUSÍŠ kopírovat:**
- Počet sloupců v tabulkách
- Formát příkladů requestů (bash kód blok)
- Strukturu response JSON
- Sekce a jejich pořadí

### 2. Headers formát

**❌ ŠPATNĚ** (bullet list):
```markdown
- Authorization: Bearer {token} - Povinné
- Accept: application/json - Povinné
```

**✅ SPRÁVNĚ** (code block):
````markdown

#### Headers

```
Authorization: Bearer {token}
Accept: application/json
X-Lang: cs_CZ
X-Data-Lang: cs_CZ
X-Channel: 1
```
````

### 3. Query parametry tabulka

**✅ SPRÁVNĚ** (standardní sloupce):
| Parametr | Typ | Povinnost | Popis | Příklad |
|----------|-----|-----------|-------|---------|

**Pro POST/PATCH body:**
| Parametr | Typ | Povinnost | Validace | Popis | Enum endpoint |
|----------|-----|-----------|----------|-------|---------------|

### 4. Řazení a filtrování

**❌ ŠPATNĚ:**
```
sort=-field
filter[field]=value
```

**✅ SPRÁVNĚ:**
```
sort[0][field]=priority&sort[0][order]=desc
filter[field]=value
filter[field][value]=x&filter[field][operator]=LIKE
```

### 5. Příklady requestů

**✅ SPRÁVNĚ** (bash kód blok):
````markdown
```bash
GET /api/admin/products?page[number]=1&page[size]=20

Authorization: Bearer your_token_here
X-Lang: cs_CZ
X-Data-Lang: cs_CZ
X-Channel: 1
```
````

### 6. Response struktura

**VŽDY zahrnout:**
```json
{
  "data": [...],
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 5,
    "per_page": 20,
    "to": 20,
    "total": 95
  },
  "links": {
    "first": "/api/admin/products?page=1",
    "last": "/api/admin/products?page=5",
    "prev": null,
    "next": "/api/admin/products?page=2"
  }
}
```

### 7. Pozice filtry (*Positions)

Pro filtry končící na `*Positions` (např. `tagsPositions`):

**❌ ŠPATNĚ** (číselné):
```
filter[tagsPositions][]=[1,2,3]
```

**✅ SPRÁVNĚ** (alfanumerické):
```
filter[tagsPositions][]=001&filter[tagsPositions][]=0a2000%
```

**Důvod:** `position` pole je `string` pro řazení, ne číslo.

**Aplikuje se na:**
- `categoriesPositions`
- `tagsPositions`
- `locationsPositions`
- atd.

### 8. Specifické parametry

**`sort-random30`** - Náhodné řazení se seedem platným **30 minut**:
```
sort[0][field]=sort-random30&sort[0][order]=asc
```

## Struktura dokumentace

### 1. Hlavička modulu

```markdown

# {Název modulu} API

**Verze:** 1.0.0  
**Poslední aktualizace:** 2026-02-02  
**Odpovědná osoba:** {Jméno}

## Popis

Stručný popis co modul dělá, jaké funkcionality poskytuje.

## Autentizace

- **Admin**: Vyžaduje `auth:sanctum` + role/oprávnění
- **Public**: Veřejně přístupné / Vyžaduje autentizaci

## Base URL

- **Admin**: `/api/admin/{resource}`
- **Public**: `/api/public/{resource}`
```

### 2. Přehled endpointů

```markdown

## Přehled endpointů

| Metoda | Endpoint | Popis | Auth | Middleware |
|--------|----------|-------|------|------------|
| GET | `/` | Seznam záznamů | ✓ | vaTenant |
| GET | `/{id}` | Detail záznamu | ✓ | tenant |
| POST | `/` | Vytvoření záznamu | ✓ | tenant |
| PATCH | `/{id}` | Aktualizace záznamu | ✓ | tenant |
| DELETE | `/{id}` | Smazání záznamu | ✓ | tenant, withTrashed |
| PATCH | `/{id}/restore` | Obnovení záznamu | ✓ | tenant, withTrashed |
| POST | `/actions` | Hromadné akce | ✓ | tenant |
```

**Poznámka:** Všechny endpointy jsou relativní k Base URL.

### 3. Detailní dokumentace endpointu

Pro každý endpoint:

```markdown
---

## GET `/products`

Vrací seznam produktů s možností filtrování, řazení a stránkování.

### Request

**Autentizace:** Povinná (`auth:sanctum`)  
**Oprávnění:** `product.view.any`  
**Middleware:** `vaTenant`

#### Headers

```
Authorization: Bearer {token}
Accept: application/json
X-Lang: cs_CZ
X-Data-Lang: cs_CZ
X-Channel: 1
```

#### Query parametry

| Parametr | Typ | Povinnost | Popis | Příklad |
|----------|-----|-----------|-------|---------|
| `page[number]` | integer | volitelné | Číslo stránky | `1` |
| `page[size]` | integer | volitelné | Počet záznamů | `20` |
| `filter[state]` | string | volitelné | Filtr podle stavu | `"published"` |
| `sort[0][field]` | string | volitelné | Pole pro řazení | `"priority"` |
| `sort[0][order]` | string | volitelné | Směr (asc/desc) | `"desc"` |

#### Příklad requestu

```bash
GET /api/admin/products?page[number]=1&page[size]=20

Authorization: Bearer your_token_here
X-Lang: cs_CZ
X-Data-Lang: cs_CZ
X-Channel: 1
```

### Response

#### Success (200 OK)

```json
{
  "data": [ { "id": 1, "title": "Produkt", ... } ],
  "meta": { "current_page": 1, ... },
  "links": { "first": "...", ... }
}
```

#### Error responses

...

### HTTP Status kódy

- `200 OK` - Úspěšné načtení
- `401 Unauthorized` - Chybí token
- `403 Forbidden` - Nedostatečná oprávnění
- `422 Unprocessable Entity` - Validační chyba

### Poznámky

- Data podle `X-Data-Locale`
- `trash=true` pro smazané záznamy
```

## Registrace v MCP

**POVINNÉ** po vytvoření dokumentace!

Zaregistruj v `config/mcp-docs.php`:

```php
'landmarks' => [
    // ...existing...
    
    'products' => [
        'path' => 'modules/products.md',
        'title' => 'Products - Produkty',
        'module' => 'admin',
        'keywords' => [
            'product', 'produkty', 'produkt', 
            'goods', 'zboží', 'items'
        ],
    ],
],
```

**Parametry:**
- `path` - Relativní od `doc/api/{module}/`
- `title` - Jasný nadpis
- `module` - `admin`, `public`, nebo `shared`
- `keywords` - Minimálně 3-5 klíčových slov (CZ + EN)

## Společné konvence

### Headers

**X-Lang** - Locale aplikace (`cs_CZ`, `en_US`, `de_DE`)
- Ovlivňuje UI (tlačítka, hlášky)

**X-Data-Lang** - Locale dat (`cs_CZ`, `en_US`, `de_DE`)
- Určuje jazyk vrácených dat (názvy, popisy, texty)

**X-Channel** - ID nebo klíč kanálu
- Celé číslo (`1`, `2`) nebo string (`channel_brno`)
- **tenant** middleware: **povinný**
- **vaTenant** middleware: **volitelný**

### Naming

**camelCase** - všechny klíče v JSON jsou `camelCase`, ne `snake_case`

### Query parametry

**Strukturované formáty:**
- Pagination: `page[number]`, `page[size]`, `page[offset]`
- Filtering: `filter[field]=value` nebo `filter[field][value]=x&filter[field][operator]=<`
- Sorting: `sort[0][field]=field&sort[0][order]=desc`

### Operátory filtrování

- `=` - Rovná se (výchozí)
- `<`, `>`, `<=`, `>=` - Porovnání
- `LIKE` - SQL LIKE
- `IN` - Pole hodnot

### Časové značky

**ISO 8601:** `2024-01-15T14:30:00Z`

### Nullable hodnoty

`null` hodnoty jako `null`, ne prázdný string nebo `undefined`

### Pagination

Laravel pagination: `data`, `meta`, `links`

### Error handling

Konzistentní formát: `message` a `errors` objekt

## Changelog

Na konci dokumentace **VŽDY** přidej changelog:

```markdown

## Změny a historie

| Verze | Datum | Popis změny |
|-------|-------|------------|
| 1.0.1 | 2026-02-02 | Přidány filtry `categories` a `tags` |
| 1.0.0 | 2026-01-15 | Inicální verze dokumentace |
```

## Verze a aktualizace

**Každá dokumentace MUSÍ mít verzi a datum poslední aktualizace!**

### Základní pravidla

1. **Hlavička dokumentace MUSÍ obsahovat:**
   - `**Verze:** X.Y.Z` (semver: Major.Minor.Patch)
   - `**Poslední aktualizace:** YYYY-MM-DD` (ISO formát)
   - `**Odpovědná osoba:** Jméno` (kdo spravuje dokumentaci)

2. **Při každé změně:**
   - Aktualizuj verzi podle principů semveru
   - Aktualizuj datum poslední aktualizace
   - Přidej záznam do changelog tabulky

3. **Changelog:**
   - Tabulka s sloupci: Verze, Datum, Popis změny
   - Umístění: **na konci dokumentace**
   - Seřazeno od nejnovější verze nahoře

### Principy verzování (SemVer)

#### **Major (X.0.0)** - Zásadní, nekompatibilní změny

Zvyšuj MAJOR, když:
- Smazeš endpoint nebo jeho parametr
- Změníš strukturu response (odebrání polí)
- Změníš chování endpointu (jiný formát dat)
- Změníš HTTP metodu endpointu

**Příklady:**
- `1.0.0` → `2.0.0` - Smazání endpointu `/api/products/{id}/details`
- `1.0.0` → `2.0.0` - Změna response struktury `products` na `items`
- `1.0.0` → `2.0.0` - Změna oprávnění formátu z `product.view` na `products.list`

#### **Minor (0.X.0)** - Nové funkcionality, zpětně kompatibilní

Zvyšuj MINOR, když:
- Přidáš nový endpoint
- Přidáš nový parametr (query, body)
- Přidáš nové pole do response
- Přidáš nový filtr nebo řazení
- Přidáš novou metodu (PUT, PATCH, DELETE)

**Příklady:**
- `1.0.0` → `1.1.0` - Přidání endpointu `POST /api/products/batch`
- `1.0.0` → `1.1.0` - Přidání parametru `filter[category]` do GET `/products`
- `1.0.0` → `1.1.0` - Přidání pole `rating` do response produktu
- `1.0.0` → `1.1.0` - Přidání nového query parametru `sort[0][field]=popularity`

#### **Patch (0.0.X)** - Opravy a drobné úpravy

Zvyšuj PATCH, když:
- Opravíš chybu v dokumentaci (překlep, špatný příklad)
- Doplníš chybějící informaci (HTTP status, poznámka)
- Aktualizuješ příklady requestů/responses
- Upřesníš popis parametru
- Doplníš chybějící parametr v tabulce
- Opravíš formát kódu
- Aktualizuješ changelog

**Příklady:**
- `1.0.0` → `1.0.1` - Oprava chyby v příkladu: `page[number]` místo `page`
- `1.0.0` → `1.0.1` - Doplnění chybějícího HTTP status kódu `201 Created`
- `1.0.0` → `1.0.1` - Oprava překladu popisu parametru
- `1.0.0` → `1.0.1` - Doplnění chybějícího headeru `X-Channel`
- `1.0.0` → `1.0.1` - Aktualizace příkladu response na reálná data

### Praktické příklady verzování

```
Počáteční verze:
1.0.0 - 2026-01-15 - Iniciální verze dokumentace

Přidání nového endpointu:
1.1.0 - 2026-01-20 - Přidání endpointu POST /products/bulk-update

Přidání nového filtru:
1.2.0 - 2026-01-25 - Přidání filtru filter[inStock]

Oprava chyby v dokumentaci:
1.2.1 - 2026-02-01 - Oprava formátu sort parametru v příkladu

Přidání nového parametru:
1.3.0 - 2026-02-05 - Přidání query parametru include=relations

Smazání endpointu (breaking change):
2.0.0 - 2026-02-10 - Smazání deprecated endpointu GET /products/legacy
```

### Changelog tabulka - správný formát

```markdown

## Changelog

| Verze | Datum | Popis změny |
|-------|-------|------------|
| 1.3.1 | 2026-02-04 | Oprava chyby - doplnění chybějícího parametru `include` |
| 1.3.0 | 2026-02-02 | Přidání query parametru `include=relations` pro eager loading |
| 1.2.0 | 2026-01-28 | Přidán filtr `filter[inStock]` a `filter[category]` |
| 1.1.0 | 2026-01-20 | Přidán nový endpoint `POST /products/bulk-update` |
| 1.0.0 | 2026-01-15 | Iniciální verze dokumentace |
```

**Poznámky k changelog:**
- Vždy seřazeno od NEJNOVĚJŠÍ verze nahoře
- Datum ve formátu YYYY-MM-DD (ISO)
- Popis stručný, max 1 věta
- Jasně říct: "Přidáno", "Smazáno", "Opraveno", "Aktualizováno"

### Co se updatuje společně

Když aktualizuješ dokumentaci, VŽDY updatuj:

1. **Verzi v hlavičce:**
   ```markdown
   **Verze:** 1.2.0
   **Poslední aktualizace:** 2026-02-04
   ```

2. **Changelog tabulku:**
   - Přidej nový řádek na začátek
   - Nastav správnou verzi
   - Nastav dnešní datum
   - Napiš popis změny

3. **V git commitu:**
   ```
   docs: Update products API documentation to v1.2.0
   
   - Added filter[inStock] parameter
   - Updated response examples
   - Fixes #123
   ```

### Checklist pro aktualizaci

- [ ] Identifikuj typ změny (Major/Minor/Patch)
- [ ] Spočítej novou verzi (semver)
- [ ] Aktualizuj verzi v hlavičce (`**Verze:**`)
- [ ] Aktualizuj datum v hlavičce (`**Poslední aktualizace:**`)
- [ ] Přidej nový řádek do changelog tabulky
- [ ] Seřaž changelog od nejnovější verze
- [ ] Zkontroluj, že popis změny je jasný a stručný
- [ ] Commit se zmínkou verze (`v1.2.0` v message)

## Checklist

- [ ] Hlavička (název, verze, datum, osoba)
- [ ] Přehledová tabulka endpointů
- [ ] Detailní dokumentace každého endpointu
- [ ] Všechny parametry (URL, query, body) zdokumentovány
- [ ] Enum parametry s odkazem na endpoint
- [ ] Sekce "Související API" s odkazem na Enums
- [ ] Příklady requestů i responses
- [ ] Všechny HTTP status kódy
- [ ] Datové modely a struktura
- [ ] Enum typy
- [ ] Middleware a oprávnění
- [ ] Poznámky k chování
- [ ] Changelog tabulka
- [ ] **Registrace v `config/mcp-docs.php`** ⭐

## Proces

1. **Zkontroluj router** (`routes/api.php`)
2. **Najdi controller** a jeho metody
3. **Najdi Validation** - parametry a validace
4. **Najdi Resources** - struktura responses
5. **Najdi GenericSerializer** - společné metody
6. **Vytvoř dokumentaci** podle struktury
7. **Zaregistruj v MCP** (`config/mcp-docs.php`)
8. **Aktualizuj datum a verzi** v hlavičce

## Příklady

**Vzorové dokumenty:**
- `doc/api/admin/modules/products.md` (Admin REST)
- `doc/api/public/modules/articles.md` (Public REST)
- `doc/api/shared/modules/preview.md` (Shared - signed URLs)
- `doc/api/shared/modules/thumbnails.md` (Shared - media)

**⚠️ Důležité:**
- **Data jen z kódu** - nikdy si nevymýšlej
- **Kopíruj strukturu** z existujících dokumentů
- **Headers jako code block** - ne bullet list
- **Response s meta a links** - vždy
- **Registrace v MCP** - povinné
- **Changelog** - na konci dokumentu
- **Přesné oprávnění** - formát `entity.action.scope`

Reference: [Controller Structure](controller-structure.md), [Validation Structure](validation-structure.md), [Resource Structure](resource-structure.md)
