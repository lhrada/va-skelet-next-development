---
title: Migrace v Docker kontejneru
impact: HIGH
impactDescription: Příkazy mimo Docker nebudou fungovat
tags: migration, docker, database
---

## Migrace v Docker kontejneru

**Impact: HIGH**

Všechny příkazy pro práci s migrací MUSÍ běžet v Docker kontejneru pomocí `docker compose run php`.

**Incorrect:**

```bash

# Spuštění mimo Docker - nemusí mít přístup k DB

php artisan make:migration create_products_table
php artisan migrate
```

**Correct:**

```bash

# Vytvoření migrace

docker compose run php php artisan make:migration create_products_table

# Spuštění migrace

docker compose run php php artisan migrate

# Vytvoření modelu s migrací

docker compose run php php artisan make:model Product -m

# Rollback

docker compose run php php artisan migrate:rollback

# Reset databáze + seed

docker compose run php php artisan migrate:fresh --seed
```

**Proč:**
- Aplikace běží v Docker kontejneru
- Pouze kontejner má přístup k databázi
- Zajišťuje konzistentní prostředí
