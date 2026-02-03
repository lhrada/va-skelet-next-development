# Plan: Skill – Vývoj projektu ze Skeletu Next

## Cíl
Vytvoříme skill s dokumentací pro vývojáře, kteří budou **vytvářet nové projekty ze Skeletu Next**. Skelet Next je **boilerplate/starter template** obsahující základní moduly (eshop, obsahová část, administrace, autentizace), které vývojář přizpůsobí a rozšíří o nové moduly na míru konkrétnímu zákazníkovi. Skill bude obsahovat references na existující instrukce, pravidla kódování, architekturu, databázi, routery a oprávnění – bez kopírování kódu, jen s linky na aktuální strukturu.

## Klíčové principy
- **No code duplication** – Skill není kopie existujících instrukcí, ale jejich organizovaný index
- **References instead of copy-paste** – Všechny detailní pravidla se linkují na existující `.github/instructions/`
- **Just-in-time guidance** – Skill aktivuje se, když vývojář potřebuje help s novým projektem ze Skeletu
- **Practical workflow** – Zahrnout docker příkazy, artisan commands, běžné use cases

## Membership: Klíčové domény

### 1. **Basics** (Boilerplate, technologie, setup)
   - **Skelet Next**: Boilerplate/starter template pro Laravel API projekty
   - **Zahrnuté moduly**: Eshop, obsahová část (články, galerie, SEO), administrace, autentizace, role a oprávnění
   - **Tech stack**: Laravel 12, PHP 8.4+, MariaDB 10.11, Docker
   - **Key packages**: Sanctum (auth), spatie/laravel-permission (roles), scout (search), translatable
   - **Docker setup**: Název kontejneru a služeb viz `.github/copilot-instructions.md`
   - **Základní commands**: `docker exec -it {container-name} bash`, `php artisan`, `composer`, `npm`
   - **Workflow**: Klonovat Skelet → Přizpůsobit základní moduly → Přidat nové moduly na míru zákazníkovi

### 2. **Architecture** (Kódová struktura)
   - **Controllers**: `App\Http\Controllers\{Admin|Public|...}\{Entity}Controller`
   - **Services**: `Domain\{Entity}\{Entity}Service` → `staffMethod()`, `clientMethod()`
   - **Models**: `App\Models\{Entity}` + PHPDoc @property anotace
   - **Validation**: `App\Http\Controllers\{Admin|Public}\{Entity}Validation` → `createOrReplace()`, `update()`
   - **Resources**: `App\Http\Controllers\{Admin|Public}\{Entity}Resource` + `ListResource` (camelCase output)
   - **Traits & Helpers**: `Frame\`, `Domain\` custom namespaces
   - Link: [Podrobné instrukce v router.instructions.md](.github/instructions/router.instructions.md)

### 3. **Database** (Migrační struktura, modely)
   - **Migrations** – `database/migrations/` s rollback logicou
   - **Models** – `app/Models/`, Eloquent relationships, casts, scopes
   - **Translations** – Tabulky s příponou `_translations`, balíček `astrotomic/laravel-translatable`
   - **Soft deletes** – `SoftDeletes` trait u archivovatelných entit
   - **JSON validace** – `JSON_VALID()` constraints v MariDB
   - Pravidla: Při úpravě sloupce v migraci musíš uvést všechny atributy (jinak se smaž!)
   - Link: [Detailní databázová struktura](../../instructions/database.instructions.md)

### 4. **Routing** (REST struktura, middleware, hierarchie)
   - **Sekce rout**: Sessions, OAuth, Wallet, Account, Admin, Self, Kiosk, Node, Public, Enums, Links, Deliveries
   - **Middleware**: `auth:sanctum`, `auth.optional:sanctum`, `tenant`, `localization`, `throttle`, `cache`
   - **Route prefix aliasy**: `AdminArticleController`, `PublicArticleController`, `KioskArticleController`, ...
   - **REST CRUD**: `index`, `show`, `store`, `patch`, `put`, `delete`, `restore`
   - **Actions**: `/actions` (batch), `/{entity}/actions` (specific)
   - **Throttling**: `throttle:contact-form`, `throttle:user-set-password`
   - Link: [Kompletní router instrukce](../../instructions/router.instructions.md)

### 5. **Permissions & Authorization** (Enums, policies, role management)
   - **Permission enum**: `Frame\Permissions\Permission` – PascalCase case names
   - **Permission values**: `entity.action.scope` (kebab-case, max 125 chars)
   - **Naming patterns**: `{Entity}{Action}` → `ArticleCreate`, `ArticleViewAny`, `ArticleUpdateOwn`, ...
   - **API Mappings**: `apiMappings()` → lidsky čitelné `__('Entity - akce')` v češtině
   - **Policies**: `app/Policies/{Entity}Policy.php` s `authorize()` logikou
   - **Workflow**: 1. Přidej case do enum → 2. `php artisan app:create-permissions` → 3. Doplň `apiMappings()` → 4. Vytvoř/update Policy
   - Link: [Pravidla pro oprávnění](../../instructions/permission.instructions.md)

### 6. **API Documentation** (Markdown + OpenAPI)
   - **Structure**: Admin `doc/api/admin/modules/{entity}.md`, Public `doc/api/public/modules/{entity}.md`
   - **OpenAPI spec**: `doc/api/openapi/{entity}.openapi.yml`
   - **Key rules**: 
     - Data pouze z kódu (validators, resources, models) – nesmíš si vymýšlet!
     - Přesné názvy polí (ne `seoTitle` ale `seo.title`, ne `street` ale `address1`/`address2`/`houseNumber`)
     - Query/Body params tabulka: `Parametr | Typ | Povinnost | Validace | Popis | Enum endpoint`
     - Oprávnění s přesným názvem (např. `landmark.view.any`)
     - Response struktura s `meta` a `links`
   - **GenericSerializer reference**: `published()`, `seo()`, `toDateTime()` – macros pro recursos
   - Link: [Detailní dokumentační instrukce](../../instructions/doc.instructions.md)

### 7. **Testing** (PHPUnit, Feature & Unit)
   - **Framework**: PHPUnit v11 (ne Pest!)
   - **Test types**: Feature tests (výchozí), Unit tests (`--unit` flag)
   - **Vytvoření**: `php artisan make:test FeatureName` → `php artisan make:test --unit UnitName`
   - **Factories & Seeders**: Vždy vytvořit pro nové modely
   - **Run tests**: 
     - Vše: `php artisan test --compact`
     - Soubor: `php artisan test --compact tests/Feature/ExampleTest.php`
     - Filter: `php artisan test --compact --filter=testName`
   - **Coverage**: Všechny happy paths, failure paths, edge cases
   - **Faker**: `$this->faker->word()` nebo `fake()->randomDigit()`

### 8. **Code Style & PHP 8.4 Features** (Modern PHP best practices)
   - **Strict types**: `declare(strict_types=1);` na začátku každého PHP souboru
   - **Return types**: Povinné – `public function getName(): string`
   - **Nullsafe operator**: `$user?->profile?->name`
   - **Null coalescing**: `$value ?? 'default'`
   - **match expression**: Místo `switch` pro elegantní větvení
   - **readonly properties**: Immutable pole v třídách
   - **enum**: Typované výčty místo konstant
   - **final classes**: Jako výchozí, pokud nemají subclass
   - **Constructor property promotion**: `public function __construct(public User $user) { }`
   - **PHPDoc blocks**: Místo inline komentářů, including array shapes
   - Link: Laravel Boost guidelines section "PHP"

### 9. **Docker & Local Development**
   - **Container**: Název a konfigurace viz `.github/copilot-instructions.md`
   - **Commands**: `docker exec -it {container-name} bash` - konkrétní název z hlavních instrukcí
   - **Services**: PHP + Redis + Mailpit (email testing)
   - **Startup**: `docker-compose up -d`, `docker-compose up -d --build`
   - **Env setup**: `.env` + `php artisan key:generate`
   - **Note**: Sensitive values v `.env` jsou encrypted via SOPS

### 10. **Integration & MCP Tools**
   - **Context7**: Documentation search pro libraries
   - **Laravel Boost**: `search-docs`, `tinker`, `database-query`, `list-routes`, `list-artisan-commands`, `browser-logs`, `last-error`
   - **httpProxy**: API calls na jsonplaceholder, uam, gs
   - **API-Server**: Custom API s dokumentací a hledáním

### 11. **Hoppscotch Collections** (API Testing)
   - **Účel**: JSON kolekce pro testování API endpointů v Hoppscotch
   - **Kdy vytvořit**: Po vytvoření nového controlleru (volitelné - zeptat se uživatele)
   - **Formát**: Hoppscotch v10 JSON format
   - **Umístění**: 
     - Admin: `doc/hoppscotch/admin/{ModelName}.json`
     - Public: `doc/hoppscotch/public/{ModelName}.json`
     - Společné: `doc/hoppscotch/{ModelName}.json`
   - **Požadované requesty**: index (GET), show (GET), store (POST), update PUT, update PATCH, destroy (DELETE), actions (POST)
   - **Headers**: X-Data-Lang, X-Lang, X-Channel, Accept, Content-Type, Authorization (Bearer), X-Cache-Control
   - **Environment variables**: `<<baseUri>>`, `<<path>>`, `<<authToken>>`, `<<locale>>`, `<<dataLocale>>`, `<<X-Channel>>`
   - **Test scripts**: Základní status code checks (200, 201)
   - Link: [Pravidla pro Hoppscotch kolekce](../../prompts/hoppscotch.prompt.md)

### 12. **Workflow Prompts** (Generovací šablony)
   - **Controller prompt**: Vytvoření controller + service + validation + resources + policy
     - Podporuje Admin i Public controllery
     - Vygeneruje všechny REST metody (index, show, store, update, destroy, restore, actions)
     - Integruje služby, validaci, resources a policies
   - **Migration prompt**: Vytvoření databázových migrací s rollback
     - Správné datové typy pro MariaDB
     - JSON validation constraints
     - Foreign keys a indexy
   - **Elastic prompt**: Nastavení Elasticsearch indexů a mapping
     - Scout konfigurace
     - Searchable trait pro modely
     - Mapping definice
   - **Enum controller prompt**: Vytvoření enum controlleru pro číselníky
     - Speciální struktura pro enumy/číselníky
     - EnumResource formát
   - **Hoppscotch prompt**: Generování testovacích kolekcí
     - Kompletní REST kolekce pro Hoppscotch v10
     - Environment variables setup
     - Test scripts pro validaci
   - Umístění: `.github/prompts/*.prompt.md`
   - Link: [Controller prompt](../../prompts/controller.prompt.md) | [Migration prompt](../../prompts/migration.prompt.md) | [Hoppscotch prompt](../../prompts/hoppscotch.prompt.md)

## Skill Metadata

```yaml
name: skelet-next-development
description: >-
  Vývoj nových projektů ze Skeletu Next boilerplate. Aktivuje se při vytváření nových modulů,
  entit, routin, oprávnění a API dokumentace z boilerplate template. Skelet Next obsahuje
  základní moduly (eshop, obsah, admin), které vývojář přizpůsobí a rozšíří o nové moduly
  na míru zákazníkovi. Sjednocuje instrukce z .github, architektonické rozhodnutí a Laravel 12 best practices.
applyTo:
  - app/**
  - domain/**
  - routes/**
  - database/migrations/**
  - doc/api/**
trigger:
  keywords:
    - skelet next
    - nový projekt ze skeletu
    - boilerplate
    - starter template
    - vytvořit modul
    - nový controller
    - nová entita
    - přizpůsobit skelet
```

## Activation Triggers

Skill se aktivuje, když vývojář:
- Začíná nový projekt ze Skeletu Next boilerplate
- Přizpůsobuje základní moduly (eshop, obsah) zákazníkovi
- Vytváří nový modul/kontroler/service pro konkrétního klienta
- Potřebuje guidance na architektuře API dle Skelet konvencí
- Pracuje na databázových migracích
- Tvoří oprávnění a policies
- Píše API dokumentaci
- Pracuje na testování
- Potřebuje informaci o project structure a standardech Skeletu

## File Structure (v skill adresáři)

```
.github/skills/skelet-next-development/
├── SKILL.md                    # Hlavní skill dokumentace
├── sections/
│   ├── basics.md              # Projekt + tech stack
│   ├── architecture.md        # Controllers, Services, Models
│   ├── database.md            # Migrační struktura, pravidla
│   ├── routing.md             # REST, middleware, hierarchie
│   ├── permissions.md         # Enums, policies, roles
│   ├── documentation.md       # API docs guidelines
│   ├── testing.md             # PHPUnit, workflow
│   ├── code-style.md          # PHP 8.4 features, conventions
│   ├── docker.md              # Development environment
│   ├── hoppscotch.md          # API testing collections
│   └── workflow-prompts.md    # Generovací šablony
└── examples/                  # Praktické code snippets (reference jen, ne copy-paste)
    ├── controller-example.php
    ├── service-example.php
    ├── migration-example.php
    ├── test-example.php
    └── hoppscotch-collection.json
```

## Skill Struktura (SKILL.md)

1. **Frontmatter** – Metadata (name, description, trigger keywords)
2. **When to Apply** – Kdy aktivovat skill
3. **Quick Start** – Základní workflow (docker, artisan commands)
4. **Core Sections** – Reference na .github/instructions/* + stručné přehledy
5. **Common Workflows** – Typické tasky při práci se Skeletem:
   - Vytvoření nového controlleru + validation + resource pro zákazníka
   - Přizpůsobení existujících modulů (eshop, články) specifickým potřebám
   - Přidání oprávnění a policy
   - Vytvoření migrace + modelu + factory
   - Psaní API dokumentace
   - Psaní testů
   - Generování Hoppscotch kolekce pro testování API
   - Použití workflow prompts z `.github/prompts/`
6. **Tools & Resources** – Které MCP tools použít pro co
7. **Best Practices** – Checklisty, pitfalls, quick tips

## Výstupní struktura

Po implementaci budou všechny instrukce dostupné přes `run_subagent(agentName='skelet-next-development-skill', task='...')` a skill se bude automaticky aktivovat dle trigger keywords.

---

## Next Steps (po schválení plánu)

1. Vytvořit `.github/skills/skelet-next-development/SKILL.md` s hlavní strukturou
2. Vytvořit subsekce v `sections/` (volitelně – mohou být i vloženy do SKILL.md)
3. Připravit praktické příklady v `examples/`
4. Zaregistrovat skill v `boost.json` (přidat `"skelet-next-development"` do `skills` pole)
5. Testovat aktivaci skill na praktických use cases

---

## Úvahy a open questions

1. **Scope v1**: Zaměříme se čistě na backend API (PHP/Laravel/DB), nebo zahrnout i Docker/frontend setup?
   - **Návrh**: V1 = backend + docker basics, frontend může být separátní skill později

2. **Příklady vs. reference**: Skill má obsahovat jen linky na instrukce, nebo i inline code snippets?
   - **Návrh**: Hybrid – linky + malé inline snippets (5-10 lines) jako ilustrace, vždy s referencí "viz full code v X.php"

3. **Frontend skill**: Existuje v projektu frontend (Next.js)? Měli bychom separátní skill pro frontend vývoj?
   - **Návrh**: Oddělený skill `skelet-next-frontend-development`, aby se nesešel s backend skillom

4. **Interdependencies**: Skill by měl odkazovat na context7 documentaci pro Laravel 12, nebo jen na projekt-specifické `.github` instrukce?
   - **Návrh**: Primárně projekt-specifické instrukce, context7 jen jako fallback pro "jak na to v Laravel 12 obecně"

5. **Update cadence**: Jak budeme skill udržovat aktuální, když se instrukce v `.github/instructions/` změní?
   - **Návrh**: Skill hlavně linkuje, takže změny v `.github/instructions/` se automaticky projeví. Periodic review 1x za quarter.
