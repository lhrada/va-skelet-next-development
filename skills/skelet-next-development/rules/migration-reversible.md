---
title: Reverzibilní migrace (down metoda)
impact: MEDIUM
impactDescription: Bez down() nelze udělat rollback
tags: migration, rollback, down
---

## Reverzibilní migrace (down metoda)

**Impact: MEDIUM**

Každá migrace MUSÍ být reverzibilní - implementuj vždy metodu `down()` pro rollback.

**Incorrect:**

```php
public function up(): void
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->timestamps();
    });
}

public function down(): void
{
    // Prázdná down metoda - nelze rollback!
}
```

**Correct:**

```php
public function up(): void
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('guid', 36)->unique()
            ->collation('utf8_general_ci');
        $table->string('name');
        $table->timestamps();
        
        $table->comment('Product table');
    });
}

public function down(): void
{
    Schema::dropIfExists('products');
}
```

**Pro změny sloupců:**

```php
public function up(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->decimal('price', 24, 14)->nullable()->default(null)
            ->after('name')
            ->comment('Product price');
    });
}

public function down(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->dropColumn('price');
    });
}
```

**Pro JSON constraints:**

```php
public function up(): void
{
    Schema::table('products', function (Blueprint $table) {
        $table->json('data')->nullable()->default(null)
            ->comment('Additional data');
    });
    
    DB::statement(
        "ALTER TABLE products ADD CONSTRAINT constraint_json_data CHECK (data IS NULL OR JSON_VALID(data));"
    );
}

public function down(): void
{
    DB::statement("ALTER TABLE products DROP CONSTRAINT constraint_json_data;");
    
    Schema::table('products', function (Blueprint $table) {
        $table->dropColumn('data');
    });
}
```

**Proč:**
- Umožňuje rollback při chybě
- Usnadňuje development a debugging
- Best practice v Laravelu

Reference: [Laravel Migrations](https://laravel.com/docs/migrations#rolling-back-migrations)