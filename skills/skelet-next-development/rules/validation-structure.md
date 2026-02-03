---
title: Validation struktura a pravidla
impact: HIGH
impactDescription: Správná validace zajišťuje data integritu
tags: validation, rules, common, foreign-keys
---

## Validation struktura a pravidla

**Impact: HIGH**

Validation třídy jsou v namespace controlleru a mají dvě metody: `createOrReplace()` pro celý objekt a `update()` pro částečné změny.

**Namespace:**
- **Admin**: `App\Http\Controllers\Admin\{Modul}\{Název}Validation`
- **Public**: `App\Http\Controllers\Public\{Modul}\{Název}Validation`

**Incorrect:**

```php
<?php

namespace App\Http\Controllers\Admin\Products;

use Illuminate\Http\Request;

// CHYBÍ declare(strict_types=1)
// CHYBÍ final
class ProductValidation
{
    public function rules(Request $request)  // CHYBÍ return type
    {
        return [
            'userId' => 'required|exists:users,id',  // ŠPATNĚ - má být 'user', ne 'userId'
            'email' => 'unique:products,email',      // ŠPATNĚ - chybí whereNotNull() pro nullable
        ];
    }
}
```

**Correct:**

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Admin\Products;

use App\Models\Product;
use Frame\Validation\Common;
use Illuminate\Http\Request;
use Illuminate\Validation\Rule;

final class ProductValidation
{
    /**
     * Create or replace - pro POST a PUT (celý objekt)
     */
    public function createOrReplace(Request $request): array
    {
        $common = new Common();

        return $request->validate([
            'name' => ['required', 'string', 'max:255'],
            'sku' => ['required', 'string', 'max:64', $common->skuUnique()],
            'price' => ['required', 'numeric', 'min:0'],
            
            // Vazby - BEZ suffixu Id!
            'category' => ['required', 'integer', 'exists:categories,id'],
            'user' => ['nullable', 'integer', $common->userExists()],
            'image' => ['nullable', $common->imageDocumentExists()],
            
            // Unique s whereNotNull() pro nullable sloupce
            'email' => [
                'nullable',
                'email:rfc,dns',
                Rule::unique('products', 'email')->whereNotNull('email'),
            ],
            
            // Publikace pomocí Common
            ...$common->publication(required: false, prefix: 'published'),
            
            // SEO pomocí Common
            'seo' => ['sometimes', 'array'],
            ...$common->getSeoRules('seo'),
        ]);
    }

    /**
     * Update - pro PATCH (jen změněné hodnoty)
     */
    public function update(Request $request, Product $product): array
    {
        $common = new Common();

        return $request->validate([
            'name' => ['sometimes', 'string', 'max:255'],
            'sku' => ['sometimes', 'string', 'max:64', $common->skuUnique($product->id)],
            'price' => ['sometimes', 'numeric', 'min:0'],
            
            // Vazby - BEZ suffixu Id!
            'category' => ['sometimes', 'integer', 'exists:categories,id'],
            'user' => ['sometimes', 'nullable', 'integer', $common->userExists()],
            'image' => ['sometimes', 'nullable', $common->imageDocumentExists()],
            
            // Unique s ignore() pro aktuální záznam
            'email' => [
                'sometimes',
                'nullable',
                'email:rfc,dns',
                Rule::unique('products', 'email')
                    ->ignore($product->id)
                    ->whereNotNull('email'),
            ],
            
            // Publikace a SEO stejné jako v createOrReplace
            ...$common->publication(required: false, prefix: 'published'),
            'seo' => ['sometimes', 'array'],
            ...$common->getSeoRules('seo'),
        ]);
    }
}
```

## Důležitá pravidla

### 1. Vazby (Foreign Keys) BEZ suffixu Id

**VŽDY použij název vazby BEZ suffixu `Id`:**

```php
// ✅ SPRÁVNĚ
'user' => ['required', 'integer', 'exists:users,id'],
'category' => ['nullable', 'integer', 'exists:categories,id'],
'image' => ['nullable', $common->imageDocumentExists()],

// ❌ ŠPATNĚ
'userId' => ['required', 'integer', 'exists:users,id'],
'categoryId' => ['nullable', 'integer', 'exists:categories,id'],
'imageId' => ['nullable', 'exists:documents,id'],
```

### 2. Unique s whereNotNull() pro nullable sloupce

Pokud je sloupec `nullable`, VŽDY přidej `whereNotNull()`:

```php
// ✅ SPRÁVNĚ
'email' => [
    'nullable',
    'email:rfc,dns',
    Rule::unique('products', 'email')->whereNotNull('email'),
],

// ❌ ŠPATNĚ - chybí whereNotNull()
'email' => [
    'nullable',
    'email:rfc,dns',
    Rule::unique('products', 'email'),
],
```

### 3. Update s ignore() pro aktuální záznam

V `update()` metodě přidej `.ignore($model->id)`:

```php
public function update(Request $request, Product $product): array
{
    return $request->validate([
        'email' => [
            'sometimes',
            'nullable',
            Rule::unique('products', 'email')
                ->ignore($product->id)  // ✅ Ignoruj aktuální záznam
                ->whereNotNull('email'),
        ],
    ]);
}
```

## Frame\Validation\Common

Třída `Frame\Validation\Common` obsahuje **opakující se validační pravidla**.

**Dostupná pravidla:**

### Publikace

```php
$common = new Common();

// Celá publikační logika (published, from, to, locales)
...$common->publication(required: false, prefix: 'published'),

// Vygeneruje:
// 'published' => ['sometimes', 'boolean'],
// 'publishedFrom' => ['sometimes', 'nullable', 'date'],
// 'publishedTo' => ['sometimes', 'nullable', 'date', 'after:publishedFrom'],
// 'publishedLocales' => ['sometimes', 'nullable', 'array'],
```

### SEO

```php
'seo' => ['sometimes', 'array'],
...$common->getSeoRules('seo'),

// Vygeneruje:
// 'seo.title' => ['sometimes', 'nullable', 'string', 'max:255'],
// 'seo.slug' => ['sometimes', 'nullable', 'string', 'max:255'],
// 'seo.description' => ['sometimes', 'nullable', 'string', 'max:500'],
// 'seo.keywords' => ['sometimes', 'nullable', 'string', 'max:500'],
```

### Galerie

```php
...$common->galleries(required: false, documentExists: true),

// Vygeneruje validaci pro galleries array s obrázky/videi
```

### Vazby (exists)

```php
'image' => ['nullable', $common->imageDocumentExists()],
'video' => ['nullable', $common->videoDocumentExists()],
'document' => ['nullable', $common->documentExists('type')],
'user' => ['required', $common->userExists()],
'product' => ['nullable', $common->productExists()],
'variant' => ['nullable', $common->variantExists()],
'tag' => ['required', $common->tagExists(TagType::Category)],
'branch' => ['nullable', $common->branchExists()],
```

### Unique pravidla

```php
'sku' => ['required', 'string', $common->skuUnique()],
'sku' => ['sometimes', 'string', $common->skuUnique($product->id)],  // S ignore

'plu' => ['required', 'string', $common->pluUnique()],
'plu' => ['sometimes', 'string', $common->pluUnique($product->id)],  // S ignore
```

### Ostatní

```php
// Adresy
...$common->address(required: false, prefix: 'address', withZipValidation: true),

// Kontakty
...$common->contact(required: false, prefix: 'contact'),

// Časový rozsah
...$common->validateFromToRange(required: false),  // validFrom, validTo

// Prohledávání
...$common->searchable(),  // isSearchable, searchCoefficient

// Struktura obsahu
...$common->structure(required: false),  // structure, structureContent
```

## Kdy přidat nové pravidlo do Common

1. **Pravidlo se opakuje na 2+ místech** (např. validace image v Products, Articles, Variants)
2. **Pravidlo je komplexní** a má jasné využití
3. **Přidej metodu do `Frame\Validation\Common`**:

```php
public function myCustomRule(bool $required = false): Exists
{
    $rule = Rule::exists(table: 'my_table', column: 'id')
        ->where(function (Builder $query) {
            // Dodatečné podmínky
            $query->where('active', true);
        });
    
    return $rule;
}
```

4. Použij v validacích:

```php
'myField' => ['required', $common->myCustomRule(required: true)],
```

**⚠️ Důležité:**
- **Vazby BEZ suffixu Id** - `user`, `category`, `image` (NE userId, categoryId)
- **`whereNotNull()`** pro unique na nullable sloupcích
- **`.ignore($model->id)`** v update metodě
- **`Frame\Validation\Common`** pro opakující se pravidla
- **`createOrReplace()` vs `update()`** - podle HTTP metody (PUT vs PATCH)
- **`sometimes`** v update() pro volitelné parametry
- **Spread operator `...`** pro Common pravidla

Reference: [Controller REST Methods](controller-rest-methods.md), [Validation Common Rules](validation-common-rules.md)