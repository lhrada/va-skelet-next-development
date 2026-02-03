---
title: Artisan scaffolding příkazy
impact: LOW
impactDescription: Artisan make příkazy pro generování kódu
tags: artisan, scaffolding, generator, laravel
---

## Artisan scaffolding příkazy

**Impact: LOW**

Laravel Artisan poskytuje `make:*` příkazy pro rychlé generování kódu.

**⚠️ Vždy spouštěj přes docker compose:**

```bash
docker compose run php php artisan make:{type} {name}
```

## Controller

### Základní controller

```bash
# Admin controller
docker compose run php php artisan make:controller Admin/Products/ProductController

# Public controller
docker compose run php php artisan make:controller Public/Articles/ArticleController

# Společný controller
docker compose run php php artisan make:controller AuthController
```

### S options

```bash
# Resource controller (CRUD metody)
docker compose run php php artisan make:controller Admin/Products/ProductController --resource

# API resource controller (bez create/edit)
docker compose run php php artisan make:controller Admin/Products/ProductController --api

# Invokable controller (single action)
docker compose run php php artisan make:controller Admin/Products/ExportController --invokable
```

**Poznámka:** Většinou generuješ bez options a pak ručně upravuješ podle [controller-structure.md](controller-structure.md)

## Model

### Základní model

```bash
docker compose run php php artisan make:model Product
```

### S options (kombinace)

```bash
# S migrací
docker compose run php php artisan make:model Product -m

# S factory
docker compose run php php artisan make:model Product -f

# S seederem
docker compose run php php artisan make:model Product -s

# Vše najednou (migration, factory, seeder)
docker compose run php php artisan make:model Product -mfs

# Vše + controller
docker compose run php php artisan make:model Product -mfsc

# Vše + resource controller
docker compose run php php artisan make:model Product --all
```

### Options přehled

| Option | Zkratka | Generuje |
|--------|---------|----------|
| `--migration` | `-m` | Migration |
| `--factory` | `-f` | Factory |
| `--seed` | `-s` | Seeder |
| `--controller` | `-c` | Controller |
| `--resource` | `-r` | Resource controller |
| `--all` | - | Vše výše |

## Migration

```bash
# Vytvoření tabulky
docker compose run php php artisan make:migration create_products_table

# Přidání sloupců
docker compose run php php artisan make:migration add_published_to_products_table

# Změna tabulky
docker compose run php php artisan make:migration modify_products_table
```

**Konvence pojmenování:**
- `create_{table}_table` - nová tabulka
- `add_{column}_to_{table}_table` - přidání sloupce
- `modify_{table}_table` - úprava tabulky

Viz [migration-naming.md](migration-naming.md)

## Policy

```bash
# Základní policy
docker compose run php php artisan make:policy ProductPolicy

# S modelem
docker compose run php php artisan make:policy ProductPolicy --model=Product
```

## Seeder

```bash
docker compose run php php artisan make:seeder ProductSeeder
docker compose run php php artisan make:seeder PermissionSeeder
```

## Factory

```bash
# Factory pro model
docker compose run php php artisan make:factory ProductFactory

# S modelem
docker compose run php php artisan make:factory ProductFactory --model=Product
```

## Test

```bash
# Feature test (výchozí)
docker compose run php php artisan make:test ProductTest

# Unit test
docker compose run php php artisan make:test ProductTest --unit

# Test s PHPUnit
docker compose run php php artisan make:test ProductTest --phpunit
```

## Generic Class

Pro custom třídy (Services, Helpers, atd.):

```bash
# Service
docker compose run php php artisan make:class Domain/Products/ProductService

# Helper
docker compose run php php artisan make:class Helpers/StringHelper

# Custom třída
docker compose run php php artisan make:class Frame/Validation/CustomValidator
```

## Request (Form Request)

```bash
# Form request pro validaci
docker compose run php php artisan make:request StoreProductRequest
docker compose run php php artisan make:request UpdateProductRequest
```

**Poznámka:** V projektu používáme `*Validation` třídy místo Form Requests. Viz [validation-structure.md](validation-structure.md)

## Resource

```bash
# API Resource
docker compose run php php artisan make:resource ProductResource

# Resource collection
docker compose run php php artisan make:resource ProductCollection
```

## Middleware

```bash
docker compose run php php artisan make:middleware EnsureTenantMiddleware
```

## Command

```bash
# Artisan command
docker compose run php php artisan make:command CreatePermissions
```

## Job

```bash
# Queue job
docker compose run php php artisan make:job ProcessOrder
docker compose run php php artisan make:job SendEmailJob
```

## Event & Listener

```bash
# Event
docker compose run php php artisan make:event OrderCreated

# Listener
docker compose run php php artisan make:listener SendOrderConfirmation --event=OrderCreated
```

## Mail

```bash
# Mailable
docker compose run php php artisan make:mail OrderShipped
```

## Notification

```bash
# Notification
docker compose run php php artisan make:notification InvoicePaid
```

## Rule

```bash
# Validation rule
docker compose run php php artisan make:rule Uppercase
```

## Cast

```bash
# Custom cast
docker compose run php php artisan make:cast Json
```

## Observer

```bash
# Model observer
docker compose run php php artisan make:observer ProductObserver --model=Product
```

## Nejčastější kombinace

### Nový REST modul

```bash
# 1. Model + migrace + factory + seeder
docker compose run php php artisan make:model Product -mfs

# 2. Controller
docker compose run php php artisan make:controller Admin/Products/ProductController

# 3. Policy
docker compose run php php artisan make:policy ProductPolicy --model=Product

# 4. Test
docker compose run php php artisan make:test ProductTest

# 5. Service (generic class)
docker compose run php php artisan make:class Domain/Products/ProductService
```

### Nový enum nebo helper

```bash
# Enum
docker compose run php php artisan make:class Domain/ProductType

# Helper
docker compose run php php artisan make:class Helpers/PriceHelper
```

## Poznámky

### Laravel 12 vs Laravel 10 struktura

Projekt byl upgradován z Laravel 10, **zachována původní struktura**:
- Middleware v `app/Http/Middleware/`
- Kernel v `app/Http/Kernel.php`
- Exceptions v `app/Exceptions/Handler.php`

**NEPŘECHÁZEJ** na novou Laravel 12 strukturu (`bootstrap/app.php`) bez explicitního požadavku.

### Namespace konvence

**⚠️ Projekt má vlastní namespace strukturu:**

- **Controllers**: `App\Http\Controllers\{Admin|Public}\{Modul}\{Název}Controller`
- **Validations**: `App\Http\Controllers\{Admin|Public}\{Modul}\{Název}Validation`
- **Resources**: `App\Http\Controllers\{Admin|Public}\{Modul}\{Název}Resource`
- **Services**: `Domain\{Modul}\{Název}Service`
- **Policies**: `App\Policies\{Název}Policy`

Viz [controller-structure.md](controller-structure.md)

### Po vygenerování

**VŽDY:**
1. ✅ Přidej `declare(strict_types=1);`
2. ✅ Přidej `final` k třídám (pokud nemají potomky)
3. ✅ Přidej PHPDoc s properties
4. ✅ Uprav namespace podle konvencí projektu
5. ✅ Implementuj potřebné interface
6. ✅ Přidej readonly k properties

Viz [code-style-strict-types.md](code-style-strict-types.md)

## Checklist pro scaffolding

- [ ] Použil jsi správný namespace podle typu (Admin/Public)
- [ ] Pojmenoval jsi soubor podle konvencí (PascalCase)
- [ ] Přidal jsi `declare(strict_types=1);`
- [ ] Přidal jsi `final` k třídě
- [ ] Aktualizoval jsi PHPDoc
- [ ] Zkontroloval jsi generované import statements

**⚠️ Důležité:**
- **docker compose run** - pro všechny artisan příkazy
- **Namespace** - podle konvencí projektu
- **declare(strict_types=1)** - vždy na začátek
- **final class** - pokud nemá potomky
- **PHPDoc** - pro properties a return types

Reference: [Controller Structure](controller-structure.md), [Code Style Strict Types](code-style-strict-types.md)
