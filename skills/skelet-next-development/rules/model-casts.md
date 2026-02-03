---
title: Model Casts - datové typy
impact: MEDIUM
impactDescription: Správné casts zajišťují correct data types při práci s modelem
tags: model, casts, types, enum
---

## Model Casts - datové typy

**Impact: MEDIUM**

Používej metodu `casts()` místo property `$casts` a správně definuj všechny datové typy včetně Enum tříd.

**Incorrect:**

```php
class Product extends Model
{
    // Starý způsob - property místo metody
    protected $casts = [
        'price' => 'float',
        'published' => 'boolean',
    ];
    
    // CHYBÍ casts pro datetime sloupce
    // CHYBÍ cast pro key (Enum)
}
```

**Correct:**

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Domain\Products\Key;
use Domain\Products\Status;
use Illuminate\Database\Eloquent\Model;

final class Product extends Model
{
    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            // Enum casts
            'key' => Key::class,
            'status' => Status::class,
            
            // Numeric casts
            'price' => 'float',
            'stock' => 'integer',
            'sequence' => 'integer',
            'priority' => 'integer',
            
            // Boolean casts
            'published' => 'boolean',
            'protected' => 'boolean',
            'main' => 'boolean',
            
            // Datetime casts
            'published_from' => 'datetime',
            'published_to' => 'datetime',
            'valid_from' => 'datetime',
            'valid_to' => 'datetime',
            
            // JSON casts
            'data' => 'array',
            'params' => 'array',
            'metadata' => 'json',
            
            // Collection cast
            'tags' => 'collection',
        ];
    }
}
```

**Běžné typy castů:**

| Typ sloupce v DB | Cast | Příklad |
|------------------|------|---------|
| `string` (enum hodnota) | `EnumClass::class` | `Key::class`, `Status::class` |
| `decimal`, `float` | `'float'` | `'price' => 'float'` |
| `integer`, `bigint` | `'integer'` | `'stock' => 'integer'` |
| `boolean`, `tinyint(1)` | `'boolean'` | `'published' => 'boolean'` |
| `datetime`, `timestamp` | `'datetime'` | `'published_from' => 'datetime'` |
| `date` | `'date'` | `'birth_date' => 'date'` |
| `json` (array structure) | `'array'` | `'data' => 'array'` |
| `json` (keep as json) | `'json'` | `'metadata' => 'json'` |
| `json` (collection) | `'collection'` | `'tags' => 'collection'` |

**Enum cast:**

```php
// Domain/Products/Key.php
namespace Domain\Products;

enum Key: string
{
    case NewArrival = 'new-arrival';
    case BestSeller = 'best-seller';
    case OnSale = 'on-sale';
}

// Model
protected function casts(): array
{
    return [
        'key' => Key::class,
    ];
}

// Použití
$product->key = Key::BestSeller;
$product->key->value; // 'best-seller'
```

**⚠️ Důležité:**
- Používej **metodu** `casts()`, ne property `$casts`
- Vždy přidej cast pro enum sloupce (key, status, type, ...)
- Datetime sloupce MUSÍ mít cast na `'datetime'`
- Boolean sloupce MUSÍ mít cast na `'boolean'`
- JSON sloupce podle použití: `'array'` pro data, `'json'` pro raw json, `'collection'` pro kolekce

Reference: [Laravel 12 Casts Documentation](https://laravel.com/docs/eloquent-mutators#attribute-casting)