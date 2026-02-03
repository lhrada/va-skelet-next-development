---
title: Policy struktura a autorizace
impact: HIGH
impactDescription: Policy zajišťují autorizaci akcí nad modely
tags: policy, authorization, permissions, gate
---

## Policy struktura a autorizace

**Impact: HIGH**

Policy třídy slouží k autorizaci akcí nad modely. Kontrolují oprávnění z `Permission` enumu.

**Namespace:** `App\Policies`  
**Soubory:** `app/Policies/{Model}Policy.php`

## Základní struktura

```php
<?php

declare(strict_types=1);

namespace App\Policies;

use App\Models\Product;
use App\Models\User;
use Dakujem\Strata\Http\Forbidden;
use Frame\Permissions\Permission;
use Illuminate\Auth\Access\HandlesAuthorization;

final class ProductPolicy
{
    use HandlesAuthorization;

    public function before(User $user, string $ability): bool|null
    {
        return null;
    }

    public function viewAny(?User $user): bool
    {
        if ($user && $user->can(Permission::ProductViewAny->value)) {
            return true;
        }
        
        throw new Forbidden('Product: Forbidden')
            ->convey(__('Nemáte oprávnění k zobrazení seznamu'));
    }

    public function view(?User $user, Product $product): bool
    {
        if ($user && $user->can(Permission::ProductView->value)) {
            return true;
        }
        
        throw new Forbidden('Product: Forbidden')
            ->convey(__('Nemáte oprávnění k zobrazení detailu'));
    }

    public function create(?User $user): bool
    {
        if ($user && $user->can(Permission::ProductCreate->value)) {
            return true;
        }
        
        throw new Forbidden('Product: Forbidden')
            ->convey(__('Nemáte oprávnění k vytvoření záznamu'));
    }

    public function update(?User $user, Product $product): bool
    {
        if ($user && $user->can(Permission::ProductUpdateAny->value)) {
            return true;
        }
        
        throw new Forbidden('Product: Forbidden')
            ->convey(__('Nemáte oprávnění k aktualizaci záznamu'));
    }

    public function delete(?User $user, Product $product): bool
    {
        if ($user && $user->can(Permission::ProductDelete->value)) {
            return true;
        }
        
        throw new Forbidden('Product: Forbidden')
            ->convey(__('Nemáte oprávnění ke smazání záznamu'));
    }
}
```

## Standardní metody Policy

| Metoda | Parametry | Oprávnění | Použití |
|--------|-----------|-----------|---------|
| `viewAny` | `?User` | `{Model}ViewAny` | Seznam záznamů (index) |
| `view` | `?User`, `Model` | `{Model}View` | Detail záznamu (show) |
| `create` | `?User` | `{Model}Create` | Vytvoření (store) |
| `update` | `?User`, `Model` | `{Model}UpdateAny` | Aktualizace (update) |
| `delete` | `?User`, `Model` | `{Model}Delete` | Smazání (destroy) |

## ViewAny vs ViewOwn

Pro podporu "vlastních" záznamů:

```php
public function viewAny(?User $user): bool
{
    if ($user && $user->can(Permission::ArticleViewAny->value)) {
        return true;
    }
    
    // Pokud může vidět vlastní, povol viewAny (filtrování bude v service)
    if ($user && $user->can(Permission::ArticleViewOwn->value)) {
        return true;
    }
    
    throw new Forbidden('Article: Forbidden')
        ->convey(__('Nemáte oprávnění k zobrazení seznamu'));
}

public function view(?User $user, Article $article): bool
{
    if ($user && $user->can(Permission::ArticleViewAny->value)) {
        return true;
    }
    
    // Kontrola vlastnictví
    if ($user && $user->can(Permission::ArticleViewOwn->value) && $user->id === $article->created_by) {
        return true;
    }
    
    throw new Forbidden('Article: Forbidden')
        ->convey(__('Nemáte oprávnění k zobrazení detailu'));
}
```

## UpdateAny vs UpdateOwn

```php
public function update(?User $user, Product $product): bool
{
    if ($user && $user->can(Permission::ProductUpdateAny->value)) {
        return true;
    }
    
    // Vlastní záznamy
    if ($user && $user->can(Permission::ProductUpdateOwn->value) && $user->id === $product->created_by) {
        return true;
    }
    
    throw new Forbidden('Product: Forbidden')
        ->convey(__('Nemáte oprávnění k aktualizaci záznamu'));
}
```

## before() metoda

Pro globální kontrolu před každou autorizací:

```php
public function before(User $user, string $ability): bool|null
{
    // Root má přístup ke všemu
    if ($user->can(Permission::Root->value)) {
        return true;
    }
    
    // Null = pokračovat normální kontrolou
    return null;
}
```

## Použití v Controlleru

```php
public function index(Request $request): Responsable
{
    $this->authorize('viewAny', Product::class);
    
    // ... implementace
}

public function show(Request $request, Product $product): Responsable
{
    $this->authorize('view', $product);
    
    // ... implementace
}

public function store(Request $request, ProductValidation $validation): Responsable
{
    $this->authorize('create', Product::class);
    
    // ... implementace
}

public function update(Request $request, Product $product, ProductValidation $validation): Responsable
{
    $this->authorize('update', $product);
    
    // ... implementace
}

public function destroy(Request $request, Product $product): Responsable
{
    $this->authorize('delete', $product);
    
    // ... implementace
}
```

## Registrace Policy

Policy se automaticky načítají podle konvence, ale můžeš je explicitně zaregistrovat v `AuthServiceProvider`:

```php
protected $policies = [
    Product::class => ProductPolicy::class,
    Article::class => ArticlePolicy::class,
];
```

## Vlastní metody Policy

Pro specifické akce:

```php
public function publish(?User $user, Article $article): bool
{
    if ($user && $user->can(Permission::ArticlePublish->value)) {
        return true;
    }
    
    throw new Forbidden('Article: Forbidden')
        ->convey(__('Nemáte oprávnění k publikování článku'));
}

public function duplicate(?User $user, Product $product): bool
{
    if ($user && $user->can(Permission::ProductDuplicate->value)) {
        return true;
    }
    
    throw new Forbidden('Product: Forbidden')
        ->convey(__('Nemáte oprávnění k duplikaci produktu'));
}
```

**Použití v controlleru:**

```php
public function productActions(Request $request, Product $product): Responsable
{
    $type = $request->input('type');
    
    return match ($type) {
        ProductActions::Publish->value => call_user_func(function () use ($product): Responsable {
            $this->authorize('publish', $product);
            // ... implementace
        }),
        
        ProductActions::Duplicate->value => call_user_func(function () use ($product): Responsable {
            $this->authorize('duplicate', $product);
            // ... implementace
        }),
    };
}
```

## Dakujem\Strata\Http\Forbidden

Pro konzistentní chybové zprávy používej `Forbidden` exception:

```php
use Dakujem\Strata\Http\Forbidden;

throw new Forbidden('Product: Forbidden')
    ->convey(__('Nemáte oprávnění k aktualizaci záznamu'));
```

**Výhody:**
- Automatický HTTP 403 response
- `convey()` nastaví uživatelsky přívětivou zprávu
- První parametr je technická zpráva (pro logy)

## Testování Policy

```php
use Tests\TestCase;

class ProductPolicyTest extends TestCase
{
    public function test_admin_can_view_any_products(): void
    {
        $admin = User::factory()->create();
        $admin->givePermissionTo(Permission::ProductViewAny);
        
        $this->assertTrue($admin->can('viewAny', Product::class));
    }
    
    public function test_user_without_permission_cannot_view_products(): void
    {
        $user = User::factory()->create();
        
        $this->assertFalse($user->can('viewAny', Product::class));
    }
    
    public function test_user_can_view_own_product(): void
    {
        $user = User::factory()->create();
        $user->givePermissionTo(Permission::ProductViewOwn);
        
        $product = Product::factory()->create(['created_by' => $user->id]);
        
        $this->assertTrue($user->can('view', $product));
    }
}
```

**⚠️ Důležité:**
- **`final class`** - policy nemá potomky
- **`?User`** parameter - může být null (veřejné API)
- **`Forbidden` exception** s `convey()` pro uživatelskou zprávu
- **`Permission` enum** pro názvy oprávnění
- **`before()`** pro globální kontroly (Root)
- **ViewOwn/UpdateOwn** kontrolují `created_by`
- **`$this->authorize()`** v KAŽDÉ controller metodě

**📘 Viz také:**
- **Permission enum**: `frame/Permissions/Permission.php`
- **[Permission Enum Naming](permissions-enum-naming.md)** - Pravidla pro názvy oprávnění
- **[Permission API Mappings](permissions-api-mappings.md)** - Lidsky čitelné názvy
- **[Permission Artisan Command](permissions-artisan-command.md)** - Vytvoření v DB

Reference: [Controller Structure](controller-structure.md), [Permission Enum Naming](permissions-enum-naming.md)