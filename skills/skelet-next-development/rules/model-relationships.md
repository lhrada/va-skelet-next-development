---
title: Model Relationships - Eloquent vztahy
impact: HIGH
impactDescription: Správné vztahy jsou klíčové pro fungování aplikace
tags: model, relationships, eloquent, foreign-keys
---

## Model Relationships - Eloquent vztahy

**Impact: HIGH**

Všechny Eloquent vztahy musí mít explicitní return type hints a správné pojmenování podle konvencí.

**Incorrect:**

```php
class Product extends Model
{
    // CHYBÍ return type hint
    public function category()
    {
        return $this->belongsTo(Category::class);
    }
    
    // ŠPATNÝ název metody - mělo by být createdBy (camelCase)
    public function created_by()
    {
        return $this->belongsTo(User::class, 'created_by');
    }
}
```

**Correct:**

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Database\Eloquent\Relations\MorphTo;
use Illuminate\Database\Eloquent\Relations\MorphMany;

/**
 * @property int|null $category_id
 * @property int|null $created_by
 * @property int|null $updated_by
 * 
 * @property Category|null $category
 * @property User|null $createdBy
 * @property User|null $updatedBy
 * @property Image[]|Collection $images
 * @property Tag[]|Collection $tags
 */
final class Product extends Model
{
    // BelongsTo - N:1
    public function category(): BelongsTo
    {
        return $this->belongsTo(Category::class);
    }

    // BelongsTo s custom foreign key
    public function createdBy(): BelongsTo
    {
        return $this->belongsTo(User::class, 'created_by');
    }

    public function updatedBy(): BelongsTo
    {
        return $this->belongsTo(User::class, 'updated_by');
    }

    // HasMany - 1:N
    public function images(): HasMany
    {
        return $this->hasMany(Image::class);
    }

    // BelongsToMany - N:N (pivot tabulka)
    public function tags(): BelongsToMany
    {
        return $this->belongsToMany(Tag::class, 'product_tag')
            ->withPivot('sequence', 'main')
            ->withTimestamps();
    }

    // BelongsToMany s polymorfní pivot tabulkou
    public function relatedProducts(): BelongsToMany
    {
        return $this->belongsToMany(
            Product::class,
            'products_products',
            'parent_product_id',
            'child_product_id'
        )
            ->withPivot('as', 'sequence', 'price', 'obligatory')
            ->withTimestamps();
    }

    // MorphTo - polymorfní vztah
    public function objectable(): MorphTo
    {
        return $this->morphTo('objectable');
    }

    // MorphMany - polymorfní 1:N
    public function activities(): MorphMany
    {
        return $this->morphMany(Activity::class, 'subject');
    }
}
```

**Typy vztahů:**

| Vztah | Return Type | Použití |
|-------|-------------|---------|
| BelongsTo | `BelongsTo` | N:1 (má foreign key) |
| HasMany | `HasMany` | 1:N (nemá foreign key) |
| HasOne | `HasOne` | 1:1 (nemá foreign key) |
| BelongsToMany | `BelongsToMany` | N:N (pivot tabulka) |
| MorphTo | `MorphTo` | Polymorfní vztah (má _type a _id) |
| MorphMany | `MorphMany` | Polymorfní 1:N |
| MorphOne | `MorphOne` | Polymorfní 1:1 |

**Konvence pojmenování:**

```php
// Foreign key v DB: category_id
// Metoda: category() (bez Id suffixu)
public function category(): BelongsTo
{
    return $this->belongsTo(Category::class);
}

// Foreign key v DB: created_by
// Metoda: createdBy() (camelCase)
public function createdBy(): BelongsTo
{
    return $this->belongsTo(User::class, 'created_by');
}

// Pivot tabulka: product_tag
// Metoda: tags() (množné číslo)
public function tags(): BelongsToMany
{
    return $this->belongsToMany(Tag::class);
}
```

**PHPDoc pro vztahy:**

```php
/**
 * Foreign keys (scalar values)
 * @property int|null $category_id
 * @property int|null $created_by
 * 
 * Relationships (objects/collections)
 * @property Category|null $category
 * @property User|null $createdBy
 * @property Image[]|Collection $images
 * @property Tag[]|Collection $tags
 */
```

**⚠️ Důležité:**
- **Vždy** explicitní return type hint
- Metody v **camelCase** (createdBy, ne created_by)
- PHPDoc obsahuje foreign key I vztah
- Pro N:N používej `withPivot()` a `withTimestamps()`
- Pro polymorfní vztahy specifikuj morph name ('objectable')

Reference: [Laravel Eloquent Relationships](https://laravel.com/docs/eloquent-relationships)
