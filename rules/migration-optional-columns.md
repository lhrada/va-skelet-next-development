---
title: Standardní volitelné sloupce
impact: MEDIUM
impactDescription: Konzistentní sloupce napříč projektem usnadňují údržbu
tags: migration, columns, standard, optional, publishing, sorting
---

## Standardní volitelné sloupce

**Impact: MEDIUM**

Používej standardizované volitelné sloupce pro běžné funkce jako publikování, řazení, ochranu před smazáním a tracking starých ID.

**Publikační sloupce:**

```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('guid', 36)->unique()->collation('utf8_general_ci');
    
    // Publikační sloupce
    $table->boolean('published')->index()->default(false)
        ->comment('The main publication status');
    $table->dateTime('published_from')->index()->nullable()->default(null)
        ->comment('Published from date');
    $table->dateTime('published_to')->index()->nullable()->default(null)
        ->comment('Published to date');
    
    // Složený index pro efektivní dotazy na publikované záznamy
    $table->index(['published', 'published_from', 'published_to']);
    
    $table->timestamps();
});
```

**Řazení a priorita:**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('guid', 36)->unique()->collation('utf8_general_ci');
    
    // Sequence - pořadí v seznamech
    $table->integer('sequence')->default(0)->index()
        ->comment('Specifies the order in the listings');
    
    // Priority - priorita záznamu
    $table->integer('priority')->default(0)->index()
        ->comment('Object priority (higher = more important)');
    
    $table->timestamps();
});
```

**Ochrana před smazáním:**

```php
$table->boolean('protected')->default(false)
    ->comment('Object is protected (e.g., cannot be deleted)');
```

**Staré ID z předchozího systému:**

```php
// Jedno staré ID
$table->integer('old_id')->nullable()->default(null)->index()
    ->comment('Old ID from previous system');

// Více starých ID (JSON)
$table->json('old_ids')->nullable()->default(null)
    ->comment('Old IDs from previous systems (array)');

// Nezapomeň na JSON constraint
DB::statement(
    "ALTER TABLE table_name ADD CONSTRAINT constraint_json_old_ids CHECK (old_ids IS NULL OR JSON_VALID(old_ids));"
);
```

**Import tracking:**

```php
// Kód importu
$table->string('import_code')->nullable()->default(null)->index()
    ->comment('Identifier when importing');

// Batch importu
$table->bigInteger('import_batch')->nullable()->default(null)->index()
    ->comment('Import batch number');
```

**Interní poznámka:**

```php
$table->string('note', 1024)->nullable()->default(null)
    ->comment('Internal note for staff');
```

**User tracking (created_by, updated_by):**

```php
$table->foreignId('created_by')->nullable()->default(null)
    ->comment('User who created this record')
    ->constrained('users')
    ->nullOnDelete()
    ->cascadeOnUpdate();

$table->foreignId('updated_by')->nullable()->default(null)
    ->comment('User who last updated this record')
    ->constrained('users')
    ->nullOnDelete()
    ->cascadeOnUpdate();
```

**Kompletní příklad s volitelnými sloupci:**

```php
Schema::create('articles', function (Blueprint $table) {
    // Povinné
    $table->id();
    $table->string('guid', 36)->unique()->collation('utf8_general_ci')
        ->comment('Global Unique Identifier');
    
    // Business sloupce
    $table->string('title');
    $table->text('perex')->nullable();
    
    // Volitelné standardní sloupce
    $table->string('key', 255)->unique()->nullable()->default(null)
        ->collation('utf8_general_ci')
        ->comment('Internal unique identifier');
    
    // Publikace
    $table->boolean('published')->index()->default(false)
        ->comment('Publication status');
    $table->dateTime('published_from')->index()->nullable()->default(null)
        ->comment('Published from date');
    $table->dateTime('published_to')->index()->nullable()->default(null)
        ->comment('Published to date');
    $table->index(['published', 'published_from', 'published_to']);
    
    // Řazení
    $table->integer('sequence')->default(0)->index()
        ->comment('Order in listings');
    $table->integer('priority')->default(0)->index()
        ->comment('Priority');
    
    // Ochrana
    $table->boolean('protected')->default(false)
        ->comment('Protected from deletion');
    
    // Import tracking
    $table->integer('old_id')->nullable()->default(null)->index()
        ->comment('Old ID from previous system');
    $table->string('import_code')->nullable()->default(null)->index()
        ->comment('Import identifier');
    $table->bigInteger('import_batch')->nullable()->default(null)->index()
        ->comment('Import batch number');
    
    // Interní poznámka
    $table->string('note', 1024)->nullable()->default(null)
        ->comment('Internal staff note');
    
    // User tracking
    $table->foreignId('created_by')->nullable()->default(null)
        ->constrained('users')->nullOnDelete()->cascadeOnUpdate();
    $table->foreignId('updated_by')->nullable()->default(null)
        ->constrained('users')->nullOnDelete()->cascadeOnUpdate();
    
    // Timestamps
    $table->softDeletes();
    $table->timestamps();
    
    $table->comment('Articles for content management');
});
```

**Kdy použít které sloupce:**

| Sloupec | Kdy použít |
|---------|------------|
| `key` | Interní identifikátor pro speciální záznamy (vyžaduje Enum) |
| `published`, `published_from`, `published_to` | Publikování obsahu s časovým omezením |
| `sequence` | Manuální řazení záznamů v seznamech |
| `priority` | Automatické řazení podle důležitosti |
| `protected` | Ochrana důležitých záznamů před smazáním |
| `old_id`, `old_ids` | Migrace ze starého systému |
| `import_code`, `import_batch` | Tracking importovaných dat |
| `note` | Interní poznámky pro administrátory |
| `created_by`, `updated_by` | Sledování kdo vytvořil/upravil záznam |

**⚠️ Důležité:**
- Všechny volitelné sloupce mají `nullable()->default(null)` nebo explicitní default
- Složený index pro publikační sloupce (`['published', 'published_from', 'published_to']`)
- `sequence` a `priority` mají `default(0)` a `index()`
- `key` vyžaduje Enum třídu v `Domain\{Entity}\Key`

Reference: [Migration Required Columns](migration-required-columns.md), [Migration Indexes](migration-indexes.md)
