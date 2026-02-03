---
title: Permission enum - pojmenování a hodnoty
impact: HIGH
impactDescription: Správné pojmenování oprávnění zajišťuje konzistenci
tags: permissions, enum, naming, conventions
---

## Permission enum - pojmenování a hodnoty

**Impact: HIGH**

Permission enum definuje všechna oprávnění v systému. Musí být konzistentně pojmenované a mít správné hodnoty.

**Soubor:** `frame/Permissions/Permission.php`  
**Enum:** `Frame\Permissions\Permission`

## Konvence pro case názvy

**PascalCase** formát: `{Entity}{Action}` nebo `{Entity}{Action}{Scope}`

### Vzory akcí

| Akce | Použití | Příklad |
|------|---------|---------|
| `Create` | Vytvoření | `ArticleCreate` |
| `ViewAny` | Čtení všech | `ArticleViewAny` |
| `ViewOwn` | Čtení vlastních | `ArticleViewOwn` |
| `UpdateAny` | Úprava všech | `ArticleUpdateAny` |
| `UpdateOwn` | Úprava vlastních | `ArticleUpdateOwn` |
| `DeleteAny` | Smazání všech | `ArticleDeleteAny` |
| `DeleteOwn` | Smazání vlastních | `ArticleDeleteOwn` |

### Specifické akce

| Akce | Použití | Příklad |
|------|---------|---------|
| `Approve` | Schválení | `InventoryApproveAny` |
| `Reject` | Zamítnutí | `OrderReject` |
| `Cancel` | Zrušení | `InventoryCancelAny` |
| `Confirm` | Potvrzení | `OrderConfirm` |
| `Merge` | Sloučení | `ProfileMerge` |
| `Activate` | Aktivace | `CouponActivate` |
| `Deactivate` | Deaktivace | `CouponDeactivate` |
| `Publish` | Publikování | `ArticlePublish` |
| `Unpublish` | Odpublikování | `ArticleUnpublish` |

## Konvence pro hodnoty

**camelCase pro entitu + snake_case s tečkami**

Formát: `{entity}.{action}` nebo `{entity}.{action}.{scope}`

**Maximální délka: 125 znaků**

### Příklady CRUD operací

```php
// Základní CRUD
case ArticleCreate = 'article.create';
case ArticleViewAny = 'article.view.any';
case ArticleViewOwn = 'article.view.own';
case ArticleUpdateAny = 'article.update.any';
case ArticleUpdateOwn = 'article.update.own';
case ArticleDeleteAny = 'article.delete.any';
case ArticleDeleteOwn = 'article.delete.own';

// Produkty
case ProductCreate = 'product.create';
case ProductViewAny = 'product.view.any';
case ProductUpdateAny = 'product.update.any';
case ProductDelete = 'product.delete';
```

### Příklady specifických akcí

```php
// Inventura
case InventoryApproveAny = 'inventory.approve.any';
case InventoryCancelAny = 'inventory.cancel.any';

// Profily
case ProfileMerge = 'person.merge';

// Kupony
case CouponActivate = 'coupon.activate';
case CouponDeactivate = 'coupon.deactivate';

// Články
case ArticlePublish = 'article.publish';
case ArticleUnpublish = 'article.unpublish';
```

### Kanálová oprávnění

Pro specifické kanály:

```php
case MenuChannelLvovCreate = 'menu.channel.lvov.create';
case MenuChannelLvovViewAny = 'menu.channel.lvov.view.any';
case MenuChannelPragueCreate = 'menu.channel.prague.create';
```

### Speciální oprávnění

```php
case Root = 'root';
case Routine = 'routine';
case Printer = 'printer';
case SessionViaCard = 'session.viaCard';
case Kiosk = 'kiosk';
case Erp = 'erp';
case Economics = 'economics';
case Logistic = 'logistic';
```

## Struktura v souboru

```php
<?php

declare(strict_types=1);

namespace Frame\Permissions;

enum Permission: string
{
    // Speciální oprávnění
    case Root = 'root';
    case Routine = 'routine';
    
    // Články
    case ArticleCreate = 'article.create';
    case ArticleViewAny = 'article.view.any';
    case ArticleViewOwn = 'article.view.own';
    case ArticleUpdateAny = 'article.update.any';
    case ArticleUpdateOwn = 'article.update.own';
    case ArticleDeleteAny = 'article.delete.any';
    case ArticleDeleteOwn = 'article.delete.own';
    
    // Produkty
    case ProductCreate = 'product.create';
    case ProductViewAny = 'product.view.any';
    case ProductUpdateAny = 'product.update.any';
    case ProductDelete = 'product.delete';
    
    // ... další oprávnění
    
    // Metody enum
    public static function apiMappings(): array { /* ... */ }
    public static function descriptions(): array { /* ... */ }
    public static function getGroups(): array { /* ... */ }
    public function pair(): array { /* ... */ }
}
```

## Pravidla pojmenování

### ✅ Správně

```php
// PascalCase pro case název
case ArticleCreate = 'article.create';
case ProductViewAny = 'product.view.any';
case ProfileMerge = 'person.merge';

// camelCase entita + snake_case akce
case MenuChannelLvovCreate = 'menu.channel.lvov.create';
case CarrierInHouseAcceptMoney = 'carrier.inHouse.acceptMoney';
```

### ❌ Špatně

```php
// Špatný formát case názvu (není PascalCase)
case article_create = 'article.create';
case ARTICLE_CREATE = 'article.create';

// Špatný formát hodnoty (není camelCase + snake_case)
case ArticleCreate = 'Article.Create';
case ArticleCreate = 'article-create';
case ArticleCreate = 'ARTICLE.CREATE';
```

## Přidání nového oprávnění

### Krok 1: Přidat case do enumu

```php
// V frame/Permissions/Permission.php
case LandmarkCreate = 'landmark.create';
case LandmarkViewAny = 'landmark.view.any';
case LandmarkViewOwn = 'landmark.view.own';
case LandmarkUpdateAny = 'landmark.update.any';
case LandmarkUpdateOwn = 'landmark.update.own';
case LandmarkDelete = 'landmark.delete';
```

### Krok 2: Přidat do apiMappings()

Viz [Permission API Mappings](permissions-api-mappings.md)

### Krok 3: Přidat do descriptions()

Viz [Permission Descriptions](permissions-descriptions.md)

### Krok 4: Přidat skupinu do getGroups()

Viz [Permission Groups](permissions-groups.md)

### Krok 5: Spustit Artisan command

```bash
docker compose run php php artisan app:create-permissions
```

Viz [Permission Artisan Command](permissions-artisan-command.md)

## Prefix a jejich význam

| Prefix | Entita | Příklad |
|--------|--------|---------|
| `article` | Články | `article.create` |
| `product` | Produkty | `product.view.any` |
| `order` | Objednávky | `order.update.any` |
| `profile` | Profily/Osoby | `person.merge` |
| `user` | Uživatelé | `user.create` |
| `branch` | Pobočky | `branch.view.any` |
| `warehouse` | Sklady | `warehouse.movement` |
| `inventory` | Inventura | `inventory.approve.any` |
| `menu` | Menu | `menu.channel.lvov.create` |
| `session` | Přihlášení | `session.viaCard` |
| `permission` | Oprávnění | `permission.assign` |

## ViewAny vs ViewOwn struktura

Vždy vytvoř OBA oprávnění pro čtení:

```php
// ViewAny - vidí všechny záznamy
case ArticleViewAny = 'article.view.any';

// ViewOwn - vidí jen vlastní záznamy (created_by)
case ArticleViewOwn = 'article.view.own';
```

**V Policy:**
- `viewAny()` - kontroluje OBA oprávnění
- `view()` - kontroluje ViewAny NEBO (ViewOwn + vlastnictví)

## UpdateAny vs UpdateOwn struktura

```php
// UpdateAny - upravuje všechny záznamy
case ArticleUpdateAny = 'article.update.any';

// UpdateOwn - upravuje jen vlastní záznamy
case ArticleUpdateOwn = 'article.update.own';
```

**V Policy:**
- `update()` - kontroluje UpdateAny NEBO (UpdateOwn + vlastnictví)

**⚠️ Důležité:**
- **PascalCase** pro case názvy (`ArticleCreate`)
- **camelCase entita** + **snake_case akce** pro hodnoty (`article.create`)
- **Maximálně 125 znaků** pro hodnotu
- **ViewAny/ViewOwn, UpdateAny/UpdateOwn** - vždy oba páry
- **Konzistentní prefix** pro skupiny oprávnění
- **Spustit artisan command** po přidání

**📘 Viz také:**
- **[Permission API Mappings](permissions-api-mappings.md)** - Lidsky čitelné názvy
- **[Permission Descriptions](permissions-descriptions.md)** - Detailní popisy
- **[Permission Groups](permissions-groups.md)** - Skupiny oprávnění
- **[Permission Artisan Command](permissions-artisan-command.md)** - Vytvoření v DB

Reference: [Policy Structure](policy-structure.md)