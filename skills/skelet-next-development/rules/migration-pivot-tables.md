---
title: Pivot (vazební) tabulky struktura
impact: MEDIUM
impactDescription: Špatná struktura pivot tabulek způsobuje problémy s many-to-many vztahy
tags: migration, pivot, many-to-many, relationships
---

## Pivot (vazební) tabulky struktura

**Impact: MEDIUM**

Pivot tabulky pro many-to-many vztahy musí mít standardizovanou strukturu s foreign keys, volitelným `as` sloupcem a timestampy.

**Incorrect:**

```php
Schema::create('product_category', function (Blueprint $table) {
    $table->integer('product_id'); // Chybí foreign key
    $table->integer('category_id'); // Chybí foreign key
    // CHYBÍ id, timestamps, soft deletes
});
```

**Correct - Standardní pivot:**

```php
Schema::create('product_category', function (Blueprint $table) {
    $table->id();
    
    $table->foreignId('product_id')
        ->comment('Product reference')
        ->constrained('products')
        ->cascadeOnUpdate()
        ->cascadeOnDelete();
    
    $table->foreignId('category_id')
        ->comment('Category reference')
        ->constrained('categories')
        ->cascadeOnUpdate()
        ->cascadeOnDelete();
    
    // Typ vazby (pokud potřeba) - s utf8_general_ci collation
    $table->string('as', 64)->nullable()->default(null)->index()
        ->collation('utf8_general_ci')
        ->comment('Binding type - main/secondary/...');
    
    // Volitelné atributy vazby
    $table->integer('sequence')->default(0)->index()
        ->comment('Order in listings');
    
    $table->foreignId('created_by')->nullable()->default(null)
        ->constrained('users')
        ->nullOnDelete()
        ->cascadeOnUpdate();
    $table->foreignId('updated_by')->nullable()->default(null)
        ->constrained('users')
        ->nullOnDelete()
        ->cascadeOnUpdate();
    
    $table->softDeletes();
    $table->timestamps();
    
    $table->comment('Product to category relationship');
});
```

**Correct - Self-reference s 'as' sloupcem:**

```php
Schema::create('products_products', function (Blueprint $table) {
    $table->id();
    
    $table->foreignId('parent_product_id')
        ->comment('Product as parent')
        ->constrained(table: 'products')
        ->cascadeOnUpdate()
        ->cascadeOnDelete();
    
    $table->foreignId('child_product_id')
        ->comment('Product as child')
        ->constrained(table: 'products')
        ->cascadeOnUpdate()
        ->cascadeOnDelete();
    
    // Typ vazby (alternative/related/accessory/...)
    $table->string('as')->index()
        ->comment('Binding type - alternative/related/accessory/...');
    
    // Další atributy vazby
    $table->integer('sequence')->default(0)->index()
        ->comment('Order in listings');
    $table->decimal('price', 24, 14)->nullable()->default(null)
        ->comment('Absolute price override');
    $table->boolean('obligatory')->default(false)
        ->comment('Required item');
    
    $table->foreignId('created_by')->nullable()->default(null)
        ->constrained('users')
        ->nullOnDelete()
        ->cascadeOnUpdate();
    $table->foreignId('updated_by')->nullable()->default(null)
        ->constrained('users')
        ->nullOnDelete()
        ->cascadeOnUpdate();
    
    $table->softDeletes();
    $table->timestamps();
    
    $table->comment('Product to product relationships');
});
```

**Correct - Polymorfní vazba s business atributy:**

```php
Schema::create('profiles_objects', function (Blueprint $table) {
    $table->id();
    
    // Polymorfní vazba (morphTo)
    $table->string('objectable_type');
    $table->unsignedBigInteger('objectable_id');
    $table->index(['objectable_type', 'objectable_id']);
    
    // Foreign keys
    $table->foreignId('profile_id')->nullable()->default(null)
        ->comment('Profile reference')
        ->constrained('profiles')
        ->cascadeOnUpdate()
        ->cascadeOnDelete();
    
    $table->foreignId('organization_id')->nullable()->default(null)
        ->comment('Organization reference')
        ->constrained('profiles')
        ->cascadeOnUpdate()
        ->nullOnDelete();
    
    $table->foreignId('tag_id')->nullable()->default(null)
        ->comment('Tag reference')
        ->constrained('tags')
        ->cascadeOnUpdate()
        ->nullOnDelete();
    
    // Typ vazby (enum)
    $table->string('as')->index()
        ->collation('utf8_general_ci')
        ->comment('Relationship type - author/architect/investor/...');
    
    // Business atributy
    $table->integer('sequence')->default(0)->index()
        ->comment('Order in listings');
    $table->boolean('main')->default(false)
        ->comment('Is this the main relationship');
    
    // Separator pro UI (enum)
    $table->string('separator', 64)->nullable()->default(null)
        ->collation('utf8_general_ci')
        ->comment('Separator type for UI display');
    
    // Popisky vazby v různých jazycích - GENEROVANÉ PŘES LOOP
    foreach (\Domain\Locale::getConfig() as $locale) {
        $table->text('description_' . strtolower($locale['code']))
            ->nullable()
            ->default(null)
            ->comment('The description in ' . $locale['name']);
    }
    
    // User tracking
    $table->foreignId('created_by')->nullable()->default(null)
        ->constrained('users')
        ->nullOnDelete()
        ->cascadeOnUpdate();
    $table->foreignId('updated_by')->nullable()->default(null)
        ->constrained('users')
        ->nullOnDelete()
        ->cascadeOnUpdate();
    
    $table->timestamps();
    
    $table->comment('Polymorphic relationship between profiles and various objects');
});
```

**Povinné prvky:**
1. `id()` - Primary key
2. Foreign keys s `cascadeOnDelete()` a `cascadeOnUpdate()` (nebo polymorfní klíče)
3. `timestamps()` - Pro tracking změn
4. `softDeletes()` - Pro soft delete support (kromě polymorfních vazeb)
5. `created_by`, `updated_by` - User tracking
6. Komentář k tabulce

**Pro polymorfní vazby specificky:**
- `{relation}_type` a `{relation}_id` sloupce
- Index na obou sloupcích: `$table->index(['{relation}_type', '{relation}_id']);`
- Obvykle **BEZ** `softDeletes()` - polymorfní vazby se mažou hard delete

**Název tabulky:**
- Standardní: `{table1}_{table2}` (obě v množném čísle, alfabeticky)
  - Příklad: `category_product` (ne `product_category`)
- Self-reference: `{table}_{table}` s prefix sloupců (parent_, child_)
  - Příklad: `products_products` s `parent_product_id` a `child_product_id`

**Volitelné atributy vazby:**
- `as` - Typ vazby (pro různé druhy vztahů) - **VŽDY s `utf8_general_ci` collation**
- `sequence` - Pořadí
- `main` - Je to hlavní vazba? (boolean)
- `separator` - Oddělovač pro UI zobrazení (enum) - **s `utf8_general_ci` collation**
- `description_{locale}` - Popisky vazby v různých jazycích (text sloupce)
  - Generuj přes loop: `foreach (\Domain\Locale::getConfig() as $locale)`
  - Formát: `description_cs_cz`, `description_en_gb`, `description_de_de`
- `price` - Cena override
- `obligatory` - Povinnost
- Jakékoli další business atributy specifické pro vazbu

**Pro polymorfní vazby:**
- `{relation}_type` + `{relation}_id` - Polymorfní klíče (např. `objectable_type`, `objectable_id`)
- Index na `[{relation}_type, {relation}_id]`
- Žádný `softDeletes()` - polymorfní vazby obvykle nemají soft delete

**⚠️ Důležité pro collation:**
- Sloupce `as` a `separator` jsou řídící enum hodnoty → **`utf8_general_ci`**
- Popisky (`description_*`) jsou user content → **výchozí utf8mb4** (BEZ collation)

Reference: [Database Instructions](../../instructions/database.instructions.md)
