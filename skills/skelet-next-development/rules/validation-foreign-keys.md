---
title: Vazby BEZ suffixu Id
impact: HIGH
impactDescription: Špatné pojmenování způsobí validační chyby
tags: validation, foreign-keys, naming
---

## Vazby BEZ suffixu Id

**Impact: HIGH**

V validačních pravidlech pro foreign keys **NIKDY** nepoužívej suffix `Id`. Validační pole odpovídá názvu vztahu v Eloquent modelu, **NE** názvu sloupce v databázi.

**Incorrect:**

```php
public function createOrReplace(Request $request): array
{
    return $request->validate([
        'userId' => ['required', 'integer', 'exists:users,id'],        // ŠPATNĚ!
        'categoryId' => ['required', 'integer', 'exists:categories,id'], // ŠPATNĚ!
        'imageId' => ['nullable', $common->imageDocumentExists()],   // ŠPATNĚ!
    ]);
}
```

**Correct:**

```php
public function createOrReplace(Request $request): array
{
    $common = new Common();
    
    return $request->validate([
        'user' => ['required', 'integer', 'exists:users,id'],          // ✓ Správně
        'category' => ['required', 'integer', 'exists:categories,id'], // ✓ Správně
        'image' => ['nullable', $common->imageDocumentExists()],   // ✓ Správně
    ]);
}
```

**Proč:**
- Request data přicházejí bez `Id` suffixu: `{"user": 1, "category": 5}`
- Eloquent vztahy jsou pojmenovány bez suffixu: `$product->user`, `$product->image`
- Konzistence napříč celou aplikací

**V databázových sloupcích:**
- Sloupec v DB: `user_id`, `category_id`, `image_id`
- Validace: `user`, `category`, `mainImage`
- Eloquent vztah: `user()`, `category()`, `image()`

Reference: [Controller REST Methods](controller-rest-methods.md), [Validation Structure](validation-structure.md)
