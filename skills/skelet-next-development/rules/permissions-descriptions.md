---
title: Permission descriptions - detailní popisy
impact: MEDIUM
impactDescription: Descriptions poskytují detailní popisy oprávnění pro administraci
tags: permissions, descriptions, documentation
---

## Permission descriptions - detailní popisy

**Impact: MEDIUM**

Metoda `descriptions()` vrací detailní popisy oprávnění pro zobrazení v administraci.

**Soubor:** `frame/Permissions/Permission.php`  
**Metoda:** `public static function descriptions(): array`

## Konvence pro popisy

- Vrací array s klíči: `value`, `name`, `description`
- **`.own` oprávnění se NEZOBRAZUJÍ** - jsou automaticky vynechána
- Popis začíná: `Oprávnění k ...` nebo `Oprávnění pro ...` nebo `Povolí ...`
- Detailnější než apiMappings, vysvětluje účel oprávnění

## Struktura záznamu

```php
self::ArticleCreate->value => [
    'value' => self::ArticleCreate->value,
    'name' => self::apiMappings()[self::ArticleCreate->value],
    'description' => __('Oprávnění k vytváření článků.'),
],
```

## Struktura metody

```php
public static function descriptions(): array
{
    $payload = [
        // Speciální oprávnění
        self::Root->value => [
            'value' => self::Root->value,
            'name' => self::apiMappings()[self::Root->value],
            'description' => __('Oprávnění pro speciální účely.'),
        ],
        
        // Články - jen Any oprávnění, vynecháváme .own
        self::ArticleCreate->value => [
            'value' => self::ArticleCreate->value,
            'name' => self::apiMappings()[self::ArticleCreate->value],
            'description' => __('Oprávnění k vytváření článků.'),
        ],
        self::ArticleViewAny->value => [
            'value' => self::ArticleViewAny->value,
            'name' => self::apiMappings()[self::ArticleViewAny->value],
            'description' => __('Oprávnění k čtení všech článků.'),
        ],
        self::ArticleUpdateAny->value => [
            'value' => self::ArticleUpdateAny->value,
            'name' => self::apiMappings()[self::ArticleUpdateAny->value],
            'description' => __('Oprávnění k úpravě všech článků.'),
        ],
        self::ArticleDeleteAny->value => [
            'value' => self::ArticleDeleteAny->value,
            'name' => self::apiMappings()[self::ArticleDeleteAny->value],
            'description' => __('Oprávnění ke smazání článků.'),
        ],
        
        // VYNECHÁVÁME .own oprávnění:
        // self::ArticleViewOwn - NEVKLÁDÁME
        // self::ArticleUpdateOwn - NEVKLÁDÁME
        // self::ArticleDeleteOwn - NEVKLÁDÁME
    ];
    
    // Automatické doplnění chybějících oprávnění (bez .own)
    foreach (self::cases() as $permission) {
        if (!array_key_exists($permission->value, $payload) && !Str::of($permission->value)->endsWith('.own')) {
            $name = self::apiMappings()[$permission->value] ?? $permission->value;
            $payload[$permission->value] = [
                'value' => $permission->value,
                'name' => $name,
                'description' => __('Oprávnění pro :permission', ['permission' => $name]),
            ];
        }
    }
    
    return $payload;
}
```

## Vzory popisů

### Začátek popisu

| Začátek | Použití | Příklad |
|---------|---------|---------|
| `Oprávnění k ...` | Nejčastější | `Oprávnění k vytváření článků.` |
| `Oprávnění pro ...` | Speciální | `Oprávnění pro speciální účely.` |
| `Povolí ...` | Akce | `Povolí přihlašovat se přes kartu.` |

### Příklady podle typu

**CRUD operace:**

```php
self::ArticleCreate->value => [
    'value' => self::ArticleCreate->value,
    'name' => self::apiMappings()[self::ArticleCreate->value],
    'description' => __('Oprávnění k vytváření článků.'),
],

self::ArticleViewAny->value => [
    'value' => self::ArticleViewAny->value,
    'name' => self::apiMappings()[self::ArticleViewAny->value],
    'description' => __('Oprávnění k čtení všech článků.'),
],

self::ArticleUpdateAny->value => [
    'value' => self::ArticleUpdateAny->value,
    'name' => self::apiMappings()[self::ArticleUpdateAny->value],
    'description' => __('Oprávnění k úpravě všech článků.'),
],

self::ArticleDeleteAny->value => [
    'value' => self::ArticleDeleteAny->value,
    'name' => self::apiMappings()[self::ArticleDeleteAny->value],
    'description' => __('Oprávnění ke smazání článků.'),
],
```

**Speciální:**

```php
self::Root->value => [
    'value' => self::Root->value,
    'name' => self::apiMappings()[self::Root->value],
    'description' => __('Oprávnění pro speciální účely.'),
],

self::SessionViaCard->value => [
    'value' => self::SessionViaCard->value,
    'name' => self::apiMappings()[self::SessionViaCard->value],
    'description' => __('Povolí přihlašovat se přes kartu.'),
],

self::PermissionAssign->value => [
    'value' => self::PermissionAssign->value,
    'name' => self::apiMappings()[self::PermissionAssign->value],
    'description' => __(
        'Oprávnění k vytváření a úpravě oprávnění a rolí. Tato role umožňuje přidělovat role a kanály ostatním uživatelům.'
    ),
],
```

**Složitější popisy:**

```php
self::WarehouseMovement->value => [
    'value' => self::WarehouseMovement->value,
    'name' => self::apiMappings()[self::WarehouseMovement->value],
    'description' => __('Oprávnění ke sledování pohybu zboží na skladě.'),
],

self::InventoryApproveAny->value => [
    'value' => self::InventoryApproveAny->value,
    'name' => self::apiMappings()[self::InventoryApproveAny->value],
    'description' => __('Oprávnění ke zpracování a potvrzení inventury.'),
],
```

## Automatické doplnění

Na konci metody je automatické doplnění chybějících oprávnění:

```php
// Doplnit zbylá oprávnění pokud nejsou v payloadu
// .own oprávnění vynecháváme
foreach (self::cases() as $permission) {
    if (!array_key_exists($permission->value, $payload) && !Str::of($permission->value)->endsWith('.own')) {
        $name = self::apiMappings()[$permission->value] ?? $permission->value;
        $payload[$permission->value] = [
            'value' => $permission->value,
            'name' => $name,
            'description' => __('Oprávnění pro :permission', ['permission' => $name]),
        ];
    }
}
```

**Automaticky vygenerovaný popis:**
```
Oprávnění pro Produkty - vytvoření
```

## Pravidla

### ✅ Správně

```php
// Začíná "Oprávnění k..."
self::ArticleCreate->value => [
    'value' => self::ArticleCreate->value,
    'name' => self::apiMappings()[self::ArticleCreate->value],
    'description' => __('Oprávnění k vytváření článků.'),
],

// Začíná "Povolí..."
self::SessionViaCard->value => [
    'value' => self::SessionViaCard->value,
    'name' => self::apiMappings()[self::SessionViaCard->value],
    'description' => __('Povolí přihlašovat se přes kartu.'),
],

// Jen Any oprávnění, bez .own
self::ArticleViewAny->value => [ /* ... */ ],
// ArticleViewOwn - VYNECHÁME
```

### ❌ Špatně

```php
// Chybí funkce __()
description => 'Oprávnění k vytváření článků.',

// Není česky
description => __('Permission to create articles.'),

// Není tečka na konci
description => __('Oprávnění k vytváření článků'),

// Obsahuje .own oprávnění
self::ArticleViewOwn->value => [ /* ... */ ],  // NESPRÁVNĚ!
```

## Použití v API

```php
// EnumController
public function permissions(Request $request): Responsable
{
    $permissions = Permission::descriptions();
    
    return $this->response($permissions);
}

// Vrátí jen Any oprávnění, bez .own
```

## Přidání nového popisu

```php
public static function descriptions(): array
{
    $payload = [
        // ...existing code...
        
        // Nové oprávnění - přidej jen Any, ne .own
        self::LandmarkCreate->value => [
            'value' => self::LandmarkCreate->value,
            'name' => self::apiMappings()[self::LandmarkCreate->value],
            'description' => __('Oprávnění k vytváření památek.'),
        ],
        self::LandmarkViewAny->value => [
            'value' => self::LandmarkViewAny->value,
            'name' => self::apiMappings()[self::LandmarkViewAny->value],
            'description' => __('Oprávnění k čtení všech památek.'),
        ],
        // LandmarkViewOwn - VYNECHÁVÁME
        
        // ...existing code...
    ];
    
    // ...automatic filling...
}
```

**⚠️ Důležité:**
- **Vrací array s klíči**: `value`, `name`, `description`
- **VYNECHAT `.own` oprávnění** - nejsou v descriptions()
- **Začíná**: `Oprávnění k ...` nebo `Povolí ...`
- **VŽDY použij `__()`** pro překlad
- **Tečka na konci** popisu
- **Automatické doplnění** chybějících (bez .own)

**📘 Viz také:**
- **[Permission Enum Naming](permissions-enum-naming.md)** - Pojmenování case
- **[Permission API Mappings](permissions-api-mappings.md)** - Lidsky čitelné názvy
- **[Permission Groups](permissions-groups.md)** - Skupiny oprávnění

Reference: [Permission API Mappings](permissions-api-mappings.md)
