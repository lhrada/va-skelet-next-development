---
title: Router constraints, RouteParamType a Cache Middleware
impact: MEDIUM
impactDescription: Constraints zajišťují validaci route parametrů, cache middleware optimalizuje výkon
tags: router, constraints, validation, routeparamtype, cache, middleware
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

## Cache middleware - getRouteCacheMiddleware()

Pro cachování route responses používej helper funkci `getRouteCacheMiddleware()` s **Stale-While-Revalidate (SWR)** patternem.

### Princip SWR

```
lifetime = fresh cache (servírována přímo)
grace = stale cache (servírována okamžitě + refresh na pozadí)
Total TTL = lifetime + grace (uloženo v cache)
```

**Příklad:** `getRouteCacheMiddleware([], days(7), hours(12))`
- **0-7 dní**: cache je fresh, servírována přímo
- **7 dní - 7.5 dní**: cache je stale, servírována okamžitě + refresh na pozadí
- **Po 7.5 dnech**: cache zcela expiruje

### Signatura funkce

```php
function getRouteCacheMiddleware(
    array $tags = [],              // Cache tagy pro selektivní mazání
    int|CarbonInterval|null $lifetime = null,  // Fresh perioda (default 4h)
    int|CarbonInterval|null $grace = null,     // Stale perioda (default 1h)
): string
```

### Základní použití

```php
use function getRouteCacheMiddleware;
use Carbon\CarbonInterval;

// S výchozími hodnotami (4h fresh + 1h grace = 5h TTL)
Route::get('/products', [PublicProductController::class, 'index'])
    ->middleware([getRouteCacheMiddleware()]);

// S vlastními časy pomocí helper funkcí
Route::get('/articles', [PublicArticleController::class, 'index'])
    ->middleware([
        getRouteCacheMiddleware(
            lifetime: days(7),    // 7 dní fresh
            grace: hours(12)      // 12 hodin stale
        )
    ]);

// S vlastními časy pomocí CarbonInterval
Route::get('/categories', [PublicCategoryController::class, 'index'])
    ->middleware([
        getRouteCacheMiddleware(
            lifetime: CarbonInterval::hours(6),
            grace: CarbonInterval::minutes(30)
        )
    ]);

// S časem v sekundách
Route::get('/tags', [PublicTagController::class, 'index'])
    ->middleware([
        getRouteCacheMiddleware(
            lifetime: 3600,  // 1 hodina
            grace: 300       // 5 minut
        )
    ]);
```

### Cache tagy a invalidace

```php
use App\Models\Product;
use App\Models\Article;

// S cache tagy pro selektivní invalidaci
Route::get('/sitemap', [PublicSitemapController::class, 'index'])
    ->middleware([
        getRouteCacheMiddleware(
            tags: [Product::getCacheTag(), Article::getCacheTag()],
            lifetime: days(1),
            grace: hours(2)
        )
    ]);

// Více tagů
Route::get('/feed', [PublicFeedController::class, 'index'])
    ->middleware([
        getRouteCacheMiddleware(
            tags: ['products', 'articles', 'categories'],
            lifetime: hours(12),
            grace: hours(1)
        )
    ]);
```

### Invalidace cache

Model implementující `CacheObject` interface automaticky invaliduje cache:

```php
// Model s cache supportem
class Product extends Model implements CacheObject
{
    use Cacheable;
    
    public const string CacheTagKey = 'products';
    
    protected static function booted(): void
    {
        // Automatická invalidace při změnách
        static::saved(fn() => self::clearCache());
        static::deleted(fn() => self::clearCache());
        static::restored(fn() => self::clearCache());
    }
}
```

Manuální invalidace podle tagů:

```php
use Spatie\ResponseCache\Facades\ResponseCache;

// Invalidace konkrétních tagů
ResponseCache::clear(['products', 'articles']);

// Invalidace všech cached responses
ResponseCache::flush();
```

### Bypass cache

Pro potlačení cachování pošli header:

```http
X-Cache-Control: no-cache
```

### Doporučené lifetime hodnoty

| Typ dat | Lifetime | Grace | Poznámka |
|---------|----------|-------|----------|
| **Statické stránky** | `days(7)` | `hours(12)` | Zřídka se mění |
| **Seznamy produktů** | `hours(6)` | `hours(1)` | Střední frekvence změn |
| **Detaily produktu** | `hours(2)` | `minutes(30)` | Častější změny |
| **Košík, ceny** | `minutes(15)` | `minutes(5)` | Dynamická data |
| **Enum seznamy** | `days(30)` | `days(1)` | Téměř neměnné |
| **Sitemap, Feed** | `days(1)` | `hours(2)` | Denní aktualizace |

### Příklady podle use case

#### Veřejný listing s paginací

```php
Route::get('/products', [PublicProductController::class, 'index'])
    ->middleware([
        getRouteCacheMiddleware(
            tags: [Product::getCacheTag()],
            lifetime: hours(6),
            grace: hours(1)
        )
    ]);
```

#### Sitemap s více modely

```php
Route::get('/sitemap.xml', [PublicSitemapController::class, 'index'])
    ->middleware([
        getRouteCacheMiddleware(
            tags: [
                Product::getCacheTag(),
                Article::getCacheTag(),
                Category::getCacheTag(),
            ],
            lifetime: days(1),
            grace: hours(2)
        )
    ]);
```

#### Preview endpoint (krátká cache)

```php
Route::get('/articles/preview/{article:guid}', [PublicArticleController::class, 'preview'])
    ->middleware([
        getRouteCacheMiddleware(
            tags: [Article::getCacheTag()],
            lifetime: minutes(5),
            grace: minutes(1)
        )
    ]);
```

#### Enum výčtové seznamy (dlouhá cache)

```php
Route::get('/enums/product-types', [EnumController::class, 'productTypes'])
    ->middleware([
        getRouteCacheMiddleware(
            lifetime: days(30),
            grace: days(1)
        )
    ]);
```

**⚠️ Důležité:**
- **Defaultní hodnoty:** lifetime 4h, grace 1h (celkem 5h TTL)
- **SWR pattern:** Po lifetime se stará cache stále servíruje (grace perioda) zatímco se refreshuje na pozadí
- **Cache tagy:** Používej `Model::getCacheTag()` pro automatickou invalidaci
- **Bypass:** Header `X-Cache-Control: no-cache` potlačí cachování
- **Helper funkce:** Používej `days()`, `hours()`, `minutes()` pro čitelnost
- **Model interface:** Model musí implementovat `CacheObject` pro automatickou invalidaci

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
