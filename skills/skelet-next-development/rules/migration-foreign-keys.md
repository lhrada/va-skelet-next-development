---
title: Foreign Keys konvence
impact: HIGH
impactDescription: Špatně definované foreign keys způsobují runtime chyby
tags: migration, foreign-keys, relationships
---

## Foreign Keys konvence

**Impact: HIGH**

Foreign keys musí mít vždy správnou definici s komentářem, constraint akcemi a nullable hodnotou podle významu vazby.

**Incorrect:**

```php
// Chybí komentář, chybí cascade/null akce
$table->foreignId('user_id')->constrained();

// Povinná vazba s nullOnDelete - nesouhlasí!
$table->foreignId('order_id')
    ->constrained()
    ->nullOnDelete(); // Špatně - order je required!
```

**Correct:**

```php
// Nullable vazba (volitelná)
$table->foreignId('parent_id')->nullable()->default(null)
    ->comment('Parent category')
    ->constrained('categories')
    ->cascadeOnUpdate()
    ->nullOnDelete();

// Required vazba (povinná)
$table->foreignId('order_id')
    ->comment('Order reference')
    ->constrained('orders')
    ->cascadeOnUpdate()
    ->cascadeOnDelete();

// Vazba s restrict (nelze smazat pokud existují závislosti)
$table->foreignId('tag_id')
    ->comment('Product tag')
    ->constrained('tags')
    ->cascadeOnUpdate()
    ->restrictOnDelete();
```

**Pravidla:**
1. **Vždy přidej `->comment()`** - vysvětli účel vazby v angličtině
2. **`constrained()`** - bez parametru pokud sloupec je `{table}_id`, jinak `constrained('table_name')`
3. **`cascadeOnUpdate()`** - vždy! Při změně parent ID se aktualizují všechny children
4. **Delete akce podle typu vazby:**
   - `nullOnDelete()` - pro nullable (volitelné vazby)
   - `cascadeOnDelete()` - pro required (child nemůže existovat bez parent)
   - `restrictOnDelete()` - nelze smazat parent pokud existují children

**Naming:**
- Sloupec: `{related_table_singular}_id` (např. `user_id`, `product_id`)
- Pro self-reference: `parent_id`, `main_product_id`, `child_product_id`
