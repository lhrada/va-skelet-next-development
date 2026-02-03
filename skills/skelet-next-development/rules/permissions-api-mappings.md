---
title: Permission apiMappings - lidsky čitelné názvy
impact: HIGH
impactDescription: apiMappings poskytuje české názvy pro UI
tags: permissions, api-mappings, translations, ui
---

## Permission apiMappings - lidsky čitelné názvy

**Impact: HIGH**

Metoda `apiMappings()` vrací lidsky čitelné české názvy oprávnění pro zobrazení v UI.

**Soubor:** `frame/Permissions/Permission.php`  
**Metoda:** `public static function apiMappings(): array`

## Konvence pro názvy

- **Česky psané** popisy
- **Stručné, výstižné**
- Formát: `{Entity} - {akce}` nebo `{Entity} - {akce} {rozsah}`
- Používá funkci `__()` pro překlady

## Vzory popisů akcí

| Akce enum | Text v apiMappings |
|-----------|-------------------|
| `Create` | `vytvoření` |
| `ViewAny` | `čtení všech` |
| `ViewOwn` | `čtení vlastních` |
| `UpdateAny` | `úprava všech` |
| `UpdateOwn` | `úprava vlastních` |
| `DeleteAny` | `smazání` nebo `smazání všech` |
| `DeleteOwn` | `smazání vlastních` |
| `Approve` | `zpracování (potvrzení)` nebo `schválení` |
| `Reject` | `zamítnutí` |
| `Cancel` | `zrušení` |
| `Merge` | `slučování` |
| `Activate` | `aktivace` |
| `Deactivate` | `deaktivace` |
| `Publish` | `publikování` |
| `Unpublish` | `odpublikování` |

## Struktura metody

```php
public static function apiMappings(): array
{
    return [
        // Speciální oprávnění
        self::Root->value => __('Vše'),
        self::Routine->value => __('Rutinní úkony'),
        
        // Články - CRUD
        self::ArticleCreate->value => __('Články - vytvoření'),
        self::ArticleViewAny->value => __('Články - čtení všech'),
        self::ArticleViewOwn->value => __('Články - čtení vlastních'),
        self::ArticleUpdateAny->value => __('Články - úprava všech'),
        self::ArticleUpdateOwn->value => __('Články - úprava vlastních'),
        self::ArticleDeleteAny->value => __('Články - smazání'),
        self::ArticleDeleteOwn->value => __('Články - smazání vlastních'),
        
        // Produkty
        self::ProductCreate->value => __('Produkty - vytvoření'),
        self::ProductViewAny->value => __('Produkty - čtení všech'),
        self::ProductUpdateAny->value => __('Produkty - úprava všech'),
        self::ProductDelete->value => __('Produkty - smazání'),
        
        // Specifické akce
        self::InventoryApproveAny->value => __('Inventura - zpracování (potvrzení)'),
        self::InventoryCancelAny->value => __('Inventura - zrušení'),
        self::CouponActivate->value => __('Kupony - aktivace'),
        self::CouponDeactivate->value => __('Kupony - deaktivace'),
        self::ProfileMerge->value => __('Osoby - slučování osob'),
        
        // Přihlášení
        self::SessionViaCard->value => __('Přihlášení do systému přes kartu'),
        
        // Tiskárna
        self::Printer->value => __('Tiskárna'),
    ];
}
```

## Příklady podle typu

### Základní CRUD

```php
// Článek
self::ArticleCreate->value => __('Články - vytvoření'),
self::ArticleViewAny->value => __('Články - čtení všech'),
self::ArticleUpdateAny->value => __('Články - úprava všech'),
self::ArticleDeleteAny->value => __('Články - smazání'),

// Produkt
self::ProductCreate->value => __('Produkty - vytvoření'),
self::ProductViewAny->value => __('Produkty - čtení všech'),
self::ProductUpdateAny->value => __('Produkty - úprava všech'),
self::ProductDelete->value => __('Produkty - smazání'),
```

### ViewOwn/UpdateOwn

```php
// Vlastní záznamy
self::ArticleViewOwn->value => __('Články - čtení vlastních'),
self::ArticleUpdateOwn->value => __('Články - úprava vlastních'),
self::ArticleDeleteOwn->value => __('Články - smazání vlastních'),
```

### Specifické akce

```php
// Inventura
self::InventoryApproveAny->value => __('Inventura - zpracování (potvrzení)'),
self::InventoryCancelAny->value => __('Inventura - zrušení'),

// Kupony
self::CouponActivate->value => __('Kupony - aktivace'),
self::CouponDeactivate->value => __('Kupony - deaktivace'),

// Slučování
self::ProfileMerge->value => __('Osoby - slučování osob'),

// Publikování
self::ArticlePublish->value => __('Články - publikování'),
self::ArticleUnpublish->value => __('Články - odpublikování'),
```

### Kanálová oprávnění

```php
self::MenuChannelLvovCreate->value => __('Menu (Lvov) - vytvoření'),
self::MenuChannelLvovViewAny->value => __('Menu (Lvov) - čtení všech'),
self::MenuChannelPragueCreate->value => __('Menu (Praha) - vytvoření'),
```

### Speciální oprávnění

```php
self::Root->value => __('Vše'),
self::Routine->value => __('Rutinní úkony'),
self::Printer->value => __('Tiskárna'),
self::SessionViaCard->value => __('Přihlášení do systému přes kartu'),
self::Kiosk->value => __('Kiosk'),
self::Erp->value => __('ERP systém'),
self::Economics->value => __('Ekonomika'),
self::Logistic->value => __('Logistika'),
```

## Pravidla formátování

### ✅ Správně

```php
// Entita - akce
self::ArticleCreate->value => __('Články - vytvoření'),
self::ProductViewAny->value => __('Produkty - čtení všech'),

// Entita (specifikace) - akce
self::MenuChannelLvovCreate->value => __('Menu (Lvov) - vytvoření'),

// Entita - akce s detailem
self::InventoryApproveAny->value => __('Inventura - zpracování (potvrzení)'),

// Věta pro speciální oprávnění
self::SessionViaCard->value => __('Přihlášení do systému přes kartu'),
```

### ❌ Špatně

```php
// Chybí funkce __()
self::ArticleCreate->value => 'Články - vytvoření',

// Není česky
self::ArticleCreate->value => __('Articles - create'),

// Špatný formát
self::ArticleCreate->value => __('Vytvoření článků'),
self::ArticleCreate->value => __('článek vytvoření'),
```

## Použití v API

apiMappings se používá v:

1. **EnumController** - endpoint `/api/enums/permissions`
2. **Permission enum** - metoda `pair()`
3. **descriptions()** - jako `name` pro detaily

```php
// EnumController
public function permissions(Request $request): Responsable
{
    $permissions = collect(Permission::cases())
        ->map(fn(Permission $permission) => [
            'value' => $permission->value,
            'name' => Permission::apiMappings()[$permission->value] ?? $permission->value,
        ])
        ->values()
        ->all();
    
    return $this->response($permissions);
}
```

## Metoda pair()

Každé oprávnění má metodu `pair()` která využívá apiMappings:

```php
public function pair(): array
{
    return [
        'value' => $this->value,
        'name' => self::apiMappings()[$this->value] ?? $this->value,
        'description' => self::descriptions()[$this->value]['description'] ?? null,
    ];
}

// Použití:
Permission::ArticleCreate->pair();
// Vrátí: ['value' => 'article.create', 'name' => 'Články - vytvoření', 'description' => '...']
```

## Přidání nového mappingu

Při přidání nového oprávnění:

```php
public static function apiMappings(): array
{
    return [
        // ...existing code...
        
        // Nové oprávnění - přidej na správné místo (podle entity)
        self::LandmarkCreate->value => __('Památky - vytvoření'),
        self::LandmarkViewAny->value => __('Památky - čtení všech'),
        self::LandmarkUpdateAny->value => __('Památky - úprava všech'),
        self::LandmarkDelete->value => __('Památky - smazání'),
        
        // ...existing code...
    ];
}
```

**⚠️ Důležité:**
- **VŽDY použij funkci `__()`** pro překlad
- **Česky psané** názvy
- **Formát: `{Entity} - {akce}`**
- **Stručné a výstižné** názvy
- **Konzistentní** s ostatními oprávněními stejné entity
- **ViewOwn/UpdateOwn** má suffix `vlastních`
- **ViewAny/UpdateAny** má suffix `všech`

**📘 Viz také:**
- **[Permission Enum Naming](permissions-enum-naming.md)** - Pojmenování case
- **[Permission Descriptions](permissions-descriptions.md)** - Detailní popisy
- **[Permission Groups](permissions-groups.md)** - Skupiny oprávnění

Reference: [Permission Enum Naming](permissions-enum-naming.md)
