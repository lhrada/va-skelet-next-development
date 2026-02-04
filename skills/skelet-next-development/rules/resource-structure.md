---
title: Resource struktura a serializer
impact: HIGH
impactDescription: Resources zajišťují konzistentní formát API odpovědí
tags: resource, api, camelCase, serializer
---

## Resource struktura a serializer

**Impact: HIGH**

Resources transformují modely do API odpovědí. Používáme **camelCase** (NE snake_case) a `GenericSerializer` pro formátování.

**Namespace:**
- **Admin**: `App\Http\Controllers\Admin\{Modul}\{Název}Resource`
- **Admin List**: `App\Http\Controllers\Admin\{Modul}\{Název}ListResource`
- **Admin Enum**: `App\Http\Controllers\Admin\{Modul}\{Název}EnumResource`
- **Public**: `App\Http\Controllers\Public\{Modul}\{Název}Resource`

**Incorrect:**

```php
<?php

namespace App\Http\Controllers\Admin\Products;

use Illuminate\Http\Resources\Json\JsonResource;

// CHYBÍ declare(strict_types=1)
// CHYBÍ final
// CHYBÍ @mixin PHPDoc
class ProductResource extends JsonResource
{
    public function toArray($request)  // CHYBÍ Request type hint, return type
    {
        return [
            'created_at' => $this->created_at,  // ŠPATNĚ - snake_case místo camelCase
            'user_id' => $this->user_id,        // ŠPATNĚ - snake_case
        ];
    }
}
```

**Correct:**

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Admin\Products;

use App\Http\RequestHelper;
use App\Models\Product;
use Frame\Serializers\GenericSerializer;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

/**
 * @mixin Product
 */
final class ProductResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        $locale = RequestHelper::getDataLocale($request);
        
        return [
            // camelCase, NE snake_case!
            'id' => $this->id,
            'guid' => $this->guid,
            'key' => $this->key,
            'name' => $this->name,
            'sku' => $this->sku,
            // Objekt VA\Currency\Money::class [amount=>100.00, currency=>'USD']
            'price' => $this->price,
            
            // Boolean
            'published' => $this->published,
            'protected' => $this->protected,
            
            // Datumy přes GenericSerializer
            'publishedFrom' => GenericSerializer::toDateTime($this->published_from),
            'publishedTo' => GenericSerializer::toDateTime($this->published_to),
            'createdAt' => GenericSerializer::toDateTime($this->created_at),
            'updatedAt' => GenericSerializer::toDateTime($this->updated_at),
            'deletedAt' => GenericSerializer::toDateTime($this->deleted_at),
            
            // Vztahy - camelCase název
            'category' => new CategoryResource($this->whenLoaded('category')),
            'image' => new DocumentResource($this->whenLoaded('image')),
            'variants' => VariantResource::collection($this->whenLoaded('variants')),
            
            // User tracking
            'createdBy' => new UserResource($this->whenLoaded('createdBy')),
            'updatedBy' => new UserResource($this->whenLoaded('updatedBy')),
        ];
    }
}
```

## Typy Resources

### 1. {Model}Resource - Detail záznamu

Pro detail (`show` endpoint). Obsahuje **všechny data** včetně vztahů.

```php
/**
 * @mixin Product
 */
final class ProductResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        $locale = RequestHelper::getDataLocale($request);
        
        return [
            'id' => $this->id,
            'name' => $this->name,
            'description' => $this->description,
            'price' => GenericSerializer::toMoney($this->price),
            
            // Všechny vztahy
            'category' => new CategoryResource($this->whenLoaded('category')),
            'variants' => VariantResource::collection($this->whenLoaded('variants')),
            'galleries' => GalleryResource::collection($this->whenLoaded('galleries')),
        ];
    }
}
```

### 2. {Model}ListResource - Seznam záznamů

Pro seznamy (`index` endpoint). Obsahuje **minimální data** pro rychlost.

```php
/**
 * @mixin Product
 */
final class ProductListResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'key' => $this->key,
            'name' => $this->name,
            'sku' => $this->sku,
            'price' => GenericSerializer::toMoney($this->price),
            'published' => $this->published,
            
            // Jen základní info o vztazích
            'categoryName' => $this->category?->name,
            
            'createdAt' => GenericSerializer::toDateTime($this->created_at),
        ];
    }
}
```

### 3. {Model}EnumResource - Výčtové seznamy

Pro selecty/dropdowny. Obsahuje **minimálně id, value, name** (value = id pro select komponenty).

```php
/**
 * @mixin Product
 */
final class ProductEnumResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        $locale = RequestHelper::getDataLocale($request);
        
        return [
            'id' => $this->id,
            'value' => $this->id,  // Pro select komponenty (value = id)
            'name' => $this->name, // Zobrazovaný text
            
            // Volitelně další užitečná data pro select
            'sku' => $this->sku,
            'code' => $this->code,
            'image' => $this->image ? new DocumentLinkResource($this->image, ThumbConfig::$forEnums) : null,
        ];
    }
}
```

## GenericSerializer

Třída `Frame\Serializers\GenericSerializer` zajišťuje konzistentní formátování.

### toDateTime()

```php
'createdAt' => GenericSerializer::toDateTime($this->created_at),
'publishedFrom' => GenericSerializer::toDateTime($this->published_from),

// Výstup: "2024-01-15T14:30:00+01:00" (ISO 8601)
```

### toMoney()

```php
'price' => GenericSerializer::toMoney($this->price),
'discount' => GenericSerializer::toMoney($this->discount),

// Výstup: "1234.56" (string s 2 des. místy)
```

### toDate()

```php
'birthDate' => GenericSerializer::toDate($this->birth_date),

// Výstup: "2024-01-15"
```

## camelCase konvenze

**VŽDY používej camelCase, NE snake_case:**

| Database (snake_case) | API (camelCase) |
|----------------------|-----------------|
| `created_at` | `createdAt` |
| `updated_at` | `updatedAt` |
| `deleted_at` | `deletedAt` |
| `published_from` | `publishedFrom` |
| `published_to` | `publishedTo` |
| `created_by` | `createdBy` |
| `updated_by` | `updatedBy` |
| `main_image_id` | `image` (bez Id suffixu!) |
| `user_id` | `user` (bez Id suffixu!) |

## Vztahy (relationships)

### whenLoaded()

Používej `whenLoaded()` pro lazy loading:

```php
// Jeden vztah
'category' => new CategoryResource($this->whenLoaded('category')),

// Kolekce
'variants' => VariantResource::collection($this->whenLoaded('variants')),

// Nullable
'image' => $this->when(
    $this->relationLoaded('image') && $this->image,
    new DocumentResource($this->image)
),
```

### Nested resources

```php
return [
    'id' => $this->id,
    'name' => $this->name,
    
    // Nested resource
    'category' => new CategoryResource($this->whenLoaded('category')),
    
    // Nested collection
    'tags' => TagResource::collection($this->whenLoaded('tags')),
    
    // Podmíněné vložení
    'secret' => $this->when($request->user()?->isAdmin(), $this->secret),
];
```

## Příklad kompletního Resource

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Admin\Products;

use App\Http\Controllers\Admin\Categories\CategoryResource;
use App\Http\Controllers\Admin\Documents\DocumentResource;
use App\Http\Controllers\Admin\Users\UserResource;
use App\Http\RequestHelper;
use App\Models\Product;
use Frame\Serializers\GenericSerializer;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

/**
 * @mixin Product
 */
final class ProductResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        $locale = RequestHelper::getDataLocale($request);
        
        return [
            // Basic
            'id' => $this->id,
            'guid' => $this->guid,
            'key' => $this->key,
            
            // Business
            'name' => $this->name,
            'sku' => $this->sku,
            'price' => GenericSerializer::toMoney($this->price),
            'stock' => $this->stock,
            
            // Publishing
            'published' => $this->published,
            'publishedFrom' => GenericSerializer::toDateTime($this->published_from),
            'publishedTo' => GenericSerializer::toDateTime($this->published_to),
            
            // System
            'protected' => $this->protected,
            'sequence' => $this->sequence,
            'priority' => $this->priority,
            
            // Timestamps
            'createdAt' => GenericSerializer::toDateTime($this->created_at),
            'updatedAt' => GenericSerializer::toDateTime($this->updated_at),
            'deletedAt' => GenericSerializer::toDateTime($this->deleted_at),
            
            // Relationships
            'category' => new CategoryResource($this->whenLoaded('category')),
            'image' => new DocumentResource($this->whenLoaded('image')),
            'createdBy' => new UserResource($this->whenLoaded('createdBy')),
            'updatedBy' => new UserResource($this->whenLoaded('updatedBy')),
        ];
    }
}
```

**⚠️ Důležité:**
- **`declare(strict_types=1);`** na začátku
- **`final class`** - resource nemá potomky
- **`@mixin {Model}`** PHPDoc pro IDE support
- **camelCase** pro všechny klíče (NE snake_case)
- **GenericSerializer** pro DateTime a Money
- **`whenLoaded()`** pro relationships
- **BEZ suffixu Id** - `user`, `category`, `image` (NE userId, categoryId)
- **Resource vs ListResource vs EnumResource** - podle použití
- **EnumResource VŽDY s `value` a `name`** - value = id pro select komponenty

Reference: [Controller REST Methods](controller-rest-methods.md), [Model Relationships](model-relationships.md)
