---
title: Controller struktura a základy
impact: HIGH
impactDescription: Správná struktura controlleru zajišťuje konzistentní API
tags: controller, architecture, rest, authorization
---

## Controller struktura a základy

**Impact: HIGH**

Controllery v projektu mají specifickou strukturu s `readonly` service injection a `auth:sanctum` middleware.

**Namespace struktura:**
- **Admin**: `App\Http\Controllers\Admin\{Modul}\{Název}Controller`
- **Public**: `App\Http\Controllers\Public\{Modul}\{Název}Controller`
- **Společné**: `App\Http\Controllers\{Název}Controller` (mimo Admin/Public)

**Incorrect:**

```php
<?php

namespace App\Http\Controllers\Admin\Products;

use App\Http\Controllers\Controller;

// CHYBÍ declare(strict_types=1)
// CHYBÍ final
// CHYBÍ readonly na service
class ProductController extends Controller
{
    // CHYBÍ dependency injection
    public function index()  // CHYBÍ return type
    {
        // Business logika přímo v controlleru - ŠPATNĚ
        $products = Product::all();
        return response()->json($products);
    }
}
```

**Correct:**

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Admin\Products;

use App\Http\Controllers\Controller;
use App\Http\RequestHelper;
use App\Models\Product;
use Domain\Products\ProductService;
use Frame\Controllers\FilterBuilder;
use Frame\Http\Responder;
use Illuminate\Contracts\Support\Responsable;
use Illuminate\Http\Request;

final class ProductController extends Controller
{
    public function __construct(
        private readonly ProductService $service,
    ) {
        // Auth middleware pro admin controllery
        $this->middleware(
            middleware: 'auth:sanctum',
            options: ['except' => []]  // nebo konkrétní metody bez auth
        );
    }

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
}
```

**Povinné prvky:**

1. **`declare(strict_types=1);`** na začátku
2. **`final class`** - controller nemá subclass
3. **`readonly`** na service v constructoru
4. **`auth:sanctum` middleware** pro admin controllery
5. **`$this->authorize()`** v každé metodě
6. **Return type `Responsable`** na všech public metodách
7. **FilterBuilder** pro zpracování query parametrů
8. **RequestHelper** pro locale a trash

## Admin vs Public controllery

### Admin Controller

```php
final class ProductController extends Controller
{
    public function __construct(
        private readonly ProductService $service,
    ) {
        // VŽDY auth:sanctum pro admin
        $this->middleware('auth:sanctum');
    }

    public function index(Request $request): Responsable
    {
        $this->authorize('viewAny', Product::class);
        
        // staffSource pro admin
        return ProductListResource::collection(
            $this->service->staffSource(
                locale: RequestHelper::getDataLocale($request),
                queryObject: (new FilterBuilder())
                    ->only(...$this->getStaffOnly())
                    ->makeQueryObject($request),
                trash: RequestHelper::showTrash($request),
            )
        );
    }

    private function getStaffOnly(): array
    {
        return ['id', 'key', 'name', 'published', 'deleted'];
    }
}
```

### Public Controller

```php
final class ProductController extends Controller
{
    public function __construct(
        private readonly ProductService $service,
    ) {
        // auth.optional:sanctum pro public (volitelná autentizace)
        $this->middleware('auth.optional:sanctum');
        
        // tenant middleware pro multi-tenancy
        $this->middleware('tenant');
    }

    public function index(Request $request): Responsable
    {
        // Bez authorize - public endpoint
        
        // clientSource pro public
        return ProductListResource::collection(
            $this->service->clientSource(
                locale: RequestHelper::getDataLocale($request),
                queryObject: (new FilterBuilder())
                    ->only(...$this->getClientOnly())
                    ->makeQueryObject($request),
            )
        );
    }

    private function getClientOnly(): array
    {
        return ['id', 'name', 'price'];  // Méně filtrů než admin
    }
}
```

## Middleware podle typu

| Typ | Middleware | Použití |
|-----|-----------|---------|
| **Admin** | `auth:sanctum` | Vyžaduje přihlášení |
| **Public** | `auth.optional:sanctum` | Volitelné přihlášení |
| **Public** | `tenant` | Multi-tenancy |
| **Všechny** | `vaTenant` | Via Aurea tenant |

## FilterBuilder a QueryObject

```php
$only = ['id', 'name', 'published'];

$qo = new FilterBuilder()
    ->only(...$only)           // Povolené filtry
    ->makeQueryObject($request);

// QueryObject se předává do service
$this->service->staffSource(
    locale: $dataLocale,
    queryObject: $qo,
    trash: $trash,
);
```

**⚠️ Důležité:**
- **`final class`** - controller nemá potomky
- **`readonly`** service - immutable dependency
- **`auth:sanctum`** pro admin, **`auth.optional:sanctum`** pro public
- **`$this->authorize()`** v každé metodě (kromě public bez autorizace)
- **`Responsable` return type** - jednotné API odpovědi
- **FilterBuilder** - zpracování query parametrů
- **`staffSource()` vs `clientSource()`** - podle typu controlleru
- **`getStaffOnly()` vs `getClientOnly()`** - různé filtry pro admin/public

Reference: [Controller REST Methods](controller-rest-methods.md), [Service Builder](service-builder.md)