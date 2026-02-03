---
title: Model struktura a PHPDoc
impact: HIGH
impactDescription: Správná struktura a PHPDoc zajišťuje IDE autocomplete a type safety
tags: model, eloquent, phpdoc, structure
---

## Model struktura a PHPDoc

**Impact: HIGH**

Každý model musí mít správnou strukturu s kompletním PHPDoc obsahujícím všechny properties, vztahy a casts.

**Incorrect:**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

// CHYBÍ declare(strict_types=1)
// CHYBÍ PHPDoc s @property anotacemi
class Product extends Model
{
    protected $fillable = ['name', 'price'];
}
```

**Correct:**

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Domain\Products\Key;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Support\Carbon;

/**
 * Základní properties
 * @property int $id
 * @property string $guid
 * @property Key|null $key
 * 
 * Business properties
 * @property string $name
 * @property float $price
 * @property int $stock
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
 * Foreign keys
 * @property int|null $category_id
 * 
 * Relationships
 * @property Category|null $category
 * @property Image[]|Collection $images
 */
final class Product extends Model
{
    use SoftDeletes;

    /**
     * The attributes that are mass assignable.
     *
     * @var array<string>
     */
    protected $fillable = [
        'guid',
        'key',
        'name',
        'price',
        'stock',
        'published',
        'published_from',
        'published_to',
        'protected',
        'sequence',
        'priority',
        'category_id',
        'created_by',
        'updated_by',
    ];

    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'key' => Key::class,
            'price' => 'float',
            'stock' => 'integer',
            'published' => 'boolean',
            'published_from' => 'datetime',
            'published_to' => 'datetime',
            'protected' => 'boolean',
            'sequence' => 'integer',
            'priority' => 'integer',
        ];
    }

    // Relationships
    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    public function images(): HasMany
    {
        return $this->hasMany(Image::class);
    }

    public function createdBy(): BelongsTo
    {
        return $this->belongsTo(User::class, 'created_by');
    }

    public function updatedBy(): BelongsTo
    {
        return $this->belongsTo(User::class, 'updated_by');
    }
}
```

**Struktura PHPDoc:**
1. **Základní properties**: id, guid, key
2. **Business properties**: Specifické pro entitu
3. **Publikační properties**: published, published_from, published_to
4. **Systémové properties**: protected, sequence, priority
5. **Timestamps**: deleted_at, created_at, updated_at
6. **User tracking**: created_by, createdBy, updated_by, updatedBy
7. **Foreign keys**: {relation}_id
8. **Relationships**: Eloquent vztahy (@property-read nebo @property)

**Povinné prvky:**
- `declare(strict_types=1);` na začátku
- `final` class (pokud nemá subclass)
- Kompletní PHPDoc s @property anotacemi
- `$fillable` array
- `casts()` metoda (místo $casts property)
- Type hints pro relationship metody

Reference: [Migration PHPDoc Update](migration-phpdoc-update.md)