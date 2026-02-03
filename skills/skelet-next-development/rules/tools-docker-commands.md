---
title: Docker příkazy
impact: LOW
impactDescription: Docker příkazy pro běh aplikace a příkazů
tags: docker, docker-compose, container, php
---

## Docker příkazy

**Impact: LOW**

Všechny příkazy spouštíš v Docker containeru.

**Název containeru:** Viz `.github/copilot-instructions.md` (obvykle `skelet-api-php`)

## Základní příkazy

### Start/Stop služeb

```bash
# Spuštění služeb
docker compose up

# Spuštění služeb na pozadí
docker compose up -d

# Zastavení služeb
docker compose down

# Zastavení a odstranění orphan containers
docker compose down --remove-orphans
```

### Vstup do containeru

```bash
# Bash v PHP containeru
docker exec -it skelet-api-php bash

# Alternativa: docker compose
docker compose exec php bash
```

## Artisan příkazy

**Vždy přes docker compose run:**

```bash
# Migrace
docker compose run php php artisan migrate
docker compose run php php artisan migrate:fresh --seed
docker compose run php php artisan migrate:rollback

# Testování
docker compose run php php artisan test
docker compose run php php artisan test --compact
docker compose run php php artisan test --filter=ProductTest

# Oprávnění
docker compose run php php artisan app:create-permissions

# Scout/Elasticsearch
docker compose run php php artisan scout:import "App\Models\Product"
docker compose run php php artisan scout:flush "App\Models\Product"
docker compose run php php artisan explorer:create Product
docker compose run php php artisan explorer:delete Product

# Cache
docker compose run php php artisan cache:clear
docker compose run php php artisan config:clear
docker compose run php php artisan route:clear

# Tinker
docker compose run php php artisan tinker
```

## Scaffolding příkazy

```bash
# Controller
docker compose run php php artisan make:controller Admin/Products/ProductController

# Model s migrací, factory, seederem
docker compose run php php artisan make:model Product -mfs

# Policy
docker compose run php php artisan make:policy ProductPolicy

# Migration
docker compose run php php artisan make:migration create_products_table
docker compose run php php artisan make:migration add_published_to_products_table

# Seeder
docker compose run php php artisan make:seeder ProductSeeder

# Test
docker compose run php php artisan make:test ProductTest
docker compose run php php artisan make:test ProductTest --unit

# Generic class
docker compose run php php artisan make:class Domain/Products/ProductService
```

## Composer příkazy

```bash
# Install
docker compose run php composer install

# Update
docker compose run php composer update

# Require
docker compose run php composer require vendor/package

# Dump autoload
docker compose run php composer dump-autoload
```

## NPM příkazy

```bash
# Install
docker compose run node npm install

# Build
docker compose run node npm run build

# Dev
docker compose run node npm run dev
```

## Logs

```bash
# Sledování logů
docker compose logs -f php

# Logs konkrétní služby
docker compose logs -f mariadb
docker compose logs -f elasticsearch
```

## Databáze

```bash
# MySQL klient v containeru
docker compose exec mariadb mysql -u root -p

# Export databáze
docker compose exec mariadb mysqldump -u root -p database_name > backup.sql

# Import databáze
docker compose exec -T mariadb mysql -u root -p database_name < backup.sql
```

## docker compose vs docker exec

### `docker compose run`
- **Vytvoří nový container** pro jeden příkaz
- Použij pro: Artisan, Composer, NPM
- Po dokončení container zanikne

```bash
docker compose run php php artisan migrate
```

### `docker exec`
- **Spustí příkaz v běžícím containeru**
- Použij pro: Bash, MySQL klient, interaktivní příkazy
- Vyžaduje běžící container

```bash
docker exec -it skelet-api-php bash
```

## Běžné workflow

### Nový projekt setup

```bash
# 1. Start služeb
docker compose up -d

# 2. Install dependencies
docker compose run php composer install

# 3. Migrace
docker compose run php php artisan migrate --seed

# 4. Oprávnění
docker compose run php php artisan app:create-permissions

# 5. Cache clear
docker compose run php php artisan config:clear
```

### Development workflow

```bash
# 1. Vstup do containeru
docker exec -it skelet-api-php bash

# 2. V containeru můžeš spouštět příkazy bez docker compose run
php artisan migrate
php artisan test
composer dump-autoload
```

### Testing workflow

```bash
# Unit testy
docker compose run php php artisan test --testsuite=Unit

# Feature testy
docker compose run php php artisan test --testsuite=Feature

# Konkrétní test
docker compose run php php artisan test --filter=testCanCreateProduct

# S coverage (pokud máš Xdebug)
docker compose run php php artisan test --coverage
```

## Troubleshooting

### Container neběží

```bash
# Zkontroluj stav
docker compose ps

# Restart služby
docker compose restart php

# Rebuild image
docker compose build php
docker compose up -d
```

### Permission denied

```bash
# Oprávnění pro storage a cache
docker compose run php chmod -R 777 storage bootstrap/cache
```

### Port already in use

```bash
# Zjisti co používá port
sudo lsof -i :8000

# Změň port v docker-compose.yml nebo zastav konkurující službu
```

## Užitečné aliasy

Pro `.zshrc` nebo `.bashrc`:

```bash
# Artisan
alias da='docker compose run php php artisan'

# Composer
alias dc='docker compose run php composer'

# Test
alias dt='docker compose run php php artisan test --compact'

# Příklady použití:
# da migrate
# dc install
# dt --filter=ProductTest
```

**⚠️ Důležité:**
- **Vždy docker compose run** pro Artisan a Composer
- **docker exec** pro interaktivní shell
- **Název containeru** z `.github/copilot-instructions.md`
- **Service name** je `php`, ne název containeru

Reference: [Migration Docker Commands](migration-docker-commands.md), [Artisan Scaffolding](tools-artisan-scaffolding.md)
