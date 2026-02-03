---
title: Permission Artisan Command - vytvoření v DB
impact: HIGH
impactDescription: Artisan command vytvoří oprávnění v databázi a přiřadí je rolím
tags: permissions, artisan, command, database
---

## Permission Artisan Command - vytvoření v DB

**Impact: HIGH**

Po přidání oprávnění do Permission enum je **POVINNÉ** spustit Artisan command, který vytvoří záznamy v databázi.

**Command:** `php artisan app:create-permissions`  
**Soubor:** `app/Console/Commands/CreatePermissions.php`

## Spuštění příkazu

### V dockeru (preferovaná metoda):

```bash
docker compose run php php artisan app:create-permissions
```

### Lokálně (pokud PHP běží lokálně):

```bash
php artisan app:create-permissions
```

## Co příkaz dělá

1. **Načte všechna oprávnění** z `Permission` enumu
2. **Vytvoří záznamy** v tabulce `permissions` (pokud neexistují)
3. **Automaticky přiřadí** nová oprávnění rolím:
   - Role **"admin"** dostane všechna oprávnění
   - Role **"superadmin"** dostane všechna oprávnění

## Výstup příkazu

```
Creating permissions...

✓ Created permission: article.create
✓ Created permission: article.view.any
✓ Created permission: article.update.any
✓ Permission already exists: product.create
✓ Permission already exists: product.view.any

Assigning permissions to roles...

✓ Assigned 5 new permissions to role: admin
✓ Assigned 5 new permissions to role: superadmin

Done!
```

## Kdy spustit příkaz

### ✅ VŽDY po:

1. **Přidání nového case** do Permission enum
2. **Změně hodnoty** existujícího oprávnění (změní se klíč v DB)
3. **Deployment** nové verze aplikace s novými oprávněními

### ❌ NENÍ potřeba po:

1. Změně `apiMappings()` (jen UI text)
2. Změně `descriptions()` (jen UI text)
3. Změně `getGroups()` (jen UI organizace)

## Tabulka permissions

Příkaz vytváří záznamy v této struktuře:

```sql
CREATE TABLE permissions (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(125) NOT NULL UNIQUE,  -- Hodnota z enumu (např. 'article.create')
    guard_name VARCHAR(125) NOT NULL,    -- Obvykle 'web'
    created_at TIMESTAMP NULL,
    updated_at TIMESTAMP NULL
);
```

**Příklad záznamu:**

| id | name | guard_name |
|----|------|------------|
| 1 | article.create | web |
| 2 | article.view.any | web |
| 3 | article.update.any | web |

## Role mapping

Příkaz automaticky přiřadí nová oprávnění těmto rolím:

```php
// V CreatePermissions.php
$roles = ['admin', 'superadmin'];

foreach ($roles as $roleName) {
    $role = Role::findByName($roleName);
    $role->syncPermissions(Permission::all());
}
```

**Výsledek:**
- Role **admin** - má všechna oprávnění
- Role **superadmin** - má všechna oprávnění

## Manuální přiřazení oprávnění

Pro jiné role použij administraci nebo tinker:

```php
// V tinker nebo seederu
use Spatie\Permission\Models\Role;
use Frame\Permissions\Permission;

$role = Role::findByName('editor');
$role->givePermissionTo(Permission::ArticleCreate->value);
$role->givePermissionTo(Permission::ArticleViewAny->value);
$role->givePermissionTo(Permission::ArticleUpdateAny->value);
```

## Kontrola oprávnění v databázi

```bash
# Tinker
docker compose run php php artisan tinker

# V tinkeru
Spatie\Permission\Models\Permission::where('name', 'like', 'article.%')->get();
```

## Příklad workflow

### 1. Přidat oprávnění do enumu

```php
// V frame/Permissions/Permission.php
case LandmarkCreate = 'landmark.create';
case LandmarkViewAny = 'landmark.view.any';
case LandmarkUpdateAny = 'landmark.update.any';
case LandmarkDelete = 'landmark.delete';
```

### 2. Přidat do apiMappings()

```php
self::LandmarkCreate->value => __('Památky - vytvoření'),
self::LandmarkViewAny->value => __('Památky - čtení všech'),
// ...
```

### 3. Spustit Artisan command

```bash
docker compose run php php artisan app:create-permissions
```

### 4. Ověřit v databázi

```bash
docker compose run php php artisan tinker

# V tinkeru
Spatie\Permission\Models\Permission::where('name', 'like', 'landmark.%')->get();
```

### 5. Použít v Policy

```php
public function create(?User $user): bool
{
    if ($user && $user->can(Permission::LandmarkCreate->value)) {
        return true;
    }
    
    throw new Forbidden('Landmark: Forbidden')
        ->convey(__('Nemáte oprávnění k vytvoření památky'));
}
```

## Troubleshooting

### Oprávnění se nevytvořilo

**Příčina:** Hodnota už existuje v DB s jiným názvem

**Řešení:**
```bash
# Smazat staré oprávnění
docker compose run php php artisan tinker

# V tinkeru
Spatie\Permission\Models\Permission::where('name', 'old-permission-name')->delete();

# Znovu spustit command
docker compose run php php artisan app:create-permissions
```

### Chyba "Permission already exists"

**Příčina:** Oprávnění už je v DB (to je v pořádku)

**Řešení:** Není potřeba nic dělat, command přeskočí existující oprávnění

### Role nemá nové oprávnění

**Příčina:** Role nebyla vytvořena před spuštěním commandu

**Řešení:**
```bash
# Vytvořit roli
docker compose run php php artisan tinker

# V tinkeru
Spatie\Permission\Models\Role::create(['name' => 'editor']);

# Znovu spustit command
docker compose run php php artisan app:create-permissions
```

## Seeder

Pro development/testing můžeš použít seeder:

```php
// database/seeders/PermissionSeeder.php
public function run(): void
{
    // Vytvoří všechna oprávnění
    Artisan::call('app:create-permissions');
    
    // Vytvoří role
    Role::create(['name' => 'editor']);
    Role::create(['name' => 'viewer']);
    
    // Přiřadí specifická oprávnění
    $editor = Role::findByName('editor');
    $editor->givePermissionTo([
        Permission::ArticleCreate->value,
        Permission::ArticleViewAny->value,
        Permission::ArticleUpdateAny->value,
    ]);
}
```

**⚠️ Důležité:**
- **VŽDY spusť** po přidání nových oprávnění do enumu
- **Docker compose run** pro spuštění v containeru
- **Automatické přiřazení** admin a superadmin rolím
- **Kontroluj výstup** pro chyby
- **V tinkeru** můžeš ověřit vytvoření

**📘 Viz také:**
- **[Permission Enum Naming](permissions-enum-naming.md)** - Pojmenování oprávnění
- **[Policy Structure](policy-structure.md)** - Použití oprávnění v Policy
- **Spatie Laravel Permission** - https://spatie.be/docs/laravel-permission

Reference: [Permission Enum Naming](permissions-enum-naming.md), [Policy Structure](policy-structure.md)
