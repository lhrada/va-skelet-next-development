---
title: Povinné sloupce v každé tabulce
impact: CRITICAL
impactDescription: Chybějící povinné sloupce způsobí problémy v celé aplikaci
tags: migration, database, structure
---

## Povinné sloupce v každé tabulce

**Impact: CRITICAL**

Každá nová tabulka MUSÍ obsahovat tyto povinné sloupce pro konzistenci napříč celou aplikací.

**Incorrect:**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    // CHYBÍ guid, timestamps
});
```

**Correct:**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    
    // GUID - povinné
    $table->string('guid', 36)->unique()
        ->collation('utf8_general_ci')
        ->comment('Global Unique Identifier');
    
    // Vaše sloupce
    $table->string('name');
    $table->decimal('price', 24, 14);
    
    // Timestamps - povinné
    $table->timestamps(); // created_at, updated_at
    
    // Tabulka komentář - povinné
    $table->comment('Stores product information for the e-shop');
});
```

**Povinné sloupce:**
1. `id()` - Primary key
2. `guid` - Global Unique Identifier (36 chars, unique, utf8_general_ci collation)
3. `timestamps()` - created_at, updated_at
4. `comment()` - Komentář k tabulce v angličtině

**Volitelné standardní sloupce:**
- `key` - Interní identifikátor (vyžaduje Enum třídu, utf8_general_ci collation)
- `softDeletes()` - Soft delete support
- `created_by`, `updated_by` - User tracking
- `protected` - Ochrana před smazáním
- `published`, `published_from`, `published_to` - Publikační sloupce
- `sequence`, `priority` - Řazení
- `old_id`, `old_ids` - Migrace ze starého systému

**⚠️ Collation pro "řídící" sloupce:**

Sloupce, které slouží jako identifikátory, klíče nebo typy (guid, key, type, locale, atd.) používají **`utf8_general_ci`** místo výchozí `utf8mb4` **kvůli velikosti indexu**.

**Důvod:** utf8_general_ci používá 1 byte na znak (ASCII), zatímco utf8mb4 používá až 4 bytes. MariaDB při vytváření indexu alokuje prostor pro maximální délku, takže index na `VARCHAR(255)` s utf8mb4 zabírá 1020 bytes (255 × 4), ale s utf8_general_ci jen 255 bytes - **4× menší index!**

```php
// Řídící sloupce s utf8_general_ci
$table->string('guid', 36)->unique()
    ->collation('utf8_general_ci')
    ->comment('Global Unique Identifier');

$table->string('key', 255)->unique()->nullable()->default(null)
    ->collation('utf8_general_ci')
    ->comment('Internal unique identifier');

$table->string('type', 64)
    ->collation('utf8_general_ci')
    ->comment('Type identifier');

$table->string('locale', 64)->index()
    ->collation('utf8_general_ci')
    ->comment('Locale - cs_CZ, en_US, ...');

// Běžné textové sloupce NEMAJÍ collation (používají výchozí utf8mb4)
$table->string('name', 255)
    ->comment('Product name');

$table->text('description')->nullable()
    ->comment('Description text');
```

**Kdy použít `utf8_general_ci`:**
- `guid` - Global Unique Identifier
- `key` - Interní identifikátor
- `type` - Typ záznamu/entity
- `locale` - Jazykové kódy (cs_CZ, en_US)
- `currency` - Měnové kódy (CZK, EUR)
- `sku`, `ean` - Produktové kódy
- `as` - Enum hodnoty (v pivot tabulkách)
- Jakékoli sloupce sloužící jako identifikátory nebo enum hodnoty

**Kdy NEPOUŽÍVAT collation (výchozí utf8mb4):**
- Běžné textové sloupce (`name`, `title`, `description`)
- Email adresy, URL adresy
- Veškerý user-generated content s diakritikou

**Více informací:** Viz [migration-collation](migration-collation.md) pravidlo
