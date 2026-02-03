---
title: Router základy - aliasy, middleware, struktura
impact: HIGH
impactDescription: Správná struktura rout zajišťuje konzistentní API
tags: router, routes, middleware, aliases
---

## Router základy - aliasy, middleware, struktura

**Impact: HIGH**

Routy v projektu používají **aliasy pro controllery**, **middleware** pro autentizaci a **hierarchickou strukturu** pro admin/public sekce.

## Aliasy kontrolerů

**VŽDY používej aliasy** pro předejití kolizím názvů:

**Incorrect:**

```php
use App\Http\Controllers\Admin\Products\ProductController;
use App\Http\Controllers\Public\Products\ProductController;  // Kolize názvů!
```

**Correct:**

```php
use App\Http\Controllers\Admin\Products\ProductController as AdminProductController;
use App\Http\Controllers\Public\Products\ProductController as PublicProductController;
```

**Konvence pro aliasy:**

| Typ | Formát | Příklad |
|-----|--------|---------|
| **Admin** | `Admin{Název}Controller` | `AdminProductController`, `AdminArticleController` |
| **Public** | `Public{Název}Controller` | `PublicProductController`, `PublicArticleController` |

## Hierarchie rout

Projekt má následující hlavní sekce:

| Sekce | Prefix | Middleware | Použití |
|-------|--------|------------|---------|
| **Sessions** | `/sessions` | - | Přihlášení, autentizace |
| **OAuth** | `/oauth` | - | OAuth autentizace |
| **Account** | `/account` | - | Registrace, reset hesla |
| **Admin** | `/admin/*` | `auth:sanctum` | Administrace (chráněné) |
| **Self** | `/self` | `auth:sanctum` | Uživatelský profil |
| **Public** | `/public/*` | `auth.optional:sanctum`, `tenant` | Veřejné API |
| **Enums** | `/enums` | `auth:sanctum` | Číselníky |

## Middleware

### Základní middleware

```php
// Admin - vyžaduje přihlášení
Route::prefix('/admin')->middleware('auth:sanctum')->group(function () {
    // Admin routy
});

// Public - volitelné přihlášení
Route::prefix('/public')->middleware(['auth.optional:sanctum', 'tenant'])->group(function () {
    // Public routy
});

// Self - vyžaduje přihlášení
Route::prefix('/self')->middleware('auth:sanctum')->group(function () {
    // Self routy
});
```

### Middleware typy

| Middleware | Použití | Popis |
|------------|---------|-------|
| `auth:sanctum` | Admin, Self | **Vyžaduje** přihlášení |
| `auth.optional:sanctum` | Public | **Volitelné** přihlášení |
| `tenant` | Public | Multi-tenancy (kanál) |
| `vaTenant` | Admin (některé) | Via Aurea tenant |
| `localization` | Globálně | Lokalizace |
| `throttle:{name}` | Specifické | Rate limiting |
| `signed` | Preview | Signed URL |

### Throttle middleware

Pro rate limiting specifických endpointů:

```php
Route::post('/contact', [ContactController::class, 'store'])
    ->middleware('throttle:contact-form');

Route::post('/set', [AccountController::class, 'setPassword'])
    ->middleware('throttle:user-set-password');
```

### Cache middleware

Pro cachování odpovědí:

```php
Route::get('', [PublicProductController::class, 'index'])
    ->middleware(
        getRouteCacheMiddleware(
            tags: [Product::getCacheTag()],
            expire: 1 * 60 * 60  // 1 hodina
        )
    );
```

## Základní struktura routy

**Correct - Admin sekce:**

```php
Route::prefix('/admin')->middleware('auth:sanctum')->group(function () {
    Route::prefix('/products')
        ->where([
            'product' => RouteParamType::NumericId,
        ])
        ->group(function () {
            Route::get('', [AdminProductController::class, 'index']);
            Route::get('/{product}', [AdminProductController::class, 'show']);
            Route::post('', [AdminProductController::class, 'store']);
            Route::patch('/{product}', [AdminProductController::class, 'update']);
            Route::delete('/{product}', [AdminProductController::class, 'destroy']);
        });
});
```

**Correct - Public sekce:**

```php
Route::prefix('/public')->middleware(['auth.optional:sanctum', 'tenant'])->group(function () {
    Route::prefix('/products')
        ->where([
            'product' => RouteParamType::NumericId,
        ])
        ->group(function () {
            Route::get('', [PublicProductController::class, 'index']);
            Route::get('/{product}', [PublicProductController::class, 'show']);
            
            // Preview s signed URL
            Route::get('/p/{product}', [PublicProductController::class, 'show'])
                ->name(RouteName::ProductPreview->value)
                ->middleware(['signed']);
        });
});
```

## Model Binding

### Standardní binding

```php
Route::get('/{product}', [AdminProductController::class, 'show']);

// V kontroleru:
public function show(Product $product): Responsable
{
    return Responder::ok(new ProductResource($product));
}
```

### Custom binding (podle jiného sloupce)

```php
// Binding podle guid
Route::get('/{basket:guid}', [PublicBasketController::class, 'show']);

// Binding podle code
Route::get('/{coupon:code}', [AdminCouponController::class, 'show']);

// Binding podle key
Route::get('/keys/{article:key}', [PublicArticleController::class, 'show']);
```

### WithTrashed binding

Pro práci se soft-deleted záznamy:

```php
Route::get('/{product}', [AdminProductController::class, 'show'])
    ->withTrashed();

Route::patch('/{product}', [AdminProductController::class, 'update'])
    ->withTrashed();
```

**⚠️ Důležité:**
- **Aliasy VŽDY** - `Admin{Název}Controller`, `Public{Název}Controller`
- **Admin**: `auth:sanctum` - vyžaduje přihlášení
- **Public**: `auth.optional:sanctum`, `tenant` - volitelné přihlášení
- **`where()` před `group()`** - nikdy naopak
- **`withTrashed()`** pro soft-deleted záznamy
- **Custom binding** s `:sloupec` (např. `{product:guid}`)

Reference: [Router Admin REST](router-admin-rest.md), [Router Public REST](router-public-rest.md), [Router Constraints](router-constraints.md)