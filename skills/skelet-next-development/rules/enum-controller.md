---
title: Enum Controller - výčtové seznamy pro API
impact: MEDIUM
impactDescription: EnumController poskytuje výčtové seznamy pro select komponenty
tags: enum, controller, select, dropdown, pairs
---

## Enum Controller - výčtové seznamy pro API

**Impact: MEDIUM**

`EnumController` poskytuje výčtové hodnoty (enums) a páry klíč-hodnota pro select/dropdown komponenty v administraci i veřejné části API.

**Controller:** `App\Http\Controllers\EnumController`  
**Route prefix:** `/api/enums/`  
**Middleware:** `auth.optional:sanctum` (route) + `auth:sanctum` (controller s výjimkami)

## Typy endpointů

### 1. `-keys` - Výčet klíčů z enumů

Statická data z enum tříd s jejich názvy a hodnotami.

```php
// V __invoke() metodě:
'article-keys' => $this->response(
    $this->enumPairs(
        apiMappings: ArticleKey::apiMappings(),
        withObject: ArticleKey::class,
        plugins: ArticleKey::plugins(),
    )
),

'tag-keys' => $this->response(
    $this->enumPairs(TagKey::apiMappings(), TagKey::class)
),

'param-keys' => $this->response(
    $this->enumPairs(ParamKey::apiMappings())
),
```

**Výstup:**
```json
[
    {
        "name": "Název klíče",
        "value": "key_value",
        "enumObject": {
            "id": [1, 2]
        }
    }
]
```

### 2. `-pairs` - Páry z databáze

Dynamická data z databáze ve formátu pro select komponenty.

**A) Jednoduchá implementace (bez ElasticSearch):**

```php
private function branches(Request $request): array
{
    $dataLocale = RequestHelper::getDataLocale($request);
    
    $only = ['id', 'title', 'key'];
    $qo = new FilterBuilder()
        ->only(...$only)
        ->makeQueryObject($request);
    
    $items = $this->branchService->find(
        locale: $dataLocale,
        queryObject: $qo,
        trash: false,
    );
    
    $meta = self::getMetadata($items);
    
    return [BranchEnumResource::collection($items), $meta];
}

// V __invoke():
'branch-pairs' => $this->response(...$this->branches($request)),
```

**B) S ElasticSearch (Scout):**

```php
private function products(Request $request): array
{
    $dataLocale = RequestHelper::getDataLocale($request);
    $term = RequestHelper::searchTerm($request);
    
    $only = ['id', 'title', 'sku', 'code'];
    $queryObject = new FilterBuilder()
        ->only(...$only)
        ->makeQueryObject($request);
    
    // Scout - pokud je zadán search term
    if ($term) {
        $search = Product::search($term)->withTrashed();
        
        $limit = $queryObject->getOffset() + 1;
        $items = $search->take($limit)
            ->paginate(
                perPage: $queryObject->getLimit(),
                pageName: Helpers::PageName,
                page: $queryObject->getPage()
            );
        
        $queryObject->addCondition('id', QueryObject::OP_IN, $search->get()->pluck('id'));
    } else {
        $items = $this->productService->find(
            locale: $dataLocale,
            queryObject: $queryObject,
            trash: false,
        );
    }
    
    $meta = self::getMetadata($items);
    
    return [ProductEnumResource::collection($items), $meta];
}

// V __invoke():
'product-pairs' => $this->response(...$this->products($request)),
```

**C) S MultiMatch query (pokročilé):**

```php
if ($term) {
    $fields = [
        'title^96',      // vyšší váha pro název
        'code^48',       // střední váha pro kód
        'shortTitle',    // standardní váha
    ];
    $multiMatchQuery = new MultiMatch(
        value: $term,
        fields: $fields,
        fuzziness: false,
    );
    
    $search = Product::search($term)
        ->withTrashed()
        ->must($multiMatchQuery);
    
    // Další podmínky
    if ($tenant?->channel?->id) {
        $search->must(new Nested('channels', new Matching('channels.id', $tenant->channel->id)));
    }
    
    // ... zbytek stejný
}
```

**Výstup:**
```json
{
    "data": [
        {
            "id": 123,
            "value": 123,
            "name": "Název položky"
        }
    ],
    "meta": {
        "hasMorePages": true,
        "isEmpty": false,
        "total": 150
    }
}
```

### 3. `-types` - Typy z enumů

Statická data z enum tříd (typy produktů, dokumentů apod.).

```php
'product-types' => $this->response(
    $this->enumPairs(ProductType::apiMappings())
),

'document-types' => $this->response(
    DocumentType::apiMappings()
),

'material-types' => $this->response(
    $this->enumPairs(MaterialType::apiMappings())
),
```

## Struktura __invoke() metody

```php
public function __invoke(Request $request, string $type): Responsable
{
    $result = match ($type) {
        // -keys (výčty z enumů)
        'article-keys' => $this->response(
            $this->enumPairs(ArticleKey::apiMappings(), ArticleKey::class)
        ),
        'tag-keys' => $this->response(
            $this->enumPairs(TagKey::apiMappings(), TagKey::class)
        ),
        
        // -pairs (data z DB)
        'branch-pairs' => $this->response(...$this->branches($request)),
        'product-pairs' => $this->response(...$this->products($request)),
        
        // -types (typy z enumů)
        'product-types' => $this->response(
            $this->enumPairs(ProductType::apiMappings())
        ),
        
        default => (throw new BadRequest('Unknown type.'))
            ->convey(__('Neznámý typ výčtu'))
    };
    
    return $result;
}
```

## Pomocné metody

### response()

```php
private function response($data, ?array $meta = null): Responsable
{
    return Responder::ok([
        'data' => $data,
        'meta' => $meta,
    ]);
}
```

### getMetadata()

```php
private static function getMetadata(Paginator|Collection $items): array
{
    if ($items instanceof Paginator) {
        return [
            'hasMorePages' => $items->hasMorePages(),
            'isEmpty' => $items->isEmpty(),
            'total' => $items->total(),
        ];
    }
    
    return [
        'hasMorePages' => false,
        'isEmpty' => $items->isEmpty(),
        'total' => $items->count(),
    ];
}
```

### enumPairs()

```php
private function enumPairs(
    array $apiMappings,
    ?string $withObject = null,
    ?array $plugins = null
): array
{
    // Filtruje podle aktivních pluginů
    // Volitelně propojuje na DB objekty
    // Vrací [{name, value, enumObject}]
}
```

## RequestHelper metody

```php
$dataLocale = RequestHelper::getDataLocale($request);
$term = RequestHelper::searchTerm($request);
$withoutSets = RequestHelper::withoutSets($request);
$currency = RequestHelper::currency($request);
$priceListId = RequestHelper::getPriceList($request);
$warehouse = RequestHelper::getWarehouse($request);
$channel = RequestHelper::getChannel($request);
$tenant = app(IsTenant::class)->current();
$trash = RequestHelper::showTrash($request);
$asTree = RequestHelper::asTree($request);
```

## Resource třídy

Pro `-pairs` endpointy používej dedikované `*EnumResource`:

```php
ArticleEnumResource
BranchEnumResource
ProductEnumResource
LandmarkEnumResource
MaterialEnumResource
TagEnumResource
MessageEnumResource
OrderEnumResource
InvoiceEnumResource
ProfileEnumResource
UserEnumResource
```

## Přidání nového endpointu

### 1. Pro `-keys` (enum výčet)

```php
// V __invoke():
'my-new-keys' => $this->response(
    $this->enumPairs(MyNewKey::apiMappings(), MyNewKey::class)
),
```

### 2. Pro `-pairs` (data z DB, jednoduchý)

```php
// Přidej private metodu:
private function myNews(Request $request): array
{
    $dataLocale = RequestHelper::getDataLocale($request);
    
    $only = ['id', 'title'];
    $qo = new FilterBuilder()
        ->only(...$only)
        ->makeQueryObject($request);
    
    $items = $this->myNewService->find(
        locale: $dataLocale,
        queryObject: $qo,
        trash: false,
    );
    
    $meta = self::getMetadata($items);
    
    return [MyNewEnumResource::collection($items), $meta];
}

// V __invoke():
'my-new-pairs' => $this->response(...$this->myNews($request)),
```

### 3. Pro `-pairs` (s ElasticSearch)

```php
private function myNews(Request $request): array
{
    $dataLocale = RequestHelper::getDataLocale($request);
    $term = RequestHelper::searchTerm($request);
    
    $only = ['id', 'title'];
    $queryObject = new FilterBuilder()
        ->only(...$only)
        ->makeQueryObject($request);
    
    if ($term) {
        $search = MyNew::search($term)->withTrashed();
        
        $limit = $queryObject->getOffset() + 1;
        $items = $search->take($limit)
            ->paginate(
                perPage: $queryObject->getLimit(),
                pageName: Helpers::PageName,
                page: $queryObject->getPage()
            );
        
        $queryObject->addCondition('id', QueryObject::OP_IN, $search->get()->pluck('id'));
    } else {
        $items = $this->myNewService->find(
            locale: $dataLocale,
            queryObject: $queryObject,
            trash: false,
        );
    }
    
    $meta = self::getMetadata($items);
    
    return [MyNewEnumResource::collection($items), $meta];
}
```

### 4. Pro `-types` (enum typy)

```php
// V __invoke():
'my-new-types' => $this->response(
    $this->enumPairs(MyNewType::apiMappings())
),
```

## Veřejné endpointy (bez autentizace)

Pro veřejné endpointy vytvoř dedikovanou metodu:

```php
public function publicMyNews(Request $request): Responsable
{
    $dataLocale = RequestHelper::getDataLocale($request);
    
    $only = ['id', 'title'];
    $qo = new FilterBuilder()
        ->only(...$only)
        ->makeQueryObject($request);
    
    $items = $this->myNewService->clientSource(
        locale: $dataLocale,
        queryObject: $qo,
    );
    
    $meta = self::getMetadata($items);
    
    return $this->response(
        MyNewEnumResource::collection($items),
        $meta
    );
}
```

### Registrace v routeru

Veřejné endpointy se registrují v `routes/api.php`:

```php
// V routes/api.php
Route::prefix('/enums')
    ->middleware(['auth.optional:sanctum'])
    ->group(function () {
        // Veřejné dedikované endpointy
        Route::get('/app-locales', [EnumController::class, 'publicAppLocales']);
        Route::get('/public-tags', [EnumController::class, 'publicTags']);
        Route::get('/name-days/{day}/{month}', [EnumController::class, 'publicNameDays'])
            ->where([
                'day' => RouteParamType::Day,
                'month' => RouteParamType::Month,
            ]);
        
        // Ostatní endpointy přes __invoke() (vyžadují autentizaci)
        Route::get('/{type}', [EnumController::class, '__invoke'])
            ->middleware('auth:sanctum');
    });
```

**📘 Viz kompletní seznam:** `routes/api.php` - sekce `/enums`

### V EnumController - middleware výjimky

V konstruktoru controlleru definuj výjimky z auth middleware:

```php
public function __construct(/* ... services ... */) {
    // Auth middleware s výjimkami pro veřejné metody
    $this->middleware('auth:sanctum')
        ->except([
            'publicAppLocales',
            'publicDataLocales',
            'publicTags',
            // ... další veřejné metody
        ]);
}
```

**📘 Viz kompletní seznam výjimek:** `App\Http\Controllers\EnumController::__construct()`

### Příklady veřejných metod

**Jednoduché seznamy (statická data):**

```php
public function publicAppLocales(Request $request): Responsable
{
    $locales = Locale::getConfig();
    
    $data = collect($locales)->map(function ($locale) {
        return [
            'code' => $locale['code'],
            'name' => $locale['name'],
            'flag' => $locale['flag'],
        ];
    })->values()->all();
    
    return $this->response($data);
}

public function publicCountries(Request $request): Responsable
{
    $countries = Country::apiMappings();
    
    return $this->response(
        $this->enumPairs($countries)
    );
}
```

**Seznamy z databáze (dynamická data):**

```php
public function publicTags(Request $request): Responsable
{
    $dataLocale = RequestHelper::getDataLocale($request);
    $type = $request->input('type'); // TagType
    
    $only = ['id', 'name', 'key'];
    $qo = new FilterBuilder()
        ->only(...$only)
        ->makeQueryObject($request);
    
    // Přidání typu filtru
    if ($type) {
        $qo->addCondition('type', QueryObject::OP_EQUAL, $type);
    }
    
    // Jen publikované
    $qo->addCondition('published', QueryObject::OP_EQUAL, true);
    
    $items = $this->tagService->clientSource(
        locale: $dataLocale,
        queryObject: $qo,
    );
    
    $meta = self::getMetadata($items);
    
    return $this->response(
        TagEnumResource::collection($items),
        $meta
    );
}
```

**Hierarchické seznamy (tree struktura):**

```php
public function publicTreeCategories(Request $request, string $type): Responsable
{
    $dataLocale = RequestHelper::getDataLocale($request);
    
    $items = $this->tagService->getTree(
        locale: $dataLocale,
        type: TagType::from($type),
        published: true, // Jen publikované
    );
    
    return $this->response(
        TagEnumResource::collection($items)
    );
}
```

### Rozdíly mezi veřejnými a autentizovanými endpointy

| Aspekt | Veřejné | Autentizované |
|--------|---------|---------------|
| **Registrace** | Dedikovaná route | Přes `__invoke()` |
| **Middleware** | `auth.optional:sanctum` | `auth:sanctum` |
| **Metoda v controlleru** | `public{Name}()` | V `match()` v `__invoke()` |
| **Data** | Jen publikované | Včetně nepublikovaných, smazaných |
| **Service** | `clientSource()` | `staffSource()` nebo `find()` |

**⚠️ Důležité:**
- **`__invoke()`** pro většinu endpointů
- **`-keys`** pro enum výčty (statická data)
- **`-pairs`** pro DB data (dynamická, s EnumResource)
- **`-types`** pro enum typy (statická)
- **ElasticSearch** pro fulltextové hledání
- **RequestHelper** pro získání parametrů
- **EnumResource** pro transformaci dat
- **getMetadata()** pro pagination info

Reference: [Controller Structure](controller-structure.md), [Resource Structure](resource-structure.md)