---
title: Překladové tabulky struktura
impact: HIGH
impactDescription: Špatná struktura překladových tabulek znemožňuje vícejazyčnost
tags: migration, translations, i18n, multilanguage
---

## Překladové tabulky struktura

**Impact: HIGH**

Překladové tabulky musí mít standardizovanou strukturu s foreign key na hlavní tabulku, locale sloupcem a unique indexem.

**Incorrect:**

```php
Schema::create('product_translations', function (Blueprint $table) {
    $table->id();
    $table->integer('product_id'); // Chybí foreign key constraint
    $table->string('lang'); // Špatný název sloupce (mělo by být locale)
    $table->string('title');
    // CHYBÍ unique index na product_id + locale
    // CHYBÍ created_by, updated_by, soft deletes
});
```

**Correct:**

```php
Schema::create('product_translations', function (Blueprint $table) {
    $table->id();
    
    // Foreign key na hlavní tabulku
    $table->foreignId('product_id')
        ->constrained('products')
        ->cascadeOnUpdate()
        ->cascadeOnDelete();
    
    // Locale sloupec - VŽDY s utf8_general_ci collation
    $table->string('locale', 64)->index()
        ->collation('utf8_general_ci');
    
    // Překladové sloupce
    $table->string('name', 1024)->nullable()->default(null);
    $table->text('perex')->nullable()->default(null)
        ->comment('Excerpt for lists');
    $table->longText('description')->nullable()->default(null)
        ->comment('Main description');
    
    // SEO sloupce (volitelné)
    $table->string('slug', 1024)->nullable()->default(null)
        ->comment('SEO - url slug');
    $table->string('meta_title', 1024)->nullable()->default(null)
        ->comment('SEO - meta title');
    $table->string('meta_description', 1024)->nullable()->default(null)
        ->comment('SEO - meta description');
    $table->string('meta_keywords', 1024)->nullable()->default(null)
        ->comment('SEO - meta keywords');
    
    // User tracking
    $table->foreignId('created_by')->nullable()->default(null)
        ->constrained('users')
        ->nullOnDelete()
        ->cascadeOnUpdate();
    $table->foreignId('updated_by')->nullable()->default(null)
        ->constrained('users')
        ->nullOnDelete()
        ->cascadeOnUpdate();
    
    $table->softDeletes();
    $table->timestamps();
    
    // DŮLEŽITÉ: Unikátní kombinace product_id + locale
    $table->unique(['product_id', 'locale']);
});
```

**Model konfigurace:**

```php
use Astrotomic\Translatable\Translatable;

class Product extends Model
{
    use Translatable;
    
    public $translatedAttributes = [
        'name',
        'perex',
        'description',
        'slug',
        'meta_title',
        'meta_description',
        'meta_keywords',
    ];
}
```

**Povinné prvky:**
1. Foreign key na hlavní tabulku s `cascadeOnDelete()` a `cascadeOnUpdate()`
2. `locale` sloupec (64 chars) s indexem
3. Unique index na `[{table}_id, locale]`
4. `created_by`, `updated_by` tracking
5. `softDeletes()`, `timestamps()`

**Název tabulky:**
- Formát: `{table}_translations` (množné číslo + _translations)
- Příklad: `products` → `product_translations`
- Příklad: `articles` → `article_translations`

Reference: [Database Instructions](../../instructions/database.instructions.md)
