---
title: Service upsert metoda s ModelHelper
impact: HIGH
impactDescription: Správná upsert metoda zajišťuje konzistentní vytváření a aktualizaci
tags: service, upsert, modelhelper, transaction, uuid
---

## Service upsert metoda s ModelHelper

**Impact: HIGH**

Metoda `upsert()` slouží pro vytvoření nebo aktualizaci modelu. Používej DB transaction, Uuid::create() a ModelHelper pro hydrataci.

**Correct - Základní upsert:**

```php
public function upsert(
    string $locale,
    ?Product $product,
    array $data,
): Product {
    return DB::transaction(function () use ($locale, $product, $data) {
        // Nový model
        if ($product === null) {
            $product = $this->product->newInstance();
            $product->guid = Uuid::create();  // VŽDY Uuid::create()
            $product->created_by = $data['created_by'] ?? null;
        } else {
            // Existující model
            $product->updated_by = $data['updated_by'] ?? null;
        }

        // Hydratace pomocí ModelHelper
        $properties = [
            'name',
            'description',
            'price',
            'stock',
            'sku',
            'sequence',
            'priority',
            'protected',
            'published',
            'note',
        ];

        ModelHelper::hydrateModel($product, $data, $properties);

        $product->save();

        return $product->refresh();
    });
}
```

**Correct - S překlady (Translatable):**

```php
public function upsert(
    string $locale,
    ?Article $article,
    array $data
): Article {
    return DB::transaction(function () use ($locale, $article, $data) {
        if ($article === null) {
            $article = $this->article->newInstance();
            $article->guid = Uuid::create();
            $article->created_by = $data['created_by'] ?? null;
        } else {
            $article->updated_by = $data['updated_by'] ?? null;
        }
        $userId = $data['created_by'] ?? $data['updated_by'] ?? null;

        // SEO data (volitelné)
        $seo = $data['seo'] ?? [];
        unset($data['seo']);

        // System administration properties (protected, key, plugins)
        $isSystemAdministration = (bool) Auth()->user()?->hasPermissionTo(Permission::Root);
        if ($isSystemAdministration) {
            ModelHelper::hydrateCommonAdminProps($article, $data);
        }

        // Překlad - properties v translation tabulce
        $translationProperties = [
            'title',
            'shortTitle',
            'perex',
            'text',
            'publicText',
            'structureContent',
        ];
        
        /** @var ArticleTranslation $translation */
        $translation = $article->translateOrNew($locale);
        ModelHelper::hydrateModel($translation, $data, $translationProperties);
        
        // SEO v translation tabulce
        ModelHelper::hydrateSeo($translation, $seo);

        // Hlavní tabulka - properties
        $properties = [
            'sequence',
            'priority',
            'note',
            'public',
            'isSearchable',
            'searchCoefficient',
            'oldId',
            'oldIds',
            'importCode',
            'structure',
            'contentLayout',
        ];

        ModelHelper::hydrateModel($article, $data, $properties);
        
        // Main documents (mainImage, mainVideo, mainDocument)
        ModelHelper::hydrateMainDocuments($article, $data);

        // Publikování
        if (isset($data['published'])) {
            $this->setPublished($data['published'], $article, $locale);
        }

        try {
            $article->save();
        } catch (QueryException $exception) {
            $errorCode = $exception->errorInfo[1] ?? null;
            if ($errorCode === 1062) {
                throw new UnprocessableContent($exception->getMessage())
                    ->convey(
                        __('Záznam s klíčem ":key" již existuje', ['key' => $data['key']])
                    );
            }
            throw $exception;
        }

        // Many-to-many vztahy (tagy, labels, sections)
        if (isset($data['labels'])) {
            $article->labels()->syncWithPivotValues(
                $data['labels'],
                ['as' => TagObjectAs::Label]
            );
        }
        if (isset($data['sections'])) {
            $article->sections()->syncWithPivotValues(
                $data['sections'],
                ['as' => TagObjectAs::Section]
            );
        }

        // Galerie
        if (array_key_exists('galleries', $data)) {
            ModelHelper::addGalleries(
                model: $article,
                galleries: $data['galleries'] ?? [],
                locale: $locale,
                user: $userId
            );
        }

        return $article->refresh();
    });
}
```

**ModelHelper metody pro upsert:**

### hydrateModel()

Naplní model daty z pole. Automaticky převádí camelCase na snake_case.

```php
// Jednoduché properties
$properties = ['name', 'price', 'stock'];
ModelHelper::hydrateModel($product, $data, $properties);

// S mapováním data → model
$properties = [
    'name',                    // name → name
    'firstName',               // firstName → first_name (auto snake_case)
    'branch' => 'branch_id',   // branch → branch_id (custom mapping)
];
ModelHelper::hydrateModel($model, $data, $properties);
```

### hydrateCommonAdminProps()

Naplní společné admin properties (protected, key, plugins).

```php
$isSystemAdministration = (bool) Auth()->user()?->hasPermissionTo(Permission::Root);
if ($isSystemAdministration) {
    ModelHelper::hydrateCommonAdminProps($model, $data);
}
```

### hydrateSeo()

Naplní SEO pole (title, slug, description, keywords).

```php
$seo = $data['seo'] ?? [];
ModelHelper::hydrateSeo($translation, $seo);
```

### hydrateMainDocuments()

Naplní hlavní dokumenty (mainImage, mainVideo, mainDocument).

```php
ModelHelper::hydrateMainDocuments($model, $data);
```

### addGalleries()

Přidá galerie k modelu.

```php
ModelHelper::addGalleries(
    model: $article,
    galleries: $data['galleries'] ?? [],
    locale: $locale,
    user: $userId
);
```

**⚠️ Důležité:**
- **VŽDY použij DB::transaction()** - zajišťuje atomicitu
- **Uuid::create()** místo Str::uuid()->toString()
- **ModelHelper::hydrateModel()** pro properties - automatický snake_case
- **$model->refresh()** na konci - načte fresh data včetně relationships
- **QueryException catch** pro duplicate key (1062)
- **syncWithPivotValues()** pro many-to-many s pivot daty

Reference: [Service Structure](service-structure.md), [Service Builder](service-builder.md)
