---
title: Architektura - Oddělování zájmů (Separation of Concerns)
impact: CRITICAL
impactDescription: Základní architektura určuje čistotu kódu a správné rozdělení zodpovědností
tags: architecture, separation-of-concerns, layers, controller, service, validation, resource, policy
---
## Architektura - Oddělování zájmů (Separation of Concerns)

**Impact: CRITICAL**
**Klíčový princip:** Každá vrstva má svou zodpovědnost. Controller orchestruje, Service implementuje logiku, Validation validuje, Resource transformuje, Policy autorizuje.

## Vrstvy a jejich zodpovědnosti

### 1. Controller - Orchestrace

**Zodpovědnost:** Přijetí requestu, orchestrace toku, vrácení response.

**Co PATŘÍ do Controlleru:**
- ✅ Přijetí requestu
- ✅ Volání Validation pro validaci
- ✅ Volání Policy pro autorizaci
- ✅ Volání Service pro business logiku
- ✅ Vrácení Resource (transformace výstupu)
**Co NEPATŘÍ do Controlleru:**
- ❌ Business logika (max 2-3 řádky)
- ❌ Přímá manipulace s modelem
- ❌ Komplexní SQL queries
- ❌ Transformace dat

**Princip:** Pokud máš více než 2-3 řádky logiky → přesuň do Service!
**Příklad:**
```php
// ✅ SPRÁVNĚ
public function store(ProductValidation $validation): JsonResponse
{
    $product = $this->productService->createProduct(
        $validation->createOrReplace()->validated()
    );
    return response()->json(new ProductResource($product), 201);
}
```
---
### 2. Service - Business logika

**Zodpovědnost:** Implementace veškeré business logiky aplikace.
**Co PATŘÍ do Service:**
- ✅ Vytváření, čtení, aktualizace, mazání (CRUD)
- ✅ Komplexní logika (transactions, event triggering)
- ✅ Volání dalších services
- ✅ Volání modelů a databáze
- ✅ Event triggering
**Co NEPATŘÍ do Service:**
- ❌ HTTP response
- ❌ Request validace (to dělá Validation)
- ❌ Autorizace (to dělá Policy)
- ❌ Output transformace (to dělá Resource)
**Princip:** Service vrací Model (ne Response), je volán z Controlleru s již validovanými daty.
**Příklad:**
```php
// ✅ SPRÁVNĚ
public function createProduct(array $data): Product
{
    $product = Product::create($data);
    $product->categories()->attach($data['categories']);
    $product->tags()->sync($data['tags']);
    event(new ProductCreated($product));
    return $product;
}
```
---
### 3. Validation - Vstupní validace

**Zodpovědnost:** Validace vstupních dat z requestu.
**Co PATŘÍ do Validation:**
- ✅ Validační pravidla (`required`, `numeric`, `exists`, atd.)
- ✅ Dvě metody: `createOrReplace()` (POST) a `update()` (PATCH)
- ✅ Custom rules
- ✅ Vrácení array s pravidly
**Co NEPATŘÍ do Validation:**
- ❌ Business logika
- ❌ Transformace dat
- ❌ HTTP response
**Princip:** Validation třídy jsou jen pravidla, volají se v Controlleru.
**Příklad:**
```php
// ✅ SPRÁVNĚ
public function createOrReplace(): array
{
    return [
        'title' => 'required|string|max:255',
        'price' => 'required|numeric|min:0',
        'categories' => 'required|array',
        'categories.*' => 'exists:categories,id',
    ];
}
public function update(): array
{
    return [
        'title' => 'sometimes|string|max:255',
        'price' => 'sometimes|numeric|min:0',
    ];
}
```
---
### 4. Resource - Output transformace

**Zodpovědnost:** Transformace dat z modelu do JSON response.
**Co PATŘÍ do Resource:**
- ✅ Mapování polí (snake_case → camelCase)
- ✅ Transformace typů (datetime → ISO 8601)
- ✅ Vnořené resources
- ✅ Formatování dat
**Co NEPATŘÍ do Resource:**
- ❌ Business logika
- ❌ Manipulace s modelem
- ❌ Database queries
**Princip:** Resource vrací array, je volán z Controlleru při návratu odpovědi.
**Příklad:**
```php
// ✅ SPRÁVNĚ
public function toArray($request): array
{
    return [
        'id' => $this->id,
        'title' => $this->title,
        'price' => $this->price,
        'createdAt' => $this->created_at->toIso8601String(),
        'categories' => CategoryResource::collection($this->categories),
    ];
}
```
---
### 5. Policy - Autorizace

**Zodpovědnost:** Určení, **kdo má právo** provádět akce.
**Co PATŘÍ do Policy:**
- ✅ Kontrola oprávnění (`user->can('permission')`)
- ✅ Vrácení boolean (true/false)
- ✅ Metody: `view()`, `create()`, `update()`, `delete()`, atd.
**Co NEPATŘÍ do Policy:**
- ❌ Business logika
- ❌ HTTP response
- ❌ Komplexní database queries
**Princip:** Policy odpovídá na otázku: "Má uživatel právo provést tuto akci?"
**Příklad:**
```php
// ✅ SPRÁVNĚ
public function update(User $user, Product $product): bool
{
    return $user->can('product.update.any')
        || ($user->can('product.update.own') && $user->id === $product->owner_id);
}
```
---
## Schéma toku - Request → Response

```
┌─────────────────────────────────────────┐
│ CLIENT REQUEST                          │
│ POST /api/products                      │
│ { title, price, categories }            │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ CONTROLLER (Orchestration)              │
│ - Přijetí requestu                      │
│ - Volání Validation                     │
│ - Volání Policy (authorize)             │
│ - Volání Service                        │
│ - Vrácení Resource                      │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ VALIDATION (Input validation)           │
│ - Validace vstupů                       │
│ - Vrácení validated array               │
│ - Bez transformace!                     │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ POLICY (Authorization)                  │
│ - Check "Má uživatel právo?"           │
│ - Throw Forbidden pokud nemá            │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ SERVICE (Business logic)                │
│ - Vytvoření produktu                    │
│ - Attach kategorií                      │
│ - Sync tagů                             │
│ - Trigger event                         │
│ - Vrácení Model                         │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ MODEL (Database)                        │
│ - Save do DB                            │
│ - Manage relationships                  │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ RESOURCE (Output transformation)        │
│ - Transform snake_case → camelCase      │
│ - Format datumů (ISO 8601)              │
│ - Vnořené resources                     │
│ - Vrácení JSON                          │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ CLIENT RESPONSE                         │
│ 201 Created                             │
│ { id, title, price, categories, ... }   │
└─────────────────────────────────────────┘
```
---
## Pravidla oddělování zájmů

### 1. Žádná logika v Controlleru

```php
// ❌ ŠPATNĚ
public function store(Request $request)
{
    $data = $request->validate([...]);
    $product = Product::create($data);
    $product->categories()->attach($data['categories']);  // ❌ Logika v controlleru
    return new ProductResource($product);
}
// ✅ SPRÁVNĚ
public function store(ProductValidation $validation): JsonResponse
{
    $product = $this->service->createProduct($validation->createOrReplace()->validated());
    return response()->json(new ProductResource($product), 201);
}
```
### 2. Service vrací Model, ne Response
```php
// ❌ ŠPATNĚ
public function createProduct(array $data)
{
    $product = Product::create($data);
    return response()->json([...]);  // ❌ HTTP response v service
}
// ✅ SPRÁVNĚ
public function createProduct(array $data): Product
{
    $product = Product::create($data);
    return $product;  // ✅ Vrátí Model
}
```
### 3. Validation vrací pravidla, ne transformuje data

```php
// ❌ ŠPATNĚ
public function createOrReplace()
{
    $data = request()->all();
    $data['slug'] = Str::slug($data['title']);  // ❌ Transformace v Validation
    return $data;
}

// ✅ SPRÁVNĚ - Pravidla jako pole, ne string
public function createOrReplace(): array
{
    return [
        'title' => ['required', 'string', 'max:255'],
        'price' => ['required', 'numeric', 'min:0'],
        // Transformace se dělá v Service!
    ];
}
```
### 4. Resource jen transformuje, nemodifikuje

```php
// ❌ ŠPATNĚ
public function toArray($request): array
{
    $this->resource->update(['viewed' => true]);  // ❌ Modifikace v Resource
    return [...];
}
// ✅ SPRÁVNĚ
public function toArray($request): array
{
    return [
        'id' => $this->id,
        'title' => $this->title,
        // Jen transformace!
    ];
}
```
---
## Checklist - Správné oddělení zodpovědností

- [ ] **Controller** - Max 2-3 řádky, orchestrace toku
- [ ] **Service** - Všechna business logika, vrací Model
- [ ] **Validation** - Jen pravidla, dva formáty (POST/PATCH)
- [ ] **Resource** - Jen transformace (camelCase, datetime ISO)
- [ ] **Policy** - Jen autorizace, vrací boolean
- [ ] **Žádná logika mimo Service**
- [ ] **Žádné database queries v Controlleru**
- [ ] **Žádný HTTP response v Service**
---

## Reference - Související pravidla

- [architecture-controller-structure.md](architecture-controller-structure.md) - Specifické pravidla pro Controller
- [service-structure.md](service-structure.md) - Service vrstva
- [validation-structure.md](validation-structure.md) - Validation třídy
- [resource-structure.md](resource-structure.md) - Resource transformace
- [policy-structure.md](policy-structure.md) - Policy autorizace
