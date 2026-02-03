---
title: Struktura Controlleru s Namespace konvencemi
impact: CRITICAL
impactDescription: Správné umístění a pojmenování je zásadní pro fungování projektu
tags: architecture, controller, namespace
---

## Struktura Controlleru s Namespace konvencemi

**Impact: CRITICAL**

Skelet Next používá **vlastní namespace strukturu odlišnou od standardního Laravelu**. Controllery jsou organizovány podle typu (Admin/Public) a modulu.

**Namespace konvence:**

| Typ | Namespace | Příklad |
|-----|-----------|---------|
| Admin Controller | `App\Http\Controllers\Admin\{Modul}\{Entity}Controller` | `Admin\Products\ProductController` |
| Public Controller | `App\Http\Controllers\Public\{Modul}\{Entity}Controller` | `Public\Products\ProductController` |
| Společný Controller | `App\Http\Controllers\{Entity}Controller` | `AuthController` |

**Incorrect (standardní Laravel struktura):**

```php
<?php
// app/Http/Controllers/ProductController.php
namespace App\Http\Controllers;

class ProductController extends Controller
{
    // Špatně - chybí Admin/Public/modul rozlišení
}
```

**Correct (Skelet Next struktura):**

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Admin\Products;

use App\Http\Controllers\Controller;
use App\Models\Product;
use Domain\Products\ProductService;
use Illuminate\Http\Request;

final class ProductController extends Controller
{
    public function __construct(
        private readonly ProductService $service,
    ) {
        $this->middleware('auth:sanctum');
    }
    
    // REST metody...
}
```

**Důležité:**
- **Vždy** použij `declare(strict_types=1);`
- **Vždy** použij `final` pokud třída nemá subclass
- Constructor property promotion pro dependencies
- Middleware registrace v `__construct()`

Reference: [Router Instructions](../../instructions/router.instructions.md)
