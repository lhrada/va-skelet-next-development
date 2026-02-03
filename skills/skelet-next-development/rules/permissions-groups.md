---
title: Permission groups - skupiny oprávnění
impact: LOW
impactDescription: Groups organizují oprávnění v administraci
tags: permissions, groups, organization
---

## Permission groups - skupiny oprávnění

**Impact: LOW**

Metoda `getGroups()` vrací skupiny oprávnění pro organizaci v UI administrace.

**Soubor:** `frame/Permissions/Permission.php`  
**Metoda:** `public static function getGroups(): array`

## Kdy přidat novou skupinu

Když vytváříš **první oprávnění s novým prefixem**:
- Prefix před první tečkou určuje skupinu
- Např. `article.` → skupina `article`
- Např. `product.` → skupina `product`

## Struktura skupiny

```php
'article' => [
    'name' => __('Články'),
    'description' => __('Oprávnění pro články'),
],
```

**Volitelně s poznámkou:**

```php
'printer' => [
    'name' => __('Tisková fronta'),
    'description' => __('Oprávnění pro tiskovou frontu'),
    'note' => __('Toto oprávnění slouží jen pokladnám, řídícím jednotkám a tiskovým zařízením.'),
],
```

## Kompletní příklad

```php
public static function getGroups(): array
{
    return [
        // Základní entity
        'article' => [
            'name' => __('Články'),
            'description' => __('Oprávnění pro články'),
        ],
        
        'product' => [
            'name' => __('Produkty'),
            'description' => __('Oprávnění pro produkty'),
        ],
        
        'order' => [
            'name' => __('Objednávky'),
            'description' => __('Oprávnění pro objednávky'),
        ],
        
        'profile' => [
            'name' => __('Profily'),
            'description' => __('Oprávnění pro profily a osoby'),
        ],
        
        // Skladové hospodářství
        'warehouse' => [
            'name' => __('Sklady'),
            'description' => __('Oprávnění pro sklady'),
        ],
        
        'inventory' => [
            'name' => __('Inventura'),
            'description' => __('Oprávnění pro inventuru'),
        ],
        
        // Doprava
        'shipping' => [
            'name' => __('Doprava'),
            'description' => __('Oprávnění pro dopravu'),
            'note' => __('Dopravci, zóny, ceny, sloty'),
        ],
        
        // Systémové
        'user' => [
            'name' => __('Uživatelé'),
            'description' => __('Oprávnění pro uživatele'),
        ],
        
        'permission' => [
            'name' => __('Oprávnění'),
            'description' => __('Oprávnění pro správu oprávnění a rolí'),
        ],
        
        // Speciální
        'printer' => [
            'name' => __('Tisková fronta'),
            'description' => __('Oprávnění pro tiskovou frontu'),
            'note' => __('Toto oprávnění slouží jen pokladnám, řídícím jednotkám a tiskovým zařízením.'),
        ],
        
        'session' => [
            'name' => __('Přihlášení'),
            'description' => __('Oprávnění pro přihlášení'),
        ],
        
        // Kanály
        'menu' => [
            'name' => __('Menu'),
            'description' => __('Oprávnění pro menu'),
        ],
    ];
}
```

## Prefix → Skupina mapping

| Prefix oprávnění | Skupina | Název skupiny |
|------------------|---------|---------------|
| `article.*` | `article` | Články |
| `product.*` | `product` | Produkty |
| `order.*` | `order` | Objednávky |
| `person.*` nebo `profile.*` | `profile` | Profily |
| `warehouse.*` | `warehouse` | Sklady |
| `inventory.*` | `inventory` | Inventura |
| `shipping.*` | `shipping` | Doprava |
| `user.*` | `user` | Uživatelé |
| `permission.*` | `permission` | Oprávnění |
| `printer.*` | `printer` | Tisková fronta |
| `session.*` | `session` | Přihlášení |
| `menu.*` | `menu` | Menu |

## Příklady skupin s poznámkou

```php
'shipping' => [
    'name' => __('Doprava'),
    'description' => __('Oprávnění pro dopravu'),
    'note' => __('Dopravci, zóny, ceny, sloty'),
],

'printer' => [
    'name' => __('Tisková fronta'),
    'description' => __('Oprávnění pro tiskovou frontu'),
    'note' => __('Toto oprávnění slouží jen pokladnám, řídícím jednotkám a tiskovým zařízením.'),
],

'economics' => [
    'name' => __('Ekonomika'),
    'description' => __('Oprávnění pro ekonomiku'),
    'note' => __('Účetnictví, faktury, ceníky'),
],
```

## Použití v UI

Skupiny se používají v administraci pro:
1. **Kategorizaci oprávnění** - seskupení podobných oprávnění
2. **Filtrování** - rychlé hledání oprávnění podle typu
3. **Přehlednost** - lepší organizace dlouhého seznamu oprávnění

```php
// V administraci se oprávnění zobrazují po skupinách:
// 
// Články
//   ├─ Články - vytvoření
//   ├─ Články - čtení všech
//   ├─ Články - úprava všech
//   └─ Články - smazání
//
// Produkty
//   ├─ Produkty - vytvoření
//   ├─ Produkty - čtení všech
//   ├─ Produkty - úprava všech
//   └─ Produkty - smazání
```

## Přidání nové skupiny

Při přidání oprávnění s novým prefixem:

```php
// 1. Přidáš oprávnění s novým prefixem
case LandmarkCreate = 'landmark.create';
case LandmarkViewAny = 'landmark.view.any';

// 2. Přidáš novou skupinu do getGroups()
public static function getGroups(): array
{
    return [
        // ...existing code...
        
        'landmark' => [
            'name' => __('Památky'),
            'description' => __('Oprávnění pro památky'),
        ],
        
        // ...existing code...
    ];
}
```

## Pravidla

### ✅ Správně

```php
// Základní skupina
'article' => [
    'name' => __('Články'),
    'description' => __('Oprávnění pro články'),
],

// Skupina s poznámkou
'shipping' => [
    'name' => __('Doprava'),
    'description' => __('Oprávnění pro dopravu'),
    'note' => __('Dopravci, zóny, ceny, sloty'),
],

// Klíč odpovídá prefixu oprávnění
'product' => [ /* ... */ ],  // Pro product.create, product.view.any, atd.
```

### ❌ Špatně

```php
// Chybí funkce __()
'article' => [
    'name' => 'Články',
    'description' => 'Oprávnění pro články',
],

// Klíč neodpovídá prefixu
'articles' => [ /* ... */ ],  // Pro article.create - ŠPATNĚ, má být 'article'

// Není česky
'article' => [
    'name' => __('Articles'),
    'description' => __('Permissions for articles'),
],
```

## Speciální skupiny

Některé skupiny nemají standardní strukturu:

```php
// Root, routine - speciální oprávnění
'root' => [
    'name' => __('Speciální'),
    'description' => __('Speciální oprávnění pro pokročilé funkce'),
],

// Kiosk, ERP - skupinová oprávnění
'kiosk' => [
    'name' => __('Kiosk'),
    'description' => __('Oprávnění pro kiosk aplikaci'),
],

'erp' => [
    'name' => __('ERP systém'),
    'description' => __('Oprávnění pro ERP systém'),
],
```

**⚠️ Důležité:**
- **Klíč = prefix** oprávnění (před první tečkou)
- **VŽDY použij `__()`** pro překlad
- **Česky psané** názvy a popisy
- **Volitelná poznámka** (`note`) pro dodatečné info
- **Přidej skupinu** při prvním oprávnění s novým prefixem
- **Organizuje oprávnění** v UI administrace

**📘 Viz také:**
- **[Permission Enum Naming](permissions-enum-naming.md)** - Pojmenování case
- **[Permission API Mappings](permissions-api-mappings.md)** - Lidsky čitelné názvy
- **[Permission Descriptions](permissions-descriptions.md)** - Detailní popisy

Reference: [Permission Enum Naming](permissions-enum-naming.md)
