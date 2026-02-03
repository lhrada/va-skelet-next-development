---
title: Elasticsearch implementace - Scout + Explorer
impact: HIGH
impactDescription: Elasticsearch zajišťuje full-text vyhledávání v projektu
tags: elasticsearch, scout, explorer, search, indexing
---

## Elasticsearch implementace - Scout + Explorer

**Impact: HIGH**

Projekt používá **Laravel Scout** s **jeroen-g/explorer** (Elasticsearch 8) pro full-text vyhledávání.

**Balíčky:** `laravel/scout` + `jeroen-g/explorer`  
**Konfigurace:** `config/scout.php`, `config/explorer.php`  
**Elasticsearch verze:** 8.x

## Typy implementace

### 1. Admin (pro administraci)

Model **přímo** implementuje `Explored` interface a je indexován pro vyhledávání v administraci.

**Použití:** Pokud potřebuješ vyhledávání **pouze v administraci**.

**Modely s admin implementací:**
- `Article`, `Product`, `Param`, `Order`, `Tag`, `Profile`, `Landmark`

### 2. Web (pro veřejnou část)

Vytváří se **separátní search model** v `app/Models/Search/Web/`, který extenduje původní model.

**Použití:** Pokud potřebuješ vyhledávání **na veřejné části** (eshop, web).

**Modely s web implementací:**
- `Product` → `App\Models\Search\Web\Product`
- `Article` → `App\Models\Search\Web\Article`

### 3. Obojí (Admin + Web)

Model má **obě implementace** - přímo implementuje `Explored` pro admin + má search model v `Search/Web/` pro web.

**Použití:** Pokud potřebuješ vyhledávání **jak v administraci, tak na webu**.

**Modely s oběma implementacemi:**
- `Product` (admin i web)
- `Article` (admin i web)

## Implementace - Admin

### Krok 1: Registrovat model v konfiguraci

**⚠️ DŮLEŽITÉ:** Před jakoukoliv prací s indexem musíš přidat model do `config/explorer.php`:

```php
'indexes' => [
    // ...existující modely...
    \App\Models\YourModel::class, // <-- přidej svůj model
],
```

Bez této registrace nebudou fungovat příkazy `php artisan explorer:*`!

### Krok 2: Přidat interface a traits do modelu

```php
<?php

declare(strict_types=1);

namespace App\Models;

use JeroenG\Explorer\Application\Explored;
use JeroenG\Explorer\Application\IndexSettings;
use Laravel\Scout\Searchable;
use Frame\Illuminate\Database\Eloquent\DefaultSearchIndexSetting;
use Frame\Illuminate\Database\Eloquent\Scopes\IsSearchable;

class YourModel extends Model implements
    Explored,
    IndexSettings
{
    use Searchable;
    use DefaultSearchIndexSetting;
    use IsSearchable; // přidává scope pro is_searchable
    
    // ...
}
```

### Krok 3: Přidat sloupec `is_searchable` do tabulky

Vytvoř migraci:
```bash
docker compose run php php artisan make:migration add_is_searchable_to_your_models_table
```

```php
public function up(): void
{
    Schema::table('your_models', function (Blueprint $table) {
        $table->boolean('is_searchable')->default(false)->after('published');
        $table->index('is_searchable');
    });
}
```

### Krok 4: Přidat do fillable a casts

```php
protected $fillable = [
    // ...existing fields...
    'is_searchable',
];

protected function casts(): array
{
    return [
        // ...existing casts...
        'is_searchable' => 'bool',
    ];
}
```

### Krok 5: Implementovat `mappableAs()` metodu

Definuje **strukturu indexu** v Elasticsearch:

```php
public function mappableAs(): array
{
    return [
        'id' => 'keyword',
        'name' => ['type' => 'text', 'analyzer' => 'standard'],
        'description' => ['type' => 'text'],
        'published' => 'boolean',
        'created_at' => 'date',
        // Přidej všechny fieldy, které chceš vyhledávat
    ];
}
```

### Krok 6: Implementovat `toSearchableArray()` metodu

Definuje, **jaká data** se uloží do indexu:

```php
public function toSearchableArray(): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'description' => $this->description,
        'published' => $this->published,
        'created_at' => $this->created_at?->toIso8601String(),
        // Přidej všechny fieldy, které chceš indexovat
    ];
}
```

### Krok 7: Implementovat `shouldBeSearchable()` metodu

Určuje, **které záznamy** se mají indexovat:

```php
public function shouldBeSearchable(): bool
{
    return $this->is_searchable 
        && $this->published 
        && !$this->trashed();
}
```

### Krok 8: Scout příkazy

```bash

# Vytvoření indexu

docker compose run php php artisan explorer:create YourModel

# Import všech záznamů

docker compose run php php artisan scout:import "App\Models\YourModel"

# Smazání indexu

docker compose run php php artisan explorer:delete YourModel

# Flush a reimport

docker compose run php php artisan scout:flush "App\Models\YourModel"
docker compose run php php artisan scout:import "App\Models\YourModel"
```

## Implementace - Web

### Krok 1: Registrovat v konfiguraci

V `config/explorer.php` přidej **web model**:

```php
'indexes' => [
    // Admin modely
    \App\Models\YourModel::class,
    
    // Virtuální modely pro hledání na webu
    \App\Models\Search\Web\YourModel::class, // <-- přidej web model
],
```

### Krok 2: Vytvoř Search model

Vytvoř soubor `app/Models/Search/Web/YourModel.php`:

```php
<?php

declare(strict_types=1);

namespace App\Models\Search\Web;

use App\Models\YourModel as ParentYourModel;
use App\Models\YourModelTranslation;
use Frame\Illuminate\Database\Eloquent\DefaultSearchIndexSetting;
use Frame\Illuminate\Database\Eloquent\EshopSearchable;
use Frame\Serializers\GenericSerializer;
use Illuminate\Database\Eloquent\Builder;
use JeroenG\Explorer\Application\Explored;
use JeroenG\Explorer\Application\IndexSettings;
use Laravel\Scout\Searchable;

final class YourModel extends ParentYourModel implements
    Explored,
    IndexSettings
{
    use EshopSearchable, Searchable {
        EshopSearchable::searchableAs insteadof Searchable;
    }
    use DefaultSearchIndexSetting;

    protected $table = 'your_models';

    protected $translationModel = YourModelTranslation::class;

    /**
     * Customize the query used when making all models searchable.
     */
    protected function makeAllSearchableUsing(Builder $query): Builder
    {
        return $query->where('is_searchable', true)
            ->whereNull('deleted_at')
            ->where('published', true)
            ->with([
                'translations',
                'image',
                'image.translations',
            ]);
    }

    /**
     * Define the mapping for Elasticsearch index
     */
    public function mappableAs(): array
    {
        $config = $this->getSearchConfigs();
        
        $payload = [
            'id' => 'keyword',
            'published' => ['type' => 'boolean', 'index' => false],
            'image' => 'text',
        ];

        // Jazykově závislá pole
        foreach ($this->getLocales() as $locale) {
            $code = $locale['code'];
            $this->addLocaleMappableAs($payload, 'title', $code, $config);
            $this->addLocaleMappableAs($payload, 'perex', $code, $config);
            $this->addLocaleMappableAs($payload, 'text', $code, $config);
            $this->addLocaleMappableAs($payload, 'content', $code, $config);
            $payload['_link_' . $code] = ['type' => 'nested'];
        }

        return $payload;
    }

    /**
     * Define the data to be indexed
     */
    public function toSearchableArray(): array
    {
        $this->pushSoftDeleteMetadata();

        $payload = [
            'id' => $this->id,
            'image' => $this->image?->path ?? '',
        ] + $this->scoutMetadata()
          + $this->searchCoefficient($this->archive ?? false, $this->search_coefficient);

        // Jazykově závislá data
        foreach ($this->getLocales() as $locale) {
            $code = $locale['code'];
            $translation = $this->translate($code);
            
            $this->addLocaleSearchableAs($payload, 'title', $code, $translation);
            $this->addLocaleSearchableAs($payload, 'perex', $code, $translation);
            $this->addLocaleSearchableAs($payload, 'text', $code, $translation);
            
            // Content ze structure_content
            $payload['content_' . $code] = $translation 
                ? $this->getContentText($translation) 
                : '';
            
            // Link objekt
            $payload['_link_' . $code] = GenericSerializer::getLinkObject(
                $this, 
                $code, 
                false
            );
        }

        return $payload;
    }

    /**
     * Determine if the model should be searchable
     */
    public function shouldBeSearchable(): bool
    {
        return $this->is_searchable
            && !$this->trashed()
            && $this->published;
    }

    /**
     * Get the class name for polymorphic relations.
     * Ensures that the Search model uses the parent class name.
     */
    public function getMorphClass(): string
    {
        return ParentYourModel::class;
    }
}
```

### Krok 3: Aktualizuj parent model

V původním modelu (`App\Models\YourModel`) **NEPŘIDÁVEJ** `Explored` interface, pouze ujisti se, že má:

```php
class YourModel extends Model implements Translatable
{
    // Sloupec is_searchable
    protected $fillable = ['is_searchable', /* ... */];
    
    protected function casts(): array
    {
        return [
            'is_searchable' => 'bool',
            // ...
        ];
    }
}
```

### Krok 4: Scout příkazy pro Web model

```bash

# Vytvoření indexu

docker compose run php php artisan explorer:create "App\Models\Search\Web\YourModel"

# Import všech záznamů

docker compose run php php artisan scout:import "App\Models\Search\Web\YourModel"

# Smazání indexu

docker compose run php php artisan explorer:delete "App\Models\Search\Web\YourModel"
```

## Trait EshopSearchable

Poskytuje pomocné metody pro web implementaci:

### Metody:

- **`searchableAs()`** - vrací název indexu s prefixem z configu
- **`getLocales()`** - vrací konfigurace lokálů pro eshop
- **`getSearchConfigs()`** - vrací search nastavení z configu
- **`addLocaleMappableAs()`** - přidává jazykové pole do mappingu
- **`addLocaleSearchableAs()`** - přidává jazykové data do indexu
- **`searchCoefficient()`** - počítá vyhledávací koeficient (včetně archivu)
- **`getContentText()`** - extrahuje text ze `structure_content` podle `searchable` flagů
- **`getChannelsForSearch()`** - vrací kanály pro vyhledávání

## Automatická aktualizace indexu

Scout automaticky aktualizuje index při:
- `save()` - přidá/aktualizuje záznam
- `delete()` - odstraní záznam

Pro manuální kontrolu použij:
```php
$model->searchable(); // Přidá/aktualizuje v indexu
$model->unsearchable(); // Odstraní z indexu
```

## Vyhledávání

### Admin (přímo na modelu):

```php
use App\Models\Article;

$results = Article::search('hledaný text')->get();
```

### Web (přes search model):

```php
use App\Models\Search\Web\Product;

$results = Product::search('hledaný text')->get();
```

### Pokročilé vyhledávání s Explorer:

```php
use JeroenG\Explorer\Domain\Syntax\Matching;

$results = Product::search('*')
    ->must(new Matching('published', true))
    ->must(new Matching('title_cs', 'produkt'))
    ->get();
```

## Konfigurace

### `config/scout.php`

- Driver nastavení
- Chunk size pro import
- Soft delete behavior

### `config/explorer.php`

- Elasticsearch connection
- Default index settings
- Analyzery a tokeny
- **Seznam indexovaných modelů** (`'indexes' => []`)

### `config/va-application.php`

```php
'search' => [
    'prefix' => 'am_', // Prefix pro indexy
    'settings' => [
        'cs' => ['analyzer' => 'czech'],
        'en' => ['analyzer' => 'english'],
    ],
],
```

## Tlačítka v administraci (RoutineController)

Pro přidání tlačítka pro přegenerování indexu do administrace:

**Umístění:** `app/Http/Controllers/Admin/Routines/RoutineController.php`

### Pro Admin model:

```php
// V getRoutinesList() -> sekce "Elastic Search - vyhledávání" (ID 4) -> "Administrace" (ID 2)
[
    'name' => __('Váš Model'),
    'value' => $this->route(
        RouteName::RoutineAction->value, 
        ['action' => 'elastic', 'index' => YourModel::class], 
        false
    ),
    'confirm' => __('Opravdu chcete přegenerovat index pro váš model?'),
    'icon' => 'boxes'
],
```

### Pro Web model:

```php
// V getRoutinesList() -> sekce "Elastic Search - vyhledávání" (ID 4) -> "Web" (ID 8)
[
    'name' => __('Váš Model'),
    'value' => $this->route(
        RouteName::RoutineAction->value, 
        ['action' => 'elastic', 'index' => SearchYourModel::class], 
        false
    ),
    'confirm' => __('Opravdu chcete přegenerovat index pro váš model na webu?'),
    'icon' => 'boxes'
],
```

### Import na začátku souboru:

```php
use App\Models\YourModel;
// nebo pro web
use App\Models\Search\Web\YourModel as SearchYourModel;
```

### Dostupné ikony (FontAwesome):

- `boxes` - krabice (produkty)
- `book` - kniha (články)
- `address-book` - adresář (kontakty)
- `shopping-cart` - nákupní košík (objednávky)
- `list-ol` - seznam (tagy, kategorie)
- `warehouse` - sklad
- `file` - dokument
- `sync-alt` - synchronizace

## Checklist

### Admin pouze:

- [ ] Přidat do `config/explorer.php` (`'indexes'`)
- [ ] Přidat `Explored`, `IndexSettings` interface
- [ ] Přidat `Searchable`, `DefaultSearchIndexSetting`, `IsSearchable` traits
- [ ] Přidat sloupec `is_searchable` (migrace)
- [ ] Přidat do `$fillable` a `casts()`
- [ ] Implementovat `mappableAs()`
- [ ] Implementovat `toSearchableArray()`
- [ ] Implementovat `shouldBeSearchable()`
- [ ] Vytvořit index: `explorer:create YourModel`
- [ ] Importovat data: `scout:import "App\Models\YourModel"`
- [ ] (Volitelně) Přidat tlačítko do RoutineController

### Web pouze:

- [ ] Přidat web model do `config/explorer.php` (`'indexes'`)
- [ ] Přidat sloupec `is_searchable` do parent modelu (migrace)
- [ ] Přidat do `$fillable` a `casts()` v parent modelu
- [ ] Vytvořit `app/Models/Search/Web/YourModel.php`
- [ ] Implementovat `makeAllSearchableUsing()`
- [ ] Implementovat `mappableAs()` (s jazykovými verzemi)
- [ ] Implementovat `toSearchableArray()` (s překlady)
- [ ] Implementovat `shouldBeSearchable()`
- [ ] Implementovat `getMorphClass()`
- [ ] Vytvořit index: `explorer:create "App\Models\Search\Web\YourModel"`
- [ ] Importovat data: `scout:import "App\Models\Search\Web\YourModel"`
- [ ] (Volitelně) Přidat tlačítko do RoutineController

### Obojí (Admin + Web):

- [ ] Provést všechny kroky pro Admin
- [ ] Provést všechny kroky pro Web
- [ ] Vytvořit oba indexy samostatně
- [ ] Importovat data do obou indexů samostatně

## Troubleshooting

### Index neexistuje

```bash
docker compose run php php artisan explorer:create YourModel
```

### Data nejsou aktuální

```bash
docker compose run php php artisan scout:flush "App\Models\YourModel"
docker compose run php php artisan scout:import "App\Models\YourModel"
```

### Model se neindexuje

Zkontroluj `shouldBeSearchable()` metodu - vrací `true`?

### Zjistit co je v indexu

Použij Kibana nebo Elasticsearch API přímo.

## Důležité poznámky

1. **Překlady**: Web modely používají `EshopSearchable` trait, který automaticky řeší jazykové mutace
2. **Prefix indexu**: Web modely mají prefix z konfigurace (`am_products` místo `products`)
3. **MorphClass**: Web modely **MUSÍ** mít `getMorphClass()` aby vracely parent třídu pro polymorfní vztahy
4. **Search coefficient**: Používá se pro ranking výsledků, hodnota 0-100, s archive režimem
5. **Content extraction**: `getContentText()` automaticky extrahuje texty ze `structure_content` podle `searchable` flagů
6. **Channels**: Web modely mohou filtrovat podle kanálů (eshop, hlavní tenant, atd.)
7. **Registrace v config**: **PRVNÍ KROK** - bez registrace v `config/explorer.php` nefungují artisan příkazy!

**⚠️ Důležité:**
- **VŽDY registruj model v `config/explorer.php`** jako první krok
- **Admin modely** - interface přímo na modelu
- **Web modely** - separátní třída v `Search/Web/`
- **Web modely** - `getMorphClass()` vrací parent třídu
- **Docker** - všechny příkazy přes `docker compose run php`

Reference: [Model Structure](model-structure.md), [Model Translatable](model-translatable.md)