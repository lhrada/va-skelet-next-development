---
title: Service vrstva - struktura a best practices
impact: HIGH
impactDescription: Service vrstva odděluje business logiku od controllerů
tags: service, domain, business-logic, readonly
---

## Service vrstva - struktura a best practices

**Impact: HIGH**

Service vrstva v `Domain\{Entity}\{Entity}Service` obsahuje veškerou business logiku. Používej `readonly` class a dependency injection.

**Incorrect:**

```php
<?php

namespace Domain\Products;

use App\Models\Product;

// CHYBÍ declare(strict_types=1)
// CHYBÍ readonly
class ProductService
{
    // CHYBÍ dependency injection
    public function create(array $data): Product
    {
        // Business logika přímo v metodě - špatně
        $product = new Product();
        $product->name = $data['name'];
        $product->price = $data['price'];
        $product->guid = \Str::uuid()->toString(); // ŠPATNĚ - použij Uuid::create()
        $product->save();
        
        return $product;
    }
}
```

**Correct:**

```php
<?php

declare(strict_types=1);

namespace Domain\Products;

use App\Models\Product;
use Blox\Utils\Uuid;
use Domain\Conventions\ModelHelper;
use Illuminate\Support\Facades\DB;

final readonly class ProductService
{
    public function __construct(
        private Product $product,
    ) {
    }

    /**
     * Upsert - vytvoření nebo aktualizace
     */
    public function upsert(
        string $locale,
        ?Product $product,
        array $data,
    ): Product {
        return DB::transaction(function () use ($locale, $product, $data) {
            if ($product === null) {
                $product = $this->product->newInstance();
                $product->guid = Uuid::create();  // ✓ Správně
                $product->created_by = $data['created_by'] ?? null;
            } else {
                $product->updated_by = $data['updated_by'] ?? null;
            }

            // Hydratace pomocí ModelHelper
            $properties = [
                'name',
                'description',
                'price',
                'stock',
                'sequence',
                'priority',
                'protected',
                'published',
            ];

            ModelHelper::hydrateModel($product, $data, $properties);

            $product->save();

            return $product->refresh();
        });
    }

    public function delete(Product $product): void
    {
        // VŽDY kontroluj protected flag
        ModelHelper::preventDelete($product);
        $product->delete();
    }

    public function forceDelete(Product $product): void
    {
        ModelHelper::preventDelete($product);
        $product->forceDelete();
    }
}
```

**Povinné prvky Service:**

1. **`declare(strict_types=1);`** na začátku
2. **`final readonly class`** - immutable service
3. **Constructor injection** - dependency injection modelů
4. **DB::transaction()** pro upsert operace
5. **`Uuid::create()`** místo `Str::uuid()->toString()`
6. **`ModelHelper::hydrateModel()`** pro nastavení properties
7. **`ModelHelper::preventDelete()`** před mazáním

**Struktura metod:**

```php
final readonly class ProductService
{
    public function __construct(
        private Product $product,
        private ProductTranslation $productTranslation,
    ) {
    }

    // Vytvoření/aktualizace
    public function upsert(string $locale, ?Product $product, array $data): Product
    
    // Pro admin - seznam s filtry
    public function staffSource(string $locale, QueryObject $queryObject, bool $trash = false): Paginator
    
    // Builder pro admin
    public function staffBuilder(QueryObject $queryObject, bool $trash, string $locale): Builder
    
    // Pro klienta - veřejná část
    public function clientSource(string $locale, QueryObject $queryObject): Paginator
    
    // Builder pro klienta
    public function clientBuilder(QueryObject $queryObject, string $locale): Builder
    
    // Nalezení podle ID nebo key
    public function getByKeyOrId(int|string $id, bool $trash = false, array $withRelations = []): Product
    
    // Mazání
    public function delete(Product $product): void
    public function forceDelete(Product $product): void
    public function batchDelete(array $ids): void
    public function restore(array $ids, ?int $userId): void
}
```

**Konvence pojmenování:**

| Metoda | Účel | Kde použít |
|--------|------|------------|
| `staff*` | Pro administraci | Admin controllery |
| `client*` | Pro klienty/veřejnost | Public controllery |
| `upsert()` | Create/Update | Univerzální vytvoření/aktualizace |
| `getByKeyOrId()` | Find by ID or key | Detail záznamu |
| `delete()` | Soft delete | Běžné mazání |
| `forceDelete()` | Hard delete | Permanentní mazání |
| `batchDelete()` | Multiple delete | Hromadné mazání |
| `restore()` | Undelete | Obnovení smazaných |

**⚠️ Důležité:**
- Service je **readonly** - immutable
- Používej **DB::transaction()** pro operace s databází
- **ModelHelper::hydrateModel()** pro nastavení properties
- **Uuid::create()** místo Str::uuid()
- **ModelHelper::preventDelete()** kontroluje protected flag

Reference: [Service Builder](service-builder.md), [Service Upsert](service-upsert.md)