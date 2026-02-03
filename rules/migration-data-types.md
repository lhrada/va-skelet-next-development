title: Běžné datové typy a sloupce
impact: MEDIUM
impactDescription: Správné datové typy zajišťují konzistenci a výkon
tags: migration, data-types, columns, decimal, price
---
## Běžné datové typy a sloupce
**Impact: MEDIUM**
Používej správné datové typy pro různé druhy dat. Zejména pro ceny, měny a čísla dodržuj standardní precision.
**Incorrect:**
```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    // ŠPATNĚ - float pro ceny (zaokrouhlovací chyby)
    $table->float('price');
    // ŠPATNĚ - malý decimal pro ceny
    $table->decimal('discount', 8, 2);
    // ŠPATNĚ - velký string pro měnu
    $table->string('currency', 255);
    // ŠPATNĚ - chybí collation pro měnu
    $table->string('currency', 3);
});
```
**Correct:**
```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    // Ceny - decimal(24, 14) pro vysokou přesnost
    $table->decimal('price', 24, 14)->nullable()->default(null)
        ->comment('Price of item');
    $table->decimal('original_price', 24, 14)->nullable()->default(null)
        ->comment('Original price before discount');
    $table->decimal('discount_price', 24, 14)->nullable()->default(null)
        ->comment('Discounted price');
    // DPH - decimal(5, 2) pro procenta
    $table->decimal('vat_value', 5, 2)->nullable()->default(null)
        ->comment('VAT percentage (e.g., 21.00)');
    // Koeficienty - decimal(10, 4)
    $table->decimal('price_coefficient', 10, 4)->default(1.0)
        ->comment('Coefficient for adjusting the base price');
    // Měna - string(3) s utf8_general_ci
    $table->string('currency', 3)->nullable()->default(null)
        ->collation('utf8_general_ci')
        ->comment('Currency code (CZK, EUR, USD, ...)');
    $table->timestamps();
});
```
**Běžné datové typy:**
### Stringy
```php
// Email - velký string kvůli dlouhým emailům
$table->string('email', 2048)->nullable()->default(null)
    ->comment('Email address');
// Locale - s utf8_general_ci collation
$table->string('locale', 64)
    ->collation('utf8_general_ci')
    ->comment('Locale code - cs_CZ, en_US, de_DE, ...');
// SKU, EAN - s utf8_general_ci collation
$table->string('sku', 64)->unique()
    ->collation('utf8_general_ci')
    ->comment('Stock Keeping Unit');
// Text pro krátký obsah
$table->text('perex')->nullable()->default(null)
    ->comment('Short excerpt for lists');
// LongText pro dlouhý obsah
$table->longText('description')->nullable()->default(null)
    ->comment('Full product description');
```
### Čísla
```php
// Integer pro počty
$table->integer('stock')->default(0)
    ->comment('Stock quantity');
$table->integer('quantity')->default(0)
    ->comment('Quantity of items');
// BigInteger pro externí ID
$table->bigInteger('external_id')->nullable()->default(null)
    ->comment('External system ID');
// TinyInteger pro malá čísla (0-255)
$table->tinyInteger('status')->default(0)
    ->comment('Status code');
```
### Boolean
```php
// Boolean vždy s default hodnotou
$table->boolean('is_active')->default(false)
    ->comment('Is record active');
$table->boolean('has_stock')->nullable()->default(null)
    ->comment('Has stock available (null = unknown)');
$table->boolean('in_offer')->default(true)->index()
    ->comment('The product is currently on offer');
```
### Datumy a časy
```php
// DateTime pro přesné datum a čas
$table->dateTime('published_at')->index()->nullable()->default(null)
    ->comment('Publication date and time');
// Timestamp pro události
$table->timestamp('event_at')->nullable()->default(null)
    ->comment('Date of event');
// Date pro datum bez času
$table->date('birth_date')->nullable()->default(null)
    ->comment('Birth date');
$table->date('valid_from')->nullable()->default(null)
    ->comment('Valid from date');
// Time pro čas bez data
$table->time('delivery_time_from')->nullable()->default(null)
    ->comment('Delivery time from');
$table->time('delivery_time_to')->nullable()->default(null)
    ->comment('Delivery time to');
```

### Geografické souřadnice

```php
// GPS souřadnice - decimal(10, 6)
$table->decimal('longitude', 10, 6)->nullable()->default(null)
    ->comment('GPS coordinates of the warehouse - longitude');
$table->decimal('latitude', 10, 6)->nullable()->default(null)
    ->comment('GPS coordinates of the warehouse - latitude');

// JTSK souřadnice (S-JTSK = Czech coordinate system) - decimal(20, 6)
$table->decimal('stationing', 20, 6)->nullable()->default(null)
    ->comment('Stationing/position on the axis');
$table->decimal('coordinate_x', 20, 6)
    ->comment('X coordinate');
$table->decimal('coordinate_y', 20, 6)
    ->comment('Y coordinate');
$table->decimal('coordinate_z', 20, 6)->nullable()->default(null)
    ->comment('Z coordinate (elevation)');
```

**Precision pro decimals:**

| Použití | Precision | Příklad |
|---------|-----------|---------|
| Ceny | `decimal(24, 14)` | `price`, `discount_price`, `total` |
| DPH procenta | `decimal(5, 2)` | `vat_value` (21.00) |
| Koeficienty | `decimal(10, 4)` | `price_coefficient` (1.2500) |
| Množství s přesností | `decimal(10, 3)` | `weight_kg` (15.250) |
| GPS souřadnice | `decimal(10, 6)` | `longitude`, `latitude` |
| JTSK souřadnice | `decimal(20, 6)` | `coordinate_x`, `coordinate_y`, `coordinate_z`, `stationing` |

**⚠️ Důležité:**
- **NIKDY nepoužívej float pro ceny** - má zaokrouhlovací chyby
- **Vždy decimal(24, 14) pro ceny** - zajišťuje vysokou přesnost
- **Měny s utf8_general_ci collation** - jsou to identifikátory (CZK, EUR)
- **Boolean vždy s default** - explicitně true nebo false

Reference: [Migration Required Columns](migration-required-columns.md), [Migration Collation](migration-collation.md)
