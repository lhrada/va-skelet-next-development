---
title: Pojmenování migrací
impact: MEDIUM
impactDescription: Špatné pojmenování ztěžuje orientaci v projektu
tags: migration, naming, conventions
---

## Pojmenování migrací

**Impact: MEDIUM**

Názvy migrací musí být v **anglickém jazyce** a **množném čísle**. Používej popisné názvy, které jasně vyjadřují účel migrace.

**Incorrect:**

```bash
# České názvy
docker compose run php php artisan make:migration vytvor_produkty_tabulku

# Jednotné číslo
docker compose run php php artisan make:migration create_product_table

# Nepopisný název
docker compose run php php artisan make:migration update_table
```

**Correct:**

```bash
# Vytvoření tabulky - create_{table}_table
docker compose run php php artisan make:migration create_products_table
docker compose run php php artisan make:migration create_orders_table
docker compose run php php artisan make:migration create_product_translations_table

# Přidání sloupců - add_{column}_to_{table}_table
docker compose run php php artisan make:migration add_priority_to_products_table
docker compose run php php artisan make:migration add_published_columns_to_articles_table

# Změna sloupců - change_{column}_in_{table}_table
docker compose run php php artisan make:migration change_price_in_products_table
docker compose run php php artisan make:migration change_slug_type_in_articles_table

# Odstranění sloupců - remove_{column}_from_{table}_table
docker compose run php php artisan make:migration remove_old_field_from_products_table

# Vytvoření indexů - add_{index}_index_to_{table}_table
docker compose run php php artisan make:migration add_published_index_to_articles_table
```

**Konvence:**
- **Vytvoření tabulky**: `create_{tables}_table` (množné číslo)
- **Přidání sloupců**: `add_{columns}_to_{tables}_table`
- **Změna sloupců**: `change_{column}_in_{tables}_table`
- **Odstranění sloupců**: `remove_{column}_from_{tables}_table`
- **Vytvoření indexů**: `add_{index}_index_to_{tables}_table`
- **Překladové tabulky**: `create_{table}_translations_table`
- **Pivot tabulky**: `create_{table1}_{table2}_table`

**Proč:**
- Laravel convention
- Snadná orientace v seznamu migrací
- Jasné vyjádření účelu změny
- Automatické generování tříd

Reference: [Laravel Migrations Naming](https://laravel.com/docs/migrations#generating-migrations)
