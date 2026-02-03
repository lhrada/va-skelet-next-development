---
title: PHPDoc aktualizace v modelu
impact: MEDIUM
impactDescription: Chybějící PHPDoc způsobuje problémy s IDE autocomplete
tags: model, phpdoc, documentation
---

## PHPDoc aktualizace v modelu

**Impact: MEDIUM**

Při přidání nového sloupce do migrace VŽDY aktualizuj PHPDoc v modelu s `@property` anotacemi.

**Incorrect:**

```php
// Migration: přidáváš nový sloupec 'priority'
Schema::table('products', function (Blueprint $table) {
    $table->integer('priority')->default(0)->after('sequence');
});

// Model: ZAPOMNĚL jsi aktualizovat PHPDoc!
/**
 * @property int $id
 * @property string $name
 * @property int $sequence
 * // CHYBÍ @property int $priority
 */
class Product extends Model
{
    // ...
}
```

**Correct:**

```php
// Migration: přidáváš nový sloupec
Schema::table('products', function (Blueprint $table) {
    $table->integer('priority')->default(0)->after('sequence')
        ->comment('Product priority');
});

// Model: AKTUALIZUJ PHPDoc!
/**
 * @property int $id
 * @property string $guid
 * @property string $name
 * @property int $sequence
 * @property int $priority            // ✓ PŘIDÁNO
 * 
 * @property Carbon $created_at
 * @property Carbon $updated_at
 * 
 * @property Category|null $category
 */
class Product extends Model
{
    protected $fillable = [
        'name',
        'sequence',
        'priority',  // ✓ Přidej i do $fillable pokud je mass assignable
    ];
}
```

**Struktura PHPDoc:**

```php
/**
 * Základní properties
 * @property int $id
 * @property string $guid
 * @property Key|null $key
 * 
 * Business properties
 * @property string $name
 * @property decimal $price
 * 
 * Publikační properties
 * @property bool $published
 * @property Carbon|null $published_from
 * @property Carbon|null $published_to
 * 
 * Systémové properties
 * @property bool $protected
 * @property int $sequence
 * @property int $priority
 * 
 * Timestamps
 * @property Carbon|null $deleted_at
 * @property Carbon $created_at
 * @property Carbon $updated_at
 * 
 * User tracking
 * @property int|null $created_by
 * @property User|null $createdBy
 * @property int|null $updated_by
 * @property User|null $updatedBy
 * 
 * Relationships
 * @property Category|null $category
 * @property Image[]|Collection $images
 */
```

**Proč:**
- IDE autocomplete
- Static analysis (PHPStan, Psalm)
- Dokumentace pro ostatní vývojáře
- Type safety
