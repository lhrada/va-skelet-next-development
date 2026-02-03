---
title: Model Activity Log - logování změn
impact: MEDIUM
impactDescription: Activity log zajišťuje audit trail pro důležité změny
tags: model, activity-log, audit, logging
---

## Model Activity Log - logování změn

**Impact: MEDIUM**

Pro modely, kde potřebuješ sledovat změny pro audit, použij trait `LogsActivity` a interface `HasLogsActivity`.

**Kdy logovat změny:**
- Důležité entity (articles, products, orders)
- Změny důležité pro audit
- Když potřebuješ vidět kdo a kdy změnil data

**Kdy NELOGOVAT:**
- Technické tabulky (sessions, cache)
- Velmi často měněná data (statistics, counters)
- Dočasná data

## Implementace

**Correct - Základní logování:**

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Frame\Illuminate\Database\Eloquent\LogsActivity;
use Frame\Illuminate\Database\Eloquent\Interfaces\HasLogsActivity;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 * @property int $id
 * @property string $title
 * @property string|null $perex
 * @property bool $published
 */
final class Article extends Model implements HasLogsActivity
{
    use SoftDeletes;
    use LogsActivity;  // Přidej LogsActivity trait

    protected $fillable = [
        'title',
        'perex',
        'published',
    ];

    protected function casts(): array
    {
        return [
            'published' => 'boolean',
        ];
    }
}
```

## Co se loguje automaticky

- ✅ **Všechny změny atributů** (pokud se hodnota změní)
- ❌ **Vyloučeny automaticky**: `created_at`, `updated_at` (nastaveno v traitu)
- ❌ **Prázdné logy se neukládají**

## Customizace logování (pokročilé)

Pokud potřebuješ logovat jen některé sloupce nebo vyloučit citlivá data:

```php
use Spatie\Activitylog\LogOptions;
use Frame\ActivityLog\LogName;

final class Article extends Model implements HasLogsActivity
{
    use LogsActivity;

    public function getActivitylogOptions(): LogOptions
    {
        return LogOptions::defaults()
            ->useLogName(LogName::Model->value)
            ->logOnly(['title', 'perex', 'published'])  // Jen tyto sloupce
            ->logExcept(['password', 'secret'])         // Vyloučit citlivé sloupce
            ->dontSubmitEmptyLogs();
    }
}
```

## Automatická registrace v Mapperu

Model s `HasLogsActivity` interface se **automaticky zobrazí** v administraci:

1. **Mapper najde model** - `frame/ActivityLog/Mapper.php` automaticky načte všechny modely s `HasLogsActivity`
2. **Uživatel vidí v administraci** - při filtrování activity logu může vybrat tento model

**Ruční registrace v Mapperu:**

Po vytvoření nového modelu aktualizuj `frame/ActivityLog/Mapper.php`:

```php
<?php

namespace Frame\ActivityLog;

// 1. Přidej use statement (alfabeticky)
use App\Models\Article;
use App\Models\Product;
use App\Models\Order;

final class Mapper
{
    // 2. Přidej do Mapping array (alfabeticky)
    private const Mapping = [
        'article' => Article::class,
        'order' => Order::class,
        'product' => Product::class,
    ];
}
```

**⚠️ Klíč v mappingu je vždy camelCase** název modelu.

## Příklady z projektu

```php
// Article - loguje všechny změny
use LogsActivity;

// Product - loguje jen důležité sloupce
public function getActivitylogOptions(): LogOptions
{
    return LogOptions::defaults()
        ->logOnly(['name', 'price', 'published', 'stock']);
}

// User - vyloučí citlivé údaje
public function getActivitylogOptions(): LogOptions
{
    return LogOptions::defaults()
        ->logExcept(['password', 'remember_token']);
}
```

## Zobrazení logů

Pro zobrazení logů v administraci:

```php
// Všechny logy pro model
$logs = Activity::query()
    ->where('subject_type', Article::class)
    ->where('subject_id', $article->id)
    ->get();

// Kdo provedl změnu
$log->causer; // User model

// Co se změnilo
$log->properties; // ['attributes' => [...], 'old' => [...]]
```

**⚠️ Důležité:**
- **Vždy implementuj `HasLogsActivity` interface** - zajistí registraci v Mapperu
- **Trait `LogsActivity`** zajišťuje automatické logování
- **`created_at`, `updated_at`** se NELOGUJÍ automaticky
- **Prázdné logy** (žádná změna) se neukládají
- **Klíč v Mapperu** je camelCase název modelu

**📘 Dokumentace:**
- **[Spatie Laravel Activitylog](https://spatie.be/docs/laravel-activitylog)** - Oficiální dokumentace balíčku
- **[Frame\ActivityLog\Mapper](../../frame/ActivityLog/Mapper.php)** - Mapování modelů v projektu

Reference: [Model Structure](model-structure.md)
