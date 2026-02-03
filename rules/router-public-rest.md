---
title: Router Public REST struktura
impact: HIGH
impactDescription: Standardní REST struktura pro public sekci
tags: router, public, rest, preview, signed
---

## Router Public REST struktura

**Impact: HIGH**

Public routy mají volitelnou autentizaci (`auth.optional:sanctum`) a `tenant` middleware pro multi-tenancy.

## Kompletní Public struktura

```php
Route::prefix('/public')->middleware(['auth.optional:sanctum', 'tenant'])->group(function () {
    Route::prefix('/products')
        ->where([
            'product' => RouteParamType::NumericId,
        ])
        ->group(function () {
            // Listing
            Route::get('', [PublicProductController::class, 'index']);
            
            // Detail
            Route::get('/{product}', [PublicProductController::class, 'show']);
            
            // Detail podle klíče
            Route::get('/keys/{product:key}', [PublicProductController::class, 'show']);
            
            // Preview (signed URL)
            Route::get('/p/{product}', [PublicProductController::class, 'show'])
                ->name(RouteName::ProductPreview->value)
                ->middleware(['signed']);
        });
});
```

## Přehled Public endpointů

| Metoda | Cesta | Controller Metoda | Účel |
|--------|-------|-------------------|------|
| **GET** | `/` | `index` | Seznam záznamů |
| **GET** | `/{id}` | `show` | Detail podle ID |
| **GET** | `/keys/{key}` | `show` | Detail podle klíče |
| **GET** | `/p/{id}` | `show` | Preview (signed) |

**Public NEMÁ:**
- ❌ POST, PATCH, PUT, DELETE (žádné zápisy)
- ❌ actions, restore (žádné akce)
- ❌ withTrashed (žádné smazané záznamy)

## Middleware pro Public

```php
Route::prefix('/public')
    ->middleware([
        'auth.optional:sanctum',  // Volitelná autentizace
        'tenant',                 // Multi-tenancy (kanál)
    ])
    ->group(function () {
        // Public routy
    });
```

**Rozdíly oproti Admin:**
- `auth.optional:sanctum` místo `auth:sanctum` - přihlášení je volitelné
- `tenant` middleware - pro multi-tenancy
- Bez `withTrashed()` - public nevidí smazané záznamy

## Detail podle klíče

Pro SEO-friendly URL s klíčem:

```php
// Detail podle ID
Route::get('/{article}', [PublicArticleController::class, 'show']);

// Detail podle klíče (key sloupec)
Route::get('/keys/{article:key}', [PublicArticleController::class, 'show']);

// V kontroleru - stejná metoda pro obě cesty:
public function show(Article $article): Responsable
{
    return Responder::ok(new ArticleResource($article));
}
```

## Preview routy (signed URLs)

Pro náhledy nepublikovaných záznamů:

```php
Route::get('/p/{article}', [PublicArticleController::class, 'show'])
    ->name(RouteName::ArticlePreview->value)
    ->middleware(['signed']);
```

**Vzor preview URL:**
- `/p/{id}` - preview podle ID
- `/p/keys/{key}` - preview podle klíče

**Signed middleware zajišťuje:**
- URL s časovou platností
- Hash pro zabezpečení
- Jen pro autorizované uživatele

### Generování signed URL

```php
// V aplikaci:
$url = URL::temporarySignedRoute(
    RouteName::ArticlePreview->value,
    now()->addHours(24),
    ['article' => $article->id]
);

// Vygeneruje: /public/articles/p/123?expires=...&signature=...
```

## Cache na Public routách

Public routy často používají cache:

```php
Route::get('', [PublicProductController::class, 'index'])
    ->middleware(
        getRouteCacheMiddleware(
            tags: [Product::getCacheTag()],
            expire: 1 * 60 * 60  // 1 hodina
        )
    );

Route::get('/{product}', [PublicProductController::class, 'show'])
    ->middleware(
        getRouteCacheMiddleware(
            tags: [Product::getCacheTag()],
            expire: 24 * 60 * 60  // 24 hodin
        )
    );
```

### Vícenásobné cache tags

```php
Route::get('/sitemap', PublicSitemapController::class)
    ->middleware([
        getRouteCacheMiddleware(
            tags: [
                Product::getCacheTag(),
                Article::getCacheTag(),
                Tag::getCacheTag(),
            ],
            expire: 24 * 60 * 60
        )
    ]);
```

## Příklad komplexní Public struktury

```php
Route::prefix('/public')->middleware(['auth.optional:sanctum', 'tenant'])->group(function () {
    Route::prefix('/articles')
        ->where([
            'article' => RouteParamType::NumericId,
        ])
        ->group(function () {
            // Listing s cache
            Route::get('', [PublicArticleController::class, 'index'])
                ->middleware(
                    getRouteCacheMiddleware(
                        tags: [Article::getCacheTag()],
                        expire: 1 * 60 * 60
                    )
                );
            
            // Detail podle ID
            Route::get('/{article}', [PublicArticleController::class, 'show'])
                ->middleware(
                    getRouteCacheMiddleware(
                        tags: [Article::getCacheTag()],
                        expire: 24 * 60 * 60
                    )
                );
            
            // Detail podle klíče (SEO-friendly)
            Route::get('/keys/{article:key}', [PublicArticleController::class, 'show'])
                ->middleware(
                    getRouteCacheMiddleware(
                        tags: [Article::getCacheTag()],
                        expire: 24 * 60 * 60
                    )
                );
            
            // Preview (signed URL)
            Route::get('/p/{article}', [PublicArticleController::class, 'show'])
                ->name(RouteName::ArticlePreview->value)
                ->middleware(['signed']);
            
            // Preview podle klíče
            Route::get('/p/keys/{article:key}', [PublicArticleController::class, 'show'])
                ->name(RouteName::ArticlePreviewByKey->value)
                ->middleware(['signed']);
        });
});
```

## Rozdíly Admin vs Public

| Aspekt | Admin | Public |
|--------|-------|--------|
| **Autentizace** | `auth:sanctum` (povinná) | `auth.optional:sanctum` (volitelná) |
| **Middleware** | - | `tenant` |
| **Metody** | GET, POST, PATCH, PUT, DELETE | Jen GET |
| **withTrashed** | Ano | Ne |
| **actions** | Ano | Ne |
| **Preview** | Ne | Ano (signed) |
| **Cache** | Občas | Často |
| **Detail podle klíče** | Ne | Ano (`/keys/{key}`) |

**⚠️ Důležité:**
- **Public VŽDY s `auth.optional:sanctum` a `tenant`**
- **Jen GET metody** - žádné zápisy
- **Preview s `signed` middleware** - `/p/{id}`
- **Detail podle klíče** - `/keys/{key}` s custom binding
- **Cache často používána** - `getRouteCacheMiddleware()`
- **Bez `withTrashed()`** - public nevidí smazané

Reference: [Router Basics](router-basics.md), [Router Admin REST](router-admin-rest.md), [Router Cache](router-cache.md)
