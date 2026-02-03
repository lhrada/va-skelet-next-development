---
title: JSON sloupce s validací
impact: HIGH
impactDescription: JSON bez validace může obsahovat nevalidní data
tags: migration, json, mariadb, validation
---

## JSON sloupce s validací

**Impact: HIGH**

Každý JSON sloupec v MariaDB MUSÍ mít validační constraint, aby se předešlo uložení nevalidního JSON.

**Incorrect:**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->json('data')->nullable()->default(null)
        ->comment('Additional data');
    $table->timestamps();
    
    // CHYBÍ JSON validační constraint!
});
```

**Correct:**

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->json('data')->nullable()->default(null)
        ->comment('Additional data in JSON format');
    $table->json('params')->nullable()->default(null)
        ->comment('Parameters');
    $table->timestamps();
    
    $table->comment('Product table');
});

// VŽDY přidej JSON validační constraint
DB::statement(
    "ALTER TABLE products ADD CONSTRAINT constraint_json_data CHECK (data IS NULL OR JSON_VALID(data));"
);
DB::statement(
    "ALTER TABLE products ADD CONSTRAINT constraint_json_params CHECK (params IS NULL OR JSON_VALID(params));"
);
```

**Naming convention pro constraints:**
- Formát: `constraint_json_{column_name}`
- Příklad: `constraint_json_data`, `constraint_json_structure_content`

**Pro překladové tabulky:**

```php
Schema::create('product_translations', function (Blueprint $table) {
    // ...
    $table->json('structure_content')->nullable()->default(null)
        ->comment('Content structure');
});

DB::statement(
    "ALTER TABLE product_translations ADD CONSTRAINT constraint_json_structure_content CHECK (structure_content IS NULL OR JSON_VALID(structure_content));"
);
```

**Komprese JSON sloupců:**

Pro velké JSON sloupce (často stock_response, external_data, api_response) použij kompresi pro úsporu místa:

```php
Schema::create('orders', function (Blueprint $table) {
    $table->id();
    $table->json('stock_response')->nullable()->default(null)
        ->comment('Stock API response - compressed');
    $table->json('external_data')->nullable()->default(null)
        ->comment('External system data - compressed');
    $table->timestamps();
});

// JSON validační constraint
DB::statement(
    "ALTER TABLE orders ADD CONSTRAINT constraint_json_stock_response CHECK (stock_response IS NULL OR JSON_VALID(stock_response));"
);
DB::statement(
    "ALTER TABLE orders ADD CONSTRAINT constraint_json_external_data CHECK (external_data IS NULL OR JSON_VALID(external_data));"
);

// Komprese sloupců pro úsporu místa
DB::statement('ALTER TABLE orders MODIFY stock_response JSON COMPRESSED=zlib NULL');
DB::statement('ALTER TABLE orders MODIFY external_data JSON COMPRESSED=zlib NULL');
```

**Kdy použít kompresi:**
- JSON sloupce s velkými daty (API responses, logs, external data)
- Sloupce `stock_response`, `api_response`, `external_data`, `logs`
- Úspora místa na disku při zachování funkčnosti

**Proč:**
- MariaDB umožňuje uložit nevalidní JSON bez constraintu
- Validační constraint zajišťuje integritu dat na DB úrovni
- Prevence runtime chyb při parsování JSON
- Komprese zlib šetří místo na disku bez dopadu na výkon

Reference: [Database Instructions](../../instructions/database.instructions.md)
