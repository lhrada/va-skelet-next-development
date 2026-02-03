---
title: Controller REST metody
impact: HIGH
impactDescription: Standardní REST metody zajišťují konzistentní API rozhraní
tags: controller, rest, crud, authorization
---

## Controller REST metody

**Impact: HIGH**

Controller má standardní REST metody: index, show, store, update, destroy, restore. Každá metoda má specifickou odpovědnost.

## 1. index - Seznam záznamů

**Pro admin (staffSource):**

```php
public function index(Request $request): Responsable
{
    $this->authorize('viewAny', Product::class);
    
    $only = $this->getStaffOnly();
    $qo = new FilterBuilder()
        ->only(...$only)
        ->makeQueryObject($request);
    $dataLocale = RequestHelper::getDataLocale($request);
    $trash = RequestHelper::showTrash($request);
    
    return ProductListResource::collection(
        $this->service->staffSource(
            locale: $dataLocale,
            queryObject: $qo,
            trash: $trash,
        )
    );
}

private function getStaffOnly(): array
{
    return [
        'id',
        'key',
        'name',
        'sku',
        'price',
        'published',
        'deleted',
    ];
}
```

**Pro public (clientSource):**

```php
public function index(Request $request): Responsable
{
    // Bez authorize pro veřejný endpoint
    
    $only = $this->getClientOnly();
    $qo = new FilterBuilder()
        ->only(...$only)
        ->makeQueryObject($request);
    $dataLocale = RequestHelper::getDataLocale($request);
    
    return ProductListResource::collection(
        $this->service->clientSource(
            locale: $dataLocale,
            queryObject: $qo,
        )
    );
}

private function getClientOnly(): array
{
    return ['id', 'name', 'price'];  // Méně filtrů
}
```

## 2. show - Detail záznamu

**S route model binding:**

```php
public function show(
    Request $request,
    Product $product,
): Responsable {
    $this->authorize('view', $product);
    
    return Responder::ok(
        new ProductResource($product)
    );
}
```

**S getByKeyOrId() (klíč nebo ID):**

```php
public function show(
    Request $request,
    int|string $keyOrId,
): Responsable {
    $product = $this->service->getByKeyOrId(
        id: $keyOrId,
        trash: false,
        withRelations: ['category', 'image', 'variants']
    );
    
    $this->authorize('view', $product);
    
    return Responder::ok(
        new ProductResource($product)
    );
}
```

## 3. store - Vytvoření nového záznamu

```php
public function store(
    Request $request,
    ProductValidation $validation,
): Responsable {
    $this->authorize('create', Product::class);
    
    $dataLocale = RequestHelper::getDataLocale($request);
    $validated = $validation->createOrReplace($request);
    
    // Přidání created_by
    $data = $validated + [
        'created_by' => $request->user()?->id,
    ];
    
    $product = $this->service->upsert(
        locale: $dataLocale,
        model: null,  // NULL = nový záznam
        data: $data
    );
    
    return Responder::created(
        new ProductResource($product),
    );
}
```

## 4. update - Aktualizace záznamu

```php
public function update(
    Request $request,
    Product $product,
    ProductValidation $validation,
): Responsable {
    $this->authorize('update', $product);
    
    $dataLocale = RequestHelper::getDataLocale($request);
    
    // PUT = celý objekt (createOrReplace), PATCH = jen změny (update)
    $validated = match ($request->getMethod()) {
        'PUT' => $validation->createOrReplace($request),
        default => $validation->update($request, $product),
    };
    
    // Přidání updated_by
    $data = $validated + [
        'updated_by' => $request->user()?->id,
    ];
    
    $product = $this->service->upsert(
        locale: $dataLocale,
        model: $product,  // Existující model
        data: $data
    );
    
    return Responder::ok(
        new ProductResource($product),
    );
}
```

## 5. destroy - Smazání záznamu

```php
public function destroy(
    Request $request,
    Product $product,
): Responsable {
    $this->authorize('delete', $product);
    
    // Soft delete nebo force delete podle parametru
    $force = RequestHelper::force($request);
    if ($force) {
        $this->service->forceDelete($product);
    } else {
        $this->service->delete($product);
    }
    
    $result = [
        'message' => __('Záznam s id :id byl smazán', ['id' => $product->id]),
        'id' => $product->id,
    ];
    
    return Responder::ok($result);
}
```

## 6. restore - Obnovení smazaného záznamu

```php
public function restore(
    Request $request,
    Product $product,
): Responsable {
    $this->authorize('update', $product);
    
    $product->restore();
    
    $result = [
        'message' => __('Produkt :id byl obnoven', ['id' => $product->id]),
        'id' => $product->id,
    ];
    
    return Responder::ok($result);
}
```

## 7. actions - Hromadné operace

```php
use Domain\Actions;
use Symfony\Component\HttpFoundation\BinaryFileResponse;

public function actions(Request $request): Responsable|BinaryFileResponse
{
    $type = $request->input('type') ?? null;
    if (!$type) {
        throw new UnprocessableContent('Action type is required.')
            ->convey(__('Typ akce je povinný.'));
    }
    
    $filter = RequestHelper::getActionFilter($request);
    $locale = RequestHelper::getDataLocale($request);
    $trash = RequestHelper::showTrash($request);
    
    // Získání IDs - buď z filtru nebo přímo z requestu
    if ($filter !== null) {
        $only = $this->getStaffOnly();
        $qo = new FilterBuilder()
            ->only(...$only)
            ->makeQueryObject($request);
        $qo->setLimit(null);
        
        $builder = $this->service->staffBuilder(
            queryObject: $qo,
            trash: $trash,
            locale: $locale
        );
        $ids = $builder->pluck('id')->toArray();
    } else {
        $ids = $request->input('ids') ?? null;
        if (empty($ids)) {
            throw new UnprocessableContent('Action ids are required.')
                ->convey(__('Nejsou vybrány žádné položky.'));
        }
    }
    
    return match ($type) {
        Actions::Restore->value => call_user_func(function () use ($ids, $request): Responsable {
            $this->authorize('update', Product::class);
            $this->service->restore($ids, $request->user()?->id);
            return Responder::ok([
                'message' => __('Záznamy :ids byly obnoveny', ['ids' => implode(', ', $ids)]),
            ]);
        }),
        
        Actions::Delete->value => call_user_func(function () use ($ids): Responsable {
            $this->authorize('delete', Product::class);
            $this->service->batchDelete($ids);
            return Responder::ok([
                'message' => __('Záznamy :ids byly smazány', ['ids' => implode(', ', $ids)]),
                'ids' => $ids,
            ]);
        }),
        
        Actions::DeleteForce->value => call_user_func(function () use ($ids): Responsable {
            $this->authorize('delete', Product::class);
            $this->service->batchForceDelete($ids);
            return Responder::ok([
                'message' => __('Záznamy :ids byly trvale smazány', ['ids' => implode(', ', $ids)]),
                'ids' => $ids,
            ]);
        }),
        
        Actions::Export->value => call_user_func(function () use ($ids, $request): BinaryFileResponse {
            $this->authorize('viewAny', Product::class);
            // Export implementace (Excel/CSV)
            return Excel::download(new ProductsExport($ids), 'products.xlsx');
        }),
        
        default => (throw new BadRequest('Unknown type from action.'))
            ->convey(__('Neznámý typ akce'))
    };
}
```

## 8. {model}Actions - Akce na jednom objektu

Pro akce specifické pro jeden objekt (např. publikování, duplikace, změna stavu):

```php
use Domain\ProductActions;

public function productActions(
    Request $request,
    Product $product,
): Responsable|BinaryFileResponse
{
    $type = $request->input('type') ?? null;
    if (!$type) {
        throw new UnprocessableContent('Action type is required.')
            ->convey(__('Typ akce je povinný.'));
    }
    
    $locale = RequestHelper::getDataLocale($request);
    
    return match ($type) {
        ProductActions::Publish->value => call_user_func(function () use ($product, $request): Responsable {
            $this->authorize('update', $product);
            $this->service->publish($product, $request->user()?->id);
            return Responder::ok([
                'message' => __('Produkt :id byl publikován', ['id' => $product->id]),
                'id' => $product->id,
            ]);
        }),
        
        ProductActions::Unpublish->value => call_user_func(function () use ($product, $request): Responsable {
            $this->authorize('update', $product);
            $this->service->unpublish($product, $request->user()?->id);
            return Responder::ok([
                'message' => __('Produkt :id byl odpublikován', ['id' => $product->id]),
                'id' => $product->id,
            ]);
        }),
        
        ProductActions::Duplicate->value => call_user_func(function () use ($product, $request, $locale): Responsable {
            $this->authorize('create', Product::class);
            $newProduct = $this->service->duplicate(
                product: $product,
                locale: $locale,
                userId: $request->user()?->id
            );
            return Responder::created(
                new ProductResource($newProduct)
            );
        }),
        
        ProductActions::GenerateSku->value => call_user_func(function () use ($product, $request): Responsable {
            $this->authorize('update', $product);
            $this->service->generateSku($product, $request->user()?->id);
            return Responder::ok([
                'message' => __('SKU bylo vygenerováno'),
                'sku' => $product->fresh()->sku,
            ]);
        }),
        
        default => (throw new BadRequest('Unknown type from action.'))
            ->convey(__('Neznámý typ akce'))
    };
}
```

**Enum pro {model}Actions:**

```php
<?php

declare(strict_types=1);

namespace Domain;

enum ProductActions: string
{
    case Publish = 'publish';
    case Unpublish = 'unpublish';
    case Duplicate = 'duplicate';
    case GenerateSku = 'generate-sku';
    
    public static function apiMappings(): array
    {
        return [
            self::Publish->value => 'Publikovat',
            self::Unpublish->value => 'Odpublikovat',
            self::Duplicate->value => 'Duplikovat',
            self::GenerateSku->value => 'Vygenerovat SKU',
        ];
    }
}
```

## Přehled REST metod

| Metoda | HTTP | Parametry | Autorizace | Použití |
|--------|------|-----------|------------|---------|
| **index** | GET | Request | `viewAny` | Seznam záznamů |
| **show** | GET | Model/keyOrId | `view` | Detail záznamu |
| **store** | POST | Request, Validation | `create` | Vytvoření |
| **update** | PUT/PATCH | Request, Model, Validation | `update` | Aktualizace |
| **destroy** | DELETE | Request, Model | `delete` | Smazání |
| **restore** | POST | Request, Model | `update` | Obnovení |
| **actions** | POST | Request | různé | Hromadné operace (více objektů) |
| **{model}Actions** | POST | Request, Model | různé | Akce na jednom objektu |

## Responder metody

```php
// 200 OK
return Responder::ok($data);
return Responder::ok(new ProductResource($product));

// 201 Created
return Responder::created(new ProductResource($product));

// 204 No Content
return Responder::noContent();

// 400 Bad Request
throw new BadRequest('Error message')->convey(__('Chybová zpráva'));

// 404 Not Found
throw new NotFound('Not found')->convey(__('Nenalezeno'));

// 422 Unprocessable Content
throw new UnprocessableContent('Validation error')->convey(__('Validační chyba'));
```

**⚠️ Důležité:**
- **`$this->authorize()`** v KAŽDÉ metodě (kromě public bez autorizace)
- **`Responder::ok()` / `created()`** pro konzistentní odpovědi
- **`RequestHelper`** pro locale, trash, force
- **`FilterBuilder`** pro zpracování filtrů
- **`match($request->getMethod())`** pro PUT vs PATCH
- **`createOrReplace()` vs `update()`** - podle HTTP metody
- **`call_user_func()`** v actions pro lepší scope control
- **`__()` překladová funkce** pro všechny uživatelské zprávy
- **`actions()`** - hromadné operace na více objektech (ids array)
- **`{model}Actions()`** - akce na jednom objektu (Model parameter)

Reference: [Controller Structure](controller-structure.md), [Validation Structure](validation-structure.md), [Service Upsert](service-upsert.md)
