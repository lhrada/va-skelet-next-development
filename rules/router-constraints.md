---
title: Router constraints a RouteParamType
impact: MEDIUM
impactDescription: Constraints zajišťují validaci route parametrů
tags: router, constraints, validation, routeparamtype
---

## Router constraints a RouteParamType

**Impact: MEDIUM**

Používej `RouteParamType` enum pro validaci route parametrů. Constraints musí být **před** `group()`.

## RouteParamType enum

```php
use Domain\RouteParamType;

Route::prefix('/products')
    ->where([
        'product' => RouteParamType::NumericId,  // pouze čísla
        'key' => RouteParamType::String,         // string bez spec. znaků
        'guid' => RouteParamType::Guid,          // UUID formát
    ])
    ->group(function () {
        // routy
    });
```

## Dostupné typy

| Typ | Použití | Regex | Příklad |
|-----|---------|-------|---------|
| `NumericId` | ID záznamu | `[0-9]+` | `123`, `456` |
| `String` | Klíč, slug | `[a-z0-9-_]+` | `article-slug`, `my-key` |
| `Guid` | UUID | UUID formát | `550e8400-e29b-41d4-a716-446655440000` |
| `SHA1` | Hash | SHA1 formát | `356a192b7913b04c54574d18c28d46e6395428ab` |
| `Day` | Den | `1-31` | `1`, `15`, `31` |
| `Month` | Měsíc | `1-12` | `1`, `6`, `12` |
| `enum(Type::class)` | PHP Enum | Enum values | `active`, `inactive` |

## Umístění where() - DŮLEŽITÉ!

**VŽDY před `group()`:**

**Correct:**

```php
Route::prefix('/products')
    ->where([
        'product' => RouteParamType::NumericId,
    ])
    ->group(function () {
        Route::get('/{product}', [AdminProductController::class, 'show']);
    });
```

**Incorrect:**

```php
Route::prefix('/products')
    ->group(function () {
        Route::get('/{product}', [AdminProductController::class, 'show']);
    })
    ->where([
        'product' => RouteParamType::NumericId,  // ŠPATNĚ - až po group()
    ]);
```

## Více parametrů

```php
Route::prefix('/orders')
    ->where([
        'order' => RouteParamType::NumericId,
        'item' => RouteParamType::NumericId,
        'hash' => RouteParamType::SHA1,
    ])
    ->group(function () {
        Route::get('/{order}', [AdminOrderController::class, 'show']);
        Route::get('/{order}/items/{item}', [AdminOrderItemController::class, 'show']);
        Route::get('/{order}/track/{hash}', [PublicOrderController::class, 'track']);
    });
```

## Enum constraints

Pro PHP enum parametry:

```php
use Domain\Orders\OrderStatus;

Route::prefix('/orders')
    ->where([
        'order' => RouteParamType::NumericId,
        'status' => RouteParamType::enum(OrderStatus::class),
    ])
    ->group(function () {
        Route::get('/status/{status}', [AdminOrderController::class, 'byStatus']);
    });

// URL: /admin/orders/status/pending
// OrderStatus enum musí mít case Pending
```

## Custom binding vs Constraints

### Custom binding (podle sloupce)

```php
// Binding podle guid sloupce
Route::get('/{basket:guid}', [PublicBasketController::class, 'show'])
    ->where(['basket' => RouteParamType::Guid]);

// Binding podle key sloupce
Route::get('/keys/{article:key}', [PublicArticleController::class, 'show'])
    ->where(['article' => RouteParamType::String]);

// Binding podle code sloupce
Route::get('/{coupon:code}', [AdminCouponController::class, 'show'])
    ->where(['coupon' => RouteParamType::String]);
```

### Standardní binding (podle ID)

```php
Route::get('/{product}', [AdminProductController::class, 'show'])
    ->where(['product' => RouteParamType::NumericId]);
```

## Příklady podle typu

### NumericId

```php
->where(['product' => RouteParamType::NumericId])

// Platné: /products/123
// Neplatné: /products/abc, /products/12.5
```

### String

```php
->where(['key' => RouteParamType::String])

// Platné: /keys/my-article-slug
// Neplatné: /keys/My Article!, /keys/článek
```

### Guid

```php
->where(['basket' => RouteParamType::Guid])

// Platné: /baskets/550e8400-e29b-41d4-a716-446655440000
// Neplatné: /baskets/123, /baskets/invalid-guid
```

### Day a Month

```php
->where([
    'day' => RouteParamType::Day,
    'month' => RouteParamType::Month,
])

// Platné: /calendar/2024/12/25
// Neplatné: /calendar/2024/13/25 (month > 12)
// Neplatné: /calendar/2024/12/32 (day > 31)
```

## Vnořené routy s constraints

```php
Route::prefix('/products')
    ->where([
        'product' => RouteParamType::NumericId,
        'variant' => RouteParamType::NumericId,
        'image' => RouteParamType::NumericId,
    ])
    ->group(function () {
        Route::get('/{product}', [AdminProductController::class, 'show']);
        
        Route::prefix('/{product}/variants')->group(function () {
            Route::get('/{variant}', [AdminVariantController::class, 'show']);
            
            Route::prefix('/{variant}/images')->group(function () {
                Route::get('/{image}', [AdminImageController::class, 'show']);
            });
        });
    });

// URL: /admin/products/123/variants/456/images/789
// Všechny parametry validovány jako NumericId
```

## Kombinace s withTrashed

```php
Route::prefix('/products')
    ->where(['product' => RouteParamType::NumericId])
    ->group(function () {
        Route::get('/{product}', [AdminProductController::class, 'show'])
            ->withTrashed();
        
        Route::patch('/{product}', [AdminProductController::class, 'update'])
            ->withTrashed();
    });
```

## Příklad kompletní struktury

```php
Route::prefix('/admin')->middleware('auth:sanctum')->group(function () {
    Route::prefix('/articles')
        ->where([
            'article' => RouteParamType::NumericId,
            'gallery' => RouteParamType::NumericId,
            'image' => RouteParamType::NumericId,
        ])
        ->group(function () {
            // Základní CRUD
            Route::get('', [AdminArticleController::class, 'index']);
            Route::get('/{article}', [AdminArticleController::class, 'show'])->withTrashed();
            Route::post('', [AdminArticleController::class, 'store']);
            Route::patch('/{article}', [AdminArticleController::class, 'update'])->withTrashed();
            Route::delete('/{article}', [AdminArticleController::class, 'destroy'])->withTrashed();
            
            // Sub-resource - galerie
            Route::prefix('/{article}/galleries')->group(function () {
                Route::get('', [AdminArticleController::class, 'galleries']);
                Route::post('', [AdminArticleController::class, 'storeGallery']);
                
                Route::prefix('/{gallery}/images')->group(function () {
                    Route::post('', [AdminGalleryController::class, 'storeImage']);
                    Route::delete('/{image}', [AdminGalleryController::class, 'destroyImage']);
                });
            });
        });
});
```

**⚠️ Důležité:**
- **`where()` VŽDY před `group()`** - nikdy naopak
- **RouteParamType** pro všechny parametry
- **NumericId** pro ID záznamů
- **String** pro klíče, slugy, kódy
- **Guid** pro UUID
- **enum()** pro PHP enum typy
- **Custom binding** s `:sloupec` (např. `{product:guid}`)
- **Constraints platí pro celou group** - i pro vnořené routy

Reference: [Router Basics](router-basics.md), [Router Admin REST](router-admin-rest.md), [Router Public REST](router-public-rest.md)
