---
title: Indexy - kdy a jak vytvářet
impact: MEDIUM
impactDescription: Chybějící nebo špatné indexy vedou k pomalým dotazům
tags: migration, indexes, performance, optimization
---

## Indexy - kdy a jak vytvářet

**Impact: MEDIUM**

Indexy jsou klíčové pro výkon databáze. Vytvárej je strategicky na sloupce používané v WHERE, ORDER BY a JOIN klauzulích.

**Kdy vytvořit index:**
- Sloupce používané často ve `WHERE` klauzulích
- Foreign keys (obvykle automaticky)
- Sloupce používané pro `ORDER BY`
- Sloupce používané pro `GROUP BY`
- Unikátní hodnoty

**Incorrect:**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('sku'); // CHYBÍ unique index
    $table->boolean('published'); // CHYBÍ index pro WHERE dotazy
    $table->integer('sequence'); // CHYBÍ index pro ORDER BY
    $table->foreignId('category_id')->constrained(); // OK - foreign key má automatický index
});
```

**Correct:**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    
    // Unique index pro SKU
    $table->string('sku', 64)->unique()
        ->comment('Stock Keeping Unit');
    
    // Standardní index pro často používané WHERE podmínky
    $table->boolean('published')->index()->default(false)
        ->comment('Publication status');
    
    // Index pro řazení
    $table->integer('sequence')->default(0)->index()
        ->comment('Order in listings');
    
    // Foreign key má automatický index
    $table->foreignId('category_id')
        ->comment('Product category')
        ->constrained('categories')
        ->cascadeOnUpdate()
        ->restrictOnDelete();
    
    // Složený index pro časté kombinace filtrů
    $table->dateTime('published_from')->nullable()->default(null);
    $table->dateTime('published_to')->nullable()->default(null);
    $table->index(['published', 'published_from', 'published_to']);
    
    $table->timestamps();
});
```

**Typy indexů:**

```php
// Standardní index
$table->index('column_name');
$table->string('email')->index();

// Unikátní index
$table->unique('column_name');
$table->string('sku')->unique();

// Složený index (více sloupců)
$table->index(['column1', 'column2', 'column3']);
$table->index(['published', 'published_from', 'published_to']);

// Unikátní složený index
$table->unique(['product_id', 'locale']);
$table->unique(['table_id', 'locale']); // Pro překladové tabulky
```

**Běžné use cases:**

```php
// Publikační sloupce - složený index
$table->boolean('published')->index()->default(false);
$table->dateTime('published_from')->index()->nullable()->default(null);
$table->dateTime('published_to')->index()->nullable()->default(null);
$table->index(['published', 'published_from', 'published_to']);

// Řazení
$table->integer('sequence')->default(0)->index();
$table->integer('priority')->default(0)->index();

// Unikátní identifikátory
$table->string('guid', 36)->unique();
$table->string('key', 255)->unique()->nullable()->default(null);
$table->string('sku', 64)->unique();

// Import tracking
$table->string('import_code')->nullable()->default(null)->index();
$table->bigInteger('import_batch')->nullable()->default(null)->index();
$table->integer('old_id')->nullable()->default(null)->index();

// Locale pro překladové tabulky
$table->string('locale', 64)->index();
$table->unique(['product_id', 'locale']);
```

**Pozor:**
- ❌ Nevytvářej indexy na všechny sloupce (overhead při INSERT/UPDATE)
- ❌ Nevytvářej indexy na low-cardinality sloupce (např. boolean s 2 hodnotami) pokud nejsou v WHERE
- ✅ Použij složené indexy pro časté kombinace filtrů
- ✅ Foreign keys mají automatický index

Reference: [Database Performance](../../instructions/database.instructions.md)
