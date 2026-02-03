---
title: Collation pro sloupce - utf8_general_ci vs utf8mb4
impact: MEDIUM
impactDescription: utf8_general_ci výrazně snižuje velikost indexů (1 byte vs 4 bytes na znak)
tags: migration, collation, charset, performance
---

## Collation pro sloupce - utf8_general_ci vs utf8mb4

**Impact: MEDIUM**

Řídící sloupce (guid, key, type, locale) používají **`utf8_general_ci`** collation místo výchozí `utf8mb4` pro lepší výkon a konzistenci.

**Incorrect:**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    
    // ŠPATNĚ - chybí collation pro guid
    $table->string('guid', 36)->unique();
    
    // ŠPATNĚ - chybí collation pro key
    $table->string('key', 255)->unique()->nullable();
    
    // ŠPATNĚ - chybí collation pro type
    $table->string('type', 64);
    
    // Běžný textový sloupec - OK bez collation
    $table->string('name', 255);
    
    $table->timestamps();
});
```

**Correct:**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    
    // GUID - řídící sloupec s utf8_general_ci
    $table->string('guid', 36)->unique()
        ->collation('utf8_general_ci')
        ->comment('Global Unique Identifier');
    
    // Key - řídící sloupec s utf8_general_ci
    $table->string('key', 255)->unique()->nullable()->default(null)
        ->collation('utf8_general_ci')
        ->comment('Internal unique identifier');
    
    // Type - řídící sloupec s utf8_general_ci
    $table->string('type', 64)
        ->collation('utf8_general_ci')
        ->comment('Product type');
    
    // Běžný textový sloupec - BEZ collation (výchozí utf8mb4)
    $table->string('name', 255)
        ->comment('Product name');
    
    $table->text('description')->nullable()
        ->comment('Product description');
    
    $table->timestamps();
});
```

**Kdy použít `utf8_general_ci`:**

| Typ sloupce | Příklad | Důvod |
|-------------|---------|-------|
| GUID | `guid` | Identifikátor, ASCII znaky |
| Interní klíč | `key` | Identifikátor, ASCII znaky |
| Typy/kategorie | `type`, `status` | Enum hodnoty, ASCII |
| Locale kódy | `locale` | cs_CZ, en_US - ASCII |
| Měny | `currency` | CZK, EUR - ASCII |
| SKU/EAN kódy | `sku`, `ean` | Produktové kódy, ASCII |
| Enum hodnoty | `as` (v pivot tabulkách) | alternative/related - ASCII |

**Kdy NEPOUŽÍVAT collation (výchozí utf8mb4):**

| Typ sloupce | Příklad | Důvod |
|-------------|---------|-------|
| Názvy | `name`, `title` | User content, diakritika |
| Texty | `description`, `text` | User content, diakritika |
| Email | `email` | User content |
| URL | `url`, `slug` | User content |
| Adresy | `address1`, `city` | User content, diakritika |
| Poznámky | `note`, `comment` | User content |

**Příklady řídících sloupců:**

```php
// GUID - vždy utf8_general_ci
$table->string('guid', 36)->unique()
    ->collation('utf8_general_ci')
    ->comment('Global Unique Identifier');

// Key - vždy utf8_general_ci
$table->string('key', 255)->unique()->nullable()->default(null)
    ->collation('utf8_general_ci')
    ->comment('Internal unique identifier');

// Locale - vždy utf8_general_ci
$table->string('locale', 64)->index()
    ->collation('utf8_general_ci')
    ->comment('Locale - cs_CZ, en_US, de_DE, ...');

// Type - vždy utf8_general_ci
$table->string('type', 64)
    ->collation('utf8_general_ci')
    ->comment('Type identifier');

// Currency - vždy utf8_general_ci
$table->string('currency', 3)->nullable()
    ->collation('utf8_general_ci')
    ->comment('Currency code - CZK, EUR, USD');

// SKU - vždy utf8_general_ci
$table->string('sku', 64)->unique()
    ->collation('utf8_general_ci')
    ->comment('Stock Keeping Unit');

// Pivot "as" - vždy utf8_general_ci
$table->string('as')->index()
    ->collation('utf8_general_ci')
    ->comment('Binding type - alternative/related/accessory');
```

**Proč utf8_general_ci pro identifikátory:**
- **Menší velikost indexu** - utf8_general_ci používá 1 byte na znak (ASCII), zatímco utf8mb4 používá až 4 bytes. Při vytváření indexu na sloupci s utf8mb4 MariaDB alokuje prostor pro maximální délku (4 bytes × délka), což významně zvětšuje celkovou velikost indexu.
  - Příklad: `key VARCHAR(255)` s utf8mb4 = 255 × 4 = **1020 bytes v indexu**
  - Příklad: `key VARCHAR(255)` s utf8_general_ci = 255 × 1 = **255 bytes v indexu**
  - **Úspora: 4x menší index!**
- **Rychlejší porovnávání** - Case-insensitive pro ASCII znaky
- **Konzistence identifikátorů** - Identifikátory jsou vždy ASCII, nepotřebují Unicode
- **Žádné problémy s diakritikou** - Identifikátory ji nemají, nemůže dojít k neočekávanému chování

**Proč výchozí utf8mb4 pro běžný text:**
- Plná podpora Unicode (emoji, speciální znaky)
- Správné řazení s diakritikou
- User-generated content
