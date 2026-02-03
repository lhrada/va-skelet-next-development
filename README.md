# Skelet Next Development

Komplexní skill balíček pro vývoj projektů ze **Skelet Next** - základního API projektu pro řešení e-commerce a správy obsahu v Laravel 12.

## O čem se jedná?

Tento skill obsahuje **44 podrobných pravidel a best practices** pro vývoj nových projektů a modulů. Pokrývá všechny aspekty vývoje: architekturu, code style, validaci, oprávnění, databázi, modely, services, testy a dokumentaci.

## Struktura

- **`SKILL.md`** - Hlavní přehled všech pravidel a referencí
- **`plan-skeletNextDevelopment.prompt.md`** - AI prompt pro plánování vývoje
- **`rules/`** - 44 detailních skill souborů s konkrétními pravidly

## Kdy se používá?

Aktivuj tento skill když:

- 📦 Začínáš nový projekt ze Skelet Next
- 🏗️ Vytváříš nový modul, kontroler nebo service
- 💾 Pracuješ na databázových migracích
- 🔐 Tvoříš oprávnění a policies
- 📖 Píšeš API dokumentaci
- ✅ Pracuješ na testování

## Hlavní kategorie pravidel

### 1. Architektura & Code Style
- Struktura projektů a modulů
- PHP 8.4+ best practices (strict types, readonly, enums, match)
- PHPDoc komentáře a type hints

### 2. Autentizace & Oprávnění
- Laravel Sanctum integrace
- Spatie/laravel-permission role a oprávnění
- Policy struktura a autorizace

### 3. API Vrstvy
- Controllers s REST metodami
- Validation třídy pro validaci vstupů
- Resources pro formátování výstupů

### 4. Databáze
- Migrace s správným naming a konvencí
- Foreign keys a indexy
- JSON validace v MariaDB
- Překladové tabulky (astrotomic/laravel-translatable)

### 5. Business Logika
- Service vrstva s business logikou
- Modely s Eloquent vztahy
- Activity log pro audit trail

### 6. Vyhledávání
- Laravel Scout integrace
- Elasticsearch (Explorer) pro full-text search

### 7. Testování
- PHPUnit testy
- Factories a seeders
- Feature a unit testy

### 8. Dokumentace
- API dokumentace
- Hoppscotch kolekce pro testování

## Použití

1. Otevři `SKILL.md` pro přehled všech pravidel
2. Vyberi relevantní pravidla dle kategorie
3. Přečti si podrobné návody v souborech `rules/`
4. Aplikuj pravidla ve svém kódu

## Příklad

Pokud vytváříš nový kontroler:
1. Podívej se na `rules/controller-structure.md`
2. Podívej se na `rules/validation-structure.md`
3. Podívej se na `rules/resource-structure.md`
4. Podívej se na `rules/policy-structure.md`

## Tech Stack

- **PHP** 8.4+
- **Laravel** 12 (upgradováno z v10)
- **MariaDB** 10.11
- **Docker** (kontejnerizace)

Speciální balíčky:
- `laravel/sanctum` - Autentizace
- `spatie/laravel-permission` - Role a oprávnění
- `astrotomic/laravel-translatable` - Vícejazyčnost
- `laravel/scout` + `jeroen-g/explorer` - Vyhledávání (Elasticsearch)

---

**Podrobnosti:** Viz [`SKILL.md`](./SKILL.md) a adresář [`rules/`](./rules/)
