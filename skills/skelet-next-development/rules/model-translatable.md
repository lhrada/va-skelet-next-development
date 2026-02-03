---
title: Model Translatable - překladové modely
impact: HIGH
impactDescription: Špatná konfigurace překladů způsobuje runtime chyby
tags: model, translatable, i18n, multilanguage
---

## Model Translatable - překladové modely

**Impact: HIGH**

Pro modely s překlady použij trait `Translatable` z balíčku `astrotomic/laravel-translatable` a správně nastav `$translatedAttributes`.

**Incorrect:**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

// CHYBÍ Translatable trait
// CHYBÍ $translatedAttributes
class Product extends Model
{
    protected $fillable = [
        'name',        // ŠPATNĚ - name je v translations tabulce
        'description', // ŠPATNĚ - description je v translations tabulce
        'price',
    ];
}
```

**Correct:**

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Astrotomic\Translatable\Translatable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 * @property int $id
 * @property string $guid
 * @property float $price
 * @property int $stock
 * 
 * Translated properties
 * @property string $name
 * @property string|null $description
 * @property string|null $slug
 * 
 * @property Carbon $created_at
 * @property Carbon $updated_at
 */
final class Product extends Model
{
    use SoftDeletes;
    use Translatable;

    /**
     * The attributes that are mass assignable (hlavní tabulka).
     *
     * @var array<string>
     */
    protected $fillable = [
        'guid',
        'price',
        'stock',
        'published',
        'created_by',
        'updated_by',
    ];

    /**
     * The attributes that are translatable (translation tabulka).
     *
     * @var array<string>
     */
    public $translatedAttributes = [
        'name',
        'description',
        'slug',
        'meta_title',
        'meta_description',
        'meta_keywords',
    ];

    protected function casts(): array
    {
        return [
            'price' => 'float',
            'stock' => 'integer',
            'published' => 'boolean',
        ];
    }
}
```

**Translation model:**

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

/**
 * @property int $id
 * @property int $product_id
 * @property string $locale
 * 
 * @property string $name
 * @property string|null $description
 * @property string|null $slug
 * @property string|null $meta_title
 * @property string|null $meta_description
 * @property string|null $meta_keywords
 */
final class ProductTranslation extends Model
{
    public $timestamps = false; // Obvykle bez timestamps
    
    protected $fillable = [
        'locale',
        'name',
        'description',
        'slug',
        'meta_title',
        'meta_description',
        'meta_keywords',
    ];
}
```

**Použití v kódu:**

```php
// Vytvoření s překlady
$product = Product::create([
    'price' => 100,
    'cs_CZ' => [
        'name' => 'Produkt',
        'description' => 'Popis produktu',
    ],
    'en_US' => [
        'name' => 'Product',
        'description' => 'Product description',
    ],
]);

// Čtení překladu
$product->translate('cs_CZ')->name; // "Produkt"
$product->translate('en_US')->name; // "Product"

// Aktuální locale (z RequestHelper)
$product->name; // Automaticky z aktuálního locale
```

**Pro nested překladové atributy (JSON struktury):**

Pokud máš překladové atributy uložené v JSON sloupci (ne v translation tabulce), použij `$additionalTranslatedAttributes`:

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Astrotomic\Translatable\Translatable;
use Illuminate\Database\Eloquent\Model;

/**
 * @property int $id
 * @property array $seo
 * @property array $published
 * 
 * Translated properties (v translation table)
 * @property string $name
 * @property string|null $description
 * 
 * Additional translated properties (v JSON sloupcích)
 * SEO data (seo JSON sloupec):
 * @property string|null $seo['title']
 * @property string|null $seo['description']
 * @property string|null $seo['keywords']
 * @property string|null $seo['slug']
 * 
 * Published locales (published JSON sloupec):
 * @property array|null $published['locales']
 */
final class Product extends Model
{
    use Translatable;

    protected $fillable = [
        'guid',
        'price',
        'seo',        // JSON sloupec
        'published',  // JSON sloupec
    ];

    public $translatedAttributes = [
        'name',
        'description',
    ];

    /**
     * These attributes are translated but NOT stored in the translation table.
     * Používá se pro JSON struktury s jazykovými variantami.
     *
     * @var array<string>
     */
    protected array $additionalTranslatedAttributes = [
        'seo.title',
        'seo.description',
        'seo.keywords',
        'seo.slug',
        'published.locales',
    ];

    protected function casts(): array
    {
        return [
            'seo' => 'array',
            'published' => 'array',
        ];
    }
}
```

**Použití additionalTranslatedAttributes:**

```php
// Vytvoření s nested překladovými atributy
$product = Product::create([
    'price' => 100,
    'seo' => [
        'cs_CZ' => [
            'title' => 'SEO Titulek',
            'description' => 'SEO Popis',
            'slug' => 'produkt-slug',
        ],
        'en_US' => [
            'title' => 'SEO Title',
            'description' => 'SEO Description',
            'slug' => 'product-slug',
        ],
    ],
    'published' => [
        'locales' => [
            'cs_CZ' => true,
            'en_US' => false,
        ],
    ],
]);

// Čtení nested překladových atributů
$locale = 'cs_CZ';
$seoTitle = $product->seo[$locale]['title'] ?? null;

// Pomocí Frame\Illuminate\Database\Eloquent\TranslationTable::getLanguageDependentProperties
// můžeš získat všechny jazykově závislé properties (včetně additionalTranslatedAttributes)
```

**Kdy použít `$additionalTranslatedAttributes`:**
- Máš JSON sloupec s jazykovými variantami (např. `seo`, `published`, `metadata`)
- Nechceš vytvářet translation tabulku pro tyto atributy
- Potřebuješ flexibilní strukturu dat s jazykovými variantami
- Používáš `Frame\Illuminate\Database\Eloquent\TranslationTable` trait

**Rozdíl mezi `$translatedAttributes` a `$additionalTranslatedAttributes`:**

| Vlastnost | `$translatedAttributes` | `$additionalTranslatedAttributes` |
|-----------|-------------------------|-----------------------------------|
| Uložení | V translation tabulce | V JSON sloupci hlavní tabulky |
| Formát | Flat hodnoty | Nested JSON struktura |
| Příklad | `name`, `description` | `seo.title`, `published.locales` |
| Automatické načítání | Přes Translatable trait | Přes TranslationTable trait |

**Důležité:**
- Překladové atributy NEJSOU v `$fillable`, ale v `$translatedAttributes`
- Translation model obvykle BEZ timestamps (`public $timestamps = false`)
- Translation model má `$fillable` obsahující locale + překladové atributy
- PHPDoc v hlavním modelu obsahuje překladové properties (označené komentářem "Translated properties")

Reference: [Migration Translations](migration-translations.md)
