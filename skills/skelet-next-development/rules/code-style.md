---
title: Code Style pravidla
impact: HIGH
impactDescription: Code style zajišťuje konzistentní a maintainovatelný kód
tags: php, code-style, strict-types, readonly, final, match, phpdoc
---

## Code Style pravidla

**Impact: HIGH**

PHP 8.4+ best practices pro Skelet Next projekty.

## Strict Types

**POVINNÉ** na začátku každého PHP souboru:

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Admin\Products;
```

**Proč:**
- Kontrola typů za běhu
- Prevence type coercion chyb
- Lepší IDE podpora

**Kdy přidat:**
- Při vytvoření nového souboru
- Když vidíš soubor bez `declare(strict_types=1);`

## Return Types

**VŽDY** explicitní return types pro metody a funkce:

```php
// ✅ SPRÁVNĚ
public function getProducts(): Collection
{
    return Product::all();
}

public function calculatePrice(int $amount): float
{
    return $amount * 1.21;
}

public function findById(int $id): ?Product
{
    return Product::find($id);
}

// ❌ ŠPATNĚ
public function getProducts()
{
    return Product::all();
}
```

**Povolené return types:**
- Skalární: `int`, `float`, `string`, `bool`
- Třídy: `Product`, `Collection`, `Response`
- Pole: `array`
- Nullable: `?Product`, `?string`
- Union: `Product|null`, `int|float`
- Void: `void` (nic nevrací)
- Mixed: `mixed` (cokoliv) - **avoid pokud možno**

## Type Hints pro parametry

**VŽDY** type hints pro všechny parametry:

```php
// ✅ SPRÁVNĚ
public function update(Product $product, array $data): bool
{
    return $product->update($data);
}

public function calculate(int $amount, float $rate, ?string $currency = null): float
{
    return $amount * $rate;
}

// ❌ ŠPATNĚ
public function update($product, $data)
{
    return $product->update($data);
}
```

## Readonly Properties

**Použij `readonly`** pro immutable data:

```php
// ✅ SPRÁVNĚ
final class ProductService
{
    public function __construct(
        private readonly ProductRepository $repository,
        private readonly CacheService $cache,
    ) {}
}

// ✅ SPRÁVNĚ - readonly property
final class Product extends Model
{
    public function __construct(
        public readonly string $guid,
    ) {
        parent::__construct();
    }
}

// ❌ ŠPATNĚ - property se nemění, měla by být readonly
final class ProductService
{
    private ProductRepository $repository;
    
    public function __construct(ProductRepository $repository)
    {
        $this->repository = $repository;
    }
}
```

**Kdy použít:**
- Constructor dependencies (services, repositories)
- Properties které se nenastaví po vytvoření
- Immutable value objects

**Kdy NEPOUŽÍT:**
- Properties které se mohou změnit
- Eloquent model attributes (kromě custom readonly properties)

## Final Classes

**Použij `final`** pokud třída nemá potomky:

```php
// ✅ SPRÁVNĚ
final class ProductController extends Controller
{
    // ...
}

final class ProductService
{
    // ...
}

final class ProductResource extends JsonResource
{
    // ...
}

// ❌ ŠPATNĚ - třída není final a nemá potomky
class ProductController extends Controller
{
    // ...
}
```

**Kdy použít:**
- Všechny Controllers (99% případů)
- Services
- Resources
- Validation třídy
- Policy třídy

**Kdy NEPOUŽÍT:**
- Base třídy určené k extendování
- Třídy v `frame/` které jsou base pro jiné

## Match Expression

**Preferuj `match`** místo `switch`:

```php
// ✅ SPRÁVNĚ - match
$status = match ($order->state) {
    OrderState::Pending => 'Čeká na zpracování',
    OrderState::Processing => 'Zpracovává se',
    OrderState::Completed => 'Dokončeno',
    OrderState::Cancelled => 'Zrušeno',
    default => 'Neznámý stav',
};

// ✅ SPRÁVNĚ - match s return
return match ($product->type) {
    ProductType::Physical => $this->calculateShipping($product),
    ProductType::Digital => 0,
    ProductType::Service => $this->calculateServiceFee($product),
};

// ❌ ŠPATNĚ - switch
switch ($order->state) {
    case OrderState::Pending:
        $status = 'Čeká na zpracování';
        break;
    case OrderState::Processing:
        $status = 'Zpracovává se';
        break;
    // ...
}
```

**Výhody match:**
- Vrací hodnotu (expression, ne statement)
- Strict comparison (===)
- Exhaustive check (musí pokrýt všechny případy)
- Kratší syntax

**Kdy použít switch:**
- Složitá logika v každé větvi
- Side effects (logování, volání metod)
- Fallthrough behavior

## Nullsafe Operator

**Použij `?->` pro nullable chains:**

```php
// ✅ SPRÁVNĚ - nullsafe
$city = $user?->address?->city;
$total = $order?->invoice?->total ?? 0;

// ❌ ŠPATNĚ - verbose
$city = null;
if ($user !== null && $user->address !== null) {
    $city = $user->address->city;
}

// ✅ SPRÁVNĚ - kombinace s null coalescing
$name = $product?->category?->name ?? 'Bez kategorie';
```

## Null Coalescing Operator

**Použij `??` pro default hodnoty:**

```php
// ✅ SPRÁVNĚ
$limit = $request->input('limit') ?? 20;
$name = $data['name'] ?? 'Bez názvu';
$config = $this->config ?? $this->getDefaultConfig();

// ❌ ŠPATNĚ
$limit = isset($request->input('limit')) ? $request->input('limit') : 20;
```

## Enums

**Použij enum** pro fixed sady hodnot:

```php
// ✅ SPRÁVNĚ - enum
enum ProductType: string
{
    case Physical = 'physical';
    case Digital = 'digital';
    case Service = 'service';
    
    public function label(): string
    {
        return match ($this) {
            self::Physical => 'Fyzický produkt',
            self::Digital => 'Digitální produkt',
            self::Service => 'Služba',
        };
    }
}

// Použití
$type = ProductType::Physical;
$label = $type->label();

// ❌ ŠPATNĚ - konstanty
class ProductType
{
    public const PHYSICAL = 'physical';
    public const DIGITAL = 'digital';
    public const SERVICE = 'service';
}
```

**Kdy použít:**
- Stavy (OrderState, ProductState)
- Typy (ProductType, DocumentType)
- Oprávnění (Permission enum)
- Fixed hodnoty bez logiky

## PHPDoc

**Aktualizuj PHPDoc** při změně properties:

```php
/**
 * @property int $id
 * @property string $guid
 * @property string $title
 * @property string|null $perex
 * @property bool $published
 * @property Carbon $created_at
 * @property Carbon $updated_at
 * 
 * @property-read Category|null $category
 * @property-read Collection<Tag> $tags
 */
final class Product extends Model
{
    // ...
}
```

**Co dokumentovat:**
- `@property` - databázové sloupce
- `@property-read` - computed properties a relationships
- `@method` - dynamic methods (scopes, macros)
- `@param` a `@return` - jen pokud nejsou jasné z type hints

**Kdy aktualizovat:**
- Přidání nového sloupce (migrace)
- Přidání nové relation
- Změna typu sloupce

## Constructor Property Promotion

**Použij PHP 8 constructor promotion:**

```php
// ✅ SPRÁVNĚ - promotion
final class ProductService
{
    public function __construct(
        private readonly ProductRepository $repository,
        private readonly CacheService $cache,
        private readonly LoggerInterface $logger,
    ) {}
}

// ❌ ŠPATNĚ - old style
final class ProductService
{
    private ProductRepository $repository;
    private CacheService $cache;
    private LoggerInterface $logger;
    
    public function __construct(
        ProductRepository $repository,
        CacheService $cache,
        LoggerInterface $logger
    ) {
        $this->repository = $repository;
        $this->cache = $cache;
        $this->logger = $logger;
    }
}
```

## Prázdné constructory

**NEPOVOL** prázdné constructory:

```php
// ❌ ŠPATNĚ - zbytečný constructor
final class ProductController extends Controller
{
    public function __construct()
    {
        //
    }
}

// ✅ SPRÁVNĚ - odstraň constructor
final class ProductController extends Controller
{
    // Žádný constructor
}

// ✅ SPRÁVNĚ - constructor s parametry
final class ProductController extends Controller
{
    public function __construct(
        private readonly ProductService $service,
    ) {
        $this->middleware('auth:sanctum');
    }
}
```

**Výjimka:** Private constructor pro singleton pattern.

## Array Shapes v PHPDoc

**Dokumentuj array struktury:**

```php
/**
 * @param array{
 *     title: string,
 *     price: float,
 *     published: bool,
 *     tags?: array<int>
 * } $data
 */
public function create(array $data): Product
{
    return Product::create($data);
}

/**
 * @return array{
 *     total: int,
 *     items: Collection<Product>,
 *     hasMore: bool
 * }
 */
public function paginate(int $page): array
{
    // ...
}
```

## Checklist pro nový soubor

- [ ] `declare(strict_types=1);` na začátku
- [ ] `final` class (pokud není base)
- [ ] Explicitní return types pro všechny metody
- [ ] Type hints pro všechny parametry
- [ ] `readonly` pro immutable properties
- [ ] Constructor property promotion
- [ ] PHPDoc pro properties a complex arrays
- [ ] `match` místo `switch` (kde to dává smysl)
- [ ] Nullsafe operator `?->` pro nullable chains
- [ ] Null coalescing `??` pro defaults
- [ ] Enum místo konstant (kde to dává smysl)

## Příklad kompletní třídy

```php
<?php

declare(strict_types=1);

namespace Domain\Products;

use App\Models\Product;
use Illuminate\Support\Collection;

/**
 * Service pro práci s produkty
 */
final class ProductService
{
    public function __construct(
        private readonly ProductRepository $repository,
        private readonly CacheService $cache,
    ) {}
    
    /**
     * @return Collection<Product>
     */
    public function getPublished(): Collection
    {
        return $this->cache->remember(
            'products.published',
            3600,
            fn () => $this->repository->getPublished()
        );
    }
    
    public function findBySlug(string $slug): ?Product
    {
        return $this->repository->findBySlug($slug);
    }
    
    /**
     * @param array{
     *     title: string,
     *     price: float,
     *     published?: bool
     * } $data
     */
    public function create(array $data): Product
    {
        $data['published'] ??= false;
        
        return $this->repository->create($data);
    }
    
    public function calculatePrice(Product $product, ?string $currency = null): float
    {
        $price = $product->price;
        
        return match ($currency) {
            'EUR' => $price * 0.04,
            'USD' => $price * 0.043,
            default => $price,
        };
    }
}
```

**⚠️ Důležité:**
- **declare(strict_types=1)** - VŽDY první řádek
- **final class** - pokud nemá potomky
- **readonly** - pro dependencies a immutable data
- **Return types** - explicitní pro všechny metody
- **Type hints** - pro všechny parametry
- **match** - preferuj místo switch
- **PHPDoc** - pro properties a complex struktury

Reference: [Controller Structure](controller-structure.md), [Validation Structure](validation-structure.md), [Tools Artisan Scaffolding](tools-artisan-scaffolding.md)