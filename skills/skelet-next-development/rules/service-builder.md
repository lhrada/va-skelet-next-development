---
title: Service builder metody s EloquentQueryTool
impact: HIGH
impactDescription: Builder metody zajišťují správné filtrování a stránkování
tags: service, builder, eloquentquerytool, dictionary, pagination
---

## Service builder metody s EloquentQueryTool

**Impact: HIGH**

Service má dvě klíčové metody:
- **`staffBuilder()`** - vytvoří builder, aplikuje `decorate()` s $mapper a $relationMapper
- **`staffSource()`** - zavolá staffBuilder() a vrátí Paginator pomocí `getPaginate()`

**EloquentQueryTool metody:**
- **`decorate()`** - aplikuje QueryObject (filtry/řazení) na builder
- **`getPaginate()`** - vytvoří Paginator z builderu (aplikuje QueryObject + stránkování)

## Kompletní příklad (ProductService)

```php
public function staffBuilder(
    string $locale,
    ?QueryObject $queryObject = null,
    bool $trash = false,
): Builder {
    if ($queryObject === null) {
        $queryObject = new QueryObject();
    }
    
    // Mapper - mapování sloupců pro WHERE a ORDER BY
    $mapper = [
        'id' => $this->product->qualifyColumn('id'),
        'deleted' => $this->product->getQualifiedDeletedAtColumn(),
        'published' => $this->product->qualifyColumn('published'),
        'key' => $this->product->qualifyColumn('key'),
        
        // Více sloupců = OR hledání
        'hasImage' => [
            'variants.image_id',
            'products.image_id',
        ],
        'order' => [
            'products.sequence',
            'products.priority',
        ],
    ];
    
    // RelationMapper - mapování vztahů
    $relationMapper = [
        // Jeden sloupec ve vztahu
        'categoryName' => new RelationMapper(
            relation: 'category',
            column: 'categories.name',
        ),
        
        // Více sloupců (OR hledání v title NEBO short_title)
        'title' => new RelationMapper(
            relation: 'translations',
            column: [
                $this->productTranslation->qualifyColumn('title'),
                $this->productTranslation->qualifyColumn('short_title'),
            ]
        ),
        
        // Kódy - aliasované sloupce (nelze qualifyColumn)
        'codes' => new RelationMapper(
            relation: 'variants',
            column: ['code', 'sku', 'gtin']
        ),
        
        // Email přes vztah
        'email' => new RelationMapper(
            relation: 'createdBy',
            column: $this->userModel->qualifyColumn('email')
        ),
    ];

    // Základní builder s eager loading
    $builder = $this->product->newQuery()
        ->with(['category', 'image', 'createdBy', 'updatedBy']);

    // Trash podmínka
    ModelHelper::addTrashCondition($queryObject, $this->product, $trash);

    // decorate() aplikuje QueryObject na builder
    // Nový styl (PHP 8.4+) - BEZ uzávorkování
    new EloquentQueryTool()->decorate(
        builder: $builder,
        queryObject: $queryObject,
        locale: $locale,
        dictionary: $mapper,
        relationMapper: $relationMapper,
    );

    return $builder;
}

public function staffSource(
    string $locale,
    ?QueryObject $queryObject = null,
    bool $trash = false,
): Paginator {
    // Zavolá staffBuilder() pro získání builderu
    $builder = $this->staffBuilder($locale, $queryObject, $trash);

    // getPaginate() vytvoří Paginator
    // Nový styl (PHP 8.4+) - BEZ uzávorkování
    return new EloquentQueryTool()->getPaginate(
        builder: $builder,
        queryObject: $queryObject,
        columns: [$this->product->qualifyColumn('*')],
    );
}
```

## Starý styl (před PHP 8.4)

Jediný rozdíl je **uzávorkování**:

```php
// Starý styl - S uzávorkováním
(new EloquentQueryTool())->decorate(...);
(new EloquentQueryTool())->getPaginate(...);

// Nový styl (PHP 8.4+) - BEZ uzávorkování
new EloquentQueryTool()->decorate(...);
new EloquentQueryTool()->getPaginate(...);
```

## Více sloupců (OR logika)

Když použiješ **array** místo string, hledá se v NĚKTERÉM ze sloupců (OR logika):

```php
// Mapper - hledá image_id v variants NEBO products
'hasImage' => [
    'variants.image_id',
    'products.image_id',
],

// RelationMapper - hledá v title NEBO short_title
'title' => new RelationMapper(
    relation: 'translations',
    column: [
        $this->productTranslation->qualifyColumn('title'),
        $this->productTranslation->qualifyColumn('short_title'),
    ]
),
```

## Dictionary (nový styl s constructor)

Místo pole můžeš použít Dictionary objekt:

```php
$mapper = new Dictionary(
    where: [
        'id' => $this->product->qualifyColumn('id'),
        'published' => $this->product->qualifyColumn('published'),
    ],
    order: [
        'sequence' => $this->product->qualifyColumn('sequence'),
        'priority' => $this->product->qualifyColumn('priority'),
    ]
);
```

## Closure v mapperu (pokročilé)

Pro složité podmínky můžeš použít closure:

```php
$mapper = [
    'price' => function ($subject, $operator, $value) use ($channelId) {
        if ($value === null) {
            throw new BadRequest("Sent null as value.");
        }
        return [sprintf('JSON_EXTRACT(prices, "$.%s")', $channelId), $operator, $value];
    },
];
```

**⚠️ Důležité:**

- **staffBuilder()** - vytvoří builder, zavolá `decorate()` s $mapper a $relationMapper
- **staffSource()** - zavolá staffBuilder(), vrátí Paginator pomocí `getPaginate()`
- **`decorate()`** - aplikuje QueryObject na builder (vrací void)
- **`getPaginate()`** - aplikuje QueryObject + vrátí Paginator
- **Nový styl (PHP 8.4+)**: `new EloquentQueryTool()` BEZ uzávorkování
- **Starý styl**: `(new EloquentQueryTool())` S uzávorkováním
- **$mapper** - mapování sloupců (přímo v staffBuilder)
- **$relationMapper** - mapování vztahů (přímo v staffBuilder)
- **Array = OR logika** - hledání v NĚKTERÉM ze sloupců
- **ModelHelper::addTrashCondition()** - vždy pro trash podporu

Reference: [Service Structure](service-structure.md), [Service Upsert](service-upsert.md)