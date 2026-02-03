---
title: Router Admin REST struktura
impact: HIGH
impactDescription: Standardní REST struktura pro admin sekci
tags: router, admin, rest, crud
---

## Router Admin REST struktura

**Impact: HIGH**

Admin routy mají standardní REST strukturu s `auth:sanctum` middleware a podporou pro soft-deleted záznamy.

## Kompletní REST struktura

```php
Route::prefix('/admin')->middleware('auth:sanctum')->group(function () {
    Route::prefix('/products')
        ->where([
            'product' => RouteParamType::NumericId,
        ])
        ->group(function () {
            // Listing
            Route::get('', [AdminProductController::class, 'index']);
            
            // Detail
            Route::get('/{product}', [AdminProductController::class, 'show'])
                ->withTrashed();
            
            // Create
            Route::post('', [AdminProductController::class, 'store']);
            
            // Update - PATCH pro partial update
            Route::patch('/{product}', [AdminProductController::class, 'update'])
                ->withTrashed();
            
            // Update - PUT pro full replacement
            Route::put('/{product}', [AdminProductController::class, 'update'])
                ->withTrashed();
            
            // Soft Delete
            Route::delete('/{product}', [AdminProductController::class, 'destroy'])
                ->withTrashed();
            
            // Restore
            Route::patch('/{product}/restore', [AdminProductController::class, 'restore'])
                ->withTrashed();
            
            // Hromadné akce (více objektů)
            Route::post('/actions', [AdminProductController::class, 'actions']);
            
            // Akce na jednom objektu
            Route::post('/{product}/actions', [AdminProductController::class, 'productActions']);
        });
});
```

## Přehled REST endpointů

| Metoda | Cesta | Controller Metoda | Účel |
|--------|-------|-------------------|------|
| **GET** | `/` | `index` | Seznam záznamů |
| **GET** | `/{id}` | `show` | Detail záznamu |
| **POST** | `/` | `store` | Vytvoření |
| **PATCH** | `/{id}` | `update` | Částečná aktualizace |
| **PUT** | `/{id}` | `update` | Kompletní nahrazení |
| **DELETE** | `/{id}` | `destroy` | Soft delete |
| **PATCH** | `/{id}/restore` | `restore` | Obnovení |
| **POST** | `/actions` | `actions` | Hromadné akce |
| **POST** | `/{id}/actions` | `{model}Actions` | Akce na jednom objektu |

## PATCH vs PUT

```php
// PATCH - částečná aktualizace (jen poslané hodnoty)
Route::patch('/{product}', [AdminProductController::class, 'update'])
    ->withTrashed();

// PUT - kompletní nahrazení (všechny hodnoty)
Route::put('/{product}', [AdminProductController::class, 'update'])
    ->withTrashed();

// V kontroleru:
public function update(Request $request, Product $product, ProductValidation $validation): Responsable
{
    $validated = match ($request->getMethod()) {
        'PUT' => $validation->createOrReplace($request),    // Celý objekt
        default => $validation->update($request, $product), // Jen změny
    };
    
    // ...
}
```

## withTrashed()

Pro práci se soft-deleted záznamy:

```php
// Detail - zobrazit i smazané
Route::get('/{product}', [AdminProductController::class, 'show'])
    ->withTrashed();

// Update - aktualizovat i smazané
Route::patch('/{product}', [AdminProductController::class, 'update'])
    ->withTrashed();

// Delete - smazat i již smazané (force delete)
Route::delete('/{product}', [AdminProductController::class, 'destroy'])
    ->withTrashed();

// Restore - obnovit smazané
Route::patch('/{product}/restore', [AdminProductController::class, 'restore'])
    ->withTrashed();
```

**Kdy použít `withTrashed()`:**
- ✅ `show` - zobrazit detail i smazaného záznamu
- ✅ `update` - aktualizovat i smazaný záznam
- ✅ `destroy` - umožnit force delete
- ✅ `restore` - obnovit smazaný záznam
- ❌ `index` - NEPOUŽÍVEJ (trash se řeší query parametrem)
- ❌ `store` - NEPOUŽÍVEJ (vytváření nového záznamu)

## Actions endpoints

### Hromadné akce (více objektů)

```php
Route::post('/actions', [AdminProductController::class, 'actions']);

// Request:
// POST /admin/products/actions
// Body: { "type": "delete", "ids": [1, 2, 3] }
// nebo: { "type": "restore", "ids": [1, 2, 3] }
```

### Akce na jednom objektu

```php
Route::post('/{product}/actions', [AdminProductController::class, 'productActions']);

// Request:
// POST /admin/products/123/actions
// Body: { "type": "publish" }
// nebo: { "type": "duplicate" }
```

## Speciální endpointy

### Summaries

Pro agregované/sumární údaje:

```php
Route::get('/summaries', [AdminOrderController::class, 'summaries']);
```

### Exists

Pro kontrolu existence:

```php
Route::post('/exists', [AdminProductController::class, 'exists']);
Route::get('/exists', [AdminProductController::class, 'exists']);
Route::post('/{product}/exists', [AdminProductController::class, 'exists']);
Route::get('/{product}/exists', [AdminProductController::class, 'exists']);
```

### Multi-update

Pro hromadnou aktualizaci:

```php
Route::patch('/multi-update', [AdminProductMultiUpdateController::class, 'multiUpdate']);
```

## Vnořené routy (Sub-resources)

Pro vztahy parent-child:

```php
Route::prefix('/orders')
    ->where([
        'order' => RouteParamType::NumericId,
        'item' => RouteParamType::NumericId,
    ])
    ->group(function () {
        Route::get('/{order}', [AdminOrderController::class, 'show']);
        
        // Items jako sub-resource
        Route::prefix('/{order}/items')->group(function () {
            Route::get('', [AdminOrderController::class, 'items']);
            Route::post('', [AdminOrderController::class, 'storeItem']);
            Route::patch('/{item}', [AdminOrderItemController::class, 'update']);
            Route::delete('/{item}', [AdminOrderItemController::class, 'destroy']);
        });
    });
```

## Tenant middleware

Pro routy závislé na tenantovi/kanálu:

```php
Route::get('', [AdminArticleController::class, 'index'])
    ->middleware('vaTenant');  // nebo 'tenant'
```

## Příklad komplexní struktury

```php
Route::prefix('/admin')->middleware('auth:sanctum')->group(function () {
    Route::prefix('/products')
        ->where([
            'product' => RouteParamType::NumericId,
        ])
        ->group(function () {
            // Základní CRUD
            Route::get('', [AdminProductController::class, 'index']);
            Route::get('/{product}', [AdminProductController::class, 'show'])->withTrashed();
            Route::post('', [AdminProductController::class, 'store']);
            Route::patch('/{product}', [AdminProductController::class, 'update'])->withTrashed();
            Route::put('/{product}', [AdminProductController::class, 'update'])->withTrashed();
            Route::delete('/{product}', [AdminProductController::class, 'destroy'])->withTrashed();
            Route::patch('/{product}/restore', [AdminProductController::class, 'restore'])->withTrashed();
            
            // Actions
            Route::post('/actions', [AdminProductController::class, 'actions']);
            Route::post('/{product}/actions', [AdminProductController::class, 'productActions']);
            
            // Speciální endpointy
            Route::get('/summaries', [AdminProductController::class, 'summaries']);
            Route::post('/exists', [AdminProductController::class, 'exists']);
            
            // Sub-resources (varianty)
            Route::prefix('/{product}/variants')
                ->where(['variant' => RouteParamType::NumericId])
                ->group(function () {
                    Route::get('', [AdminProductController::class, 'variants']);
                    Route::post('', [AdminProductController::class, 'storeVariant']);
                    Route::patch('/{variant}', [AdminVariantController::class, 'update']);
                });
        });
});
```

**⚠️ Důležité:**
- **Admin VŽDY s `auth:sanctum`** - vyžaduje přihlášení
- **`withTrashed()`** na show, update, destroy, restore
- **PATCH vs PUT** - částečná vs kompletní aktualizace
- **`where()` před `group()`** - nikdy naopak
- **actions vs {model}Actions** - hromadné vs jednotlivé
- **Sub-resources** s vnořenými prefix

Reference: [Router Basics](router-basics.md), [Router Constraints](router-constraints.md), [Controller REST Methods](controller-rest-methods.md)
