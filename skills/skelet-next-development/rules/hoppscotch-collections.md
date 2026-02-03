---
title: Hoppscotch kolekce generování
impact: MEDIUM
impactDescription: Hoppscotch kolekce umožňují rychlé testování API endpointů
tags: hoppscotch, api-testing, collections, rest
---

## Hoppscotch kolekce generování

**Impact: MEDIUM**

Po vytvoření nového kontrolleru lze **nepovinně** vygenerovat JSON kolekci pro import do Hoppscotch, která umožní okamžité testování nových API endpointů.

**Verze:** Hoppscotch v10  
**Umístění:** `doc/hoppscotch/{admin|public}/{ModelName}.json`

## Kdy použít

Po vytvoření kontrolleru se **zeptej uživatele**:

> "Chceš, abych vygeneroval Hoppscotch kolekci pro testování těchto endpointů?"

## Formát kolekce v10

```json
{
  "v": 10,
  "id": "unique_collection_id",
  "name": "Název kolekce",
  "folders": [],
  "requests": [
    {
      "v": "15",
      "name": "Popis requestu",
      "method": "GET|POST|PUT|PATCH|DELETE",
      "endpoint": "<<baseUri>><<path>>/admin/resource",
      "params": [],
      "headers": [],
      "preRequestScript": "",
      "testScript": "pw.test(\"Status code is 200\", ()=> { pw.expect(pw.response.status).toBe(200); });",
      "auth": {
        "authType": "bearer",
        "token": "<<authToken>>",
        "authActive": false
      },
      "body": {
        "contentType": "application/json",
        "body": "{...}"
      },
      "requestVariables": [],
      "responses": {}
    }
  ],
  "auth": {
    "authType": "inherit",
    "authActive": true
  },
  "headers": [],
  "variables": [],
  "_ref_id": "coll_xxx_unique-uuid"
}
```

## Požadované requesty pro REST API

### 1. index (GET) - Seznam záznamů

**Endpoint:** `<<baseUri>><<path>>/admin/{resource}`

**Parametry** (všechny neaktivní pro ukázku):
```json
"params": [
  {
    "key": "trash",
    "value": "1",
    "active": false,
    "description": "Zobrazit smazané záznamy (1 = ano)"
  },
  {
    "key": "page[size]",
    "value": "20",
    "active": false,
    "description": "Počet záznamů na stránku (výchozí: 20)"
  },
  {
    "key": "filter[id][]",
    "value": "1",
    "active": false,
    "description": "Filtr podle ID (lze zadat více hodnot)"
  },
  {
    "key": "sort[0][field]",
    "value": "id",
    "active": false,
    "description": "Pole pro řazení (id, name, created_at, ...)"
  },
  {
    "key": "sort[0][order]",
    "value": "desc",
    "active": false,
    "description": "Směr řazení (asc, desc)"
  }
]
```

### 2. show (GET) - Detail záznamu

**Endpoint:** `<<baseUri>><<path>>/admin/{resource}/1`

### 3. store (POST) - Vytvoření nového záznamu

**Endpoint:** `<<baseUri>><<path>>/admin/{resource}`

**Body:** JSON s ukázkovými daty podle validace

```json
{
  "key": "example-key",
  "name": "Příklad názvu",
  "value": 100,
  "validSince": "2023-01-01 00:00:00",
  "validTo": null
}
```

**Test script:**
```javascript
// Check status code is 201
pw.test("Status code is 201", ()=> {
    pw.expect(pw.response.status).toBe(201);
});
```

### 4. update PUT (PUT) - Přepsání záznamu

**Endpoint:** `<<baseUri>><<path>>/admin/{resource}/1`

**Body:** JSON s **kompletními** daty (createOrReplace)

### 5. update PATCH (PATCH) - Částečná aktualizace

**Endpoint:** `<<baseUri>><<path>>/admin/{resource}/1`

**Body:** JSON s **částečnými** daty (update)

### 6. destroy (DELETE) - Smazání záznamu

**Endpoint:** `<<baseUri>><<path>>/admin/{resource}/1`

### 7. actions (POST) - Hromadné akce

**Endpoint:** `<<baseUri>><<path>>/admin/{resource}/actions`

**Body s různými typy akcí:**

**Restore:**
```json
{
  "type": "restore",
  "ids": [1, 2, 3]
}
```

**Delete:**
```json
{
  "type": "delete",
  "ids": [1, 2, 3]
}
```

**DeleteForce:**
```json
{
  "type": "deleteForce",
  "ids": [1, 2, 3]
}
```

**Export:**
```json
{
  "type": "export",
  "ids": [1, 2, 3]
}
```

## Standardní headers

Každý request musí obsahovat tyto headers:

```json
"headers": [
  {
    "key": "X-Data-Lang",
    "value": "<<dataLocale>>",
    "active": true,
    "description": "Jazyk dat (cs_CZ, en_US, de_DE, ...)"
  },
  {
    "key": "X-Lang",
    "value": "<<locale>>",
    "active": true,
    "description": "Jazyk aplikace (cs_CZ, en_US, de_DE, ...)"
  },
  {
    "key": "X-Channel",
    "value": "<<X-Channel>>",
    "active": true,
    "description": "ID nebo klíč kanálu/tenantu (tenant/channel identifikátor, např. 'main' nebo '1')"
  },
  {
    "key": "Accept",
    "value": "application/json",
    "active": true,
    "description": ""
  },
  {
    "key": "Content-Type",
    "value": "application/json",
    "active": true,
    "description": ""
  },
  {
    "key": "Authorization",
    "value": "Bearer <<authToken>>",
    "active": true,
    "description": ""
  },
  {
    "key": "X-Cache-Control",
    "value": "no-cache",
    "active": false,
    "description": "Vypnout cache pro testování (nepovinné)"
  }
]
```

## Proměnné

**DŮLEŽITÉ:** V kolekci **NEPOUŽÍVAJÍ** proměnné na úrovni kolekce!

```json
"variables": []
```

Proměnné se **musí nastavit v Environment** (prostředí) v Hoppscotch.

## Environment v Hoppscotch

Kolekce používá tyto proměnné z Environment:

| Proměnná | Výchozí hodnota | Popis |
|----------|-----------------|-------|
| `baseUri` | `http://localhost:8000` | Základní URL API |
| `path` | `/api` | Cesta k API |
| `locale` | `cs_CZ` | Jazyk aplikace |
| `dataLocale` | `cs_CZ` | Jazyk dat |
| `X-Channel` | `main` | ID nebo klíč kanálu/tenantu |
| `authToken` | `tvůj_bearer_token` | Bearer token |

### Nastavení Environment

**Krok 1:** Vytvoř nové Environment v Hoppscotch
- Vlevo dole klikni na "Environments"
- Klikni na "+ Create new"
- Pojmenuj (např. "AM API Local")

**Krok 2:** Přidej proměnné do Environment

**JSON pro import:**
```json
{
  "name": "AM API Local",
  "variables": [
    { "key": "baseUri", "value": "http://localhost:8000" },
    { "key": "path", "value": "/api" },
    { "key": "locale", "value": "cs_CZ" },
    { "key": "dataLocale", "value": "cs_CZ" },
    { "key": "X-Channel", "value": "main" },
    { "key": "authToken", "value": "tvůj_bearer_token_zde" }
  ]
}
```

**Krok 3:** Vyber Environment v pravém horním rohu Hoppscotch

**Krok 4:** Spusť requesty

### Proč Environment, ne Collection variables?

- **Environment** = specifické pro každé prostředí (local, staging, production)
- **Collection variables** = globální pro všechny kolekce (nelze rozlišit prostředí)
- Hoppscotch automaticky nahradí `<<baseUri>>` hodnotou z Environment

## Testovací skripty

### Standardní test (GET, PUT, PATCH, DELETE):

```javascript
// Check status code is 200
pw.test("Status code is 200", ()=> {
    pw.expect(pw.response.status).toBe(200);
});
```

### Test pro create (POST):

```javascript
// Check status code is 201
pw.test("Status code is 201", ()=> {
    pw.expect(pw.response.status).toBe(201);
});
```

## Umístění souborů

**Pro Admin controllery:**
```
doc/hoppscotch/admin/{ModelName}.json
```

**Pro Public controllery:**
```
doc/hoppscotch/public/{ModelName}.json
```

**Pro společné controllery:**
```
doc/hoppscotch/{ModelName}.json
```

**Příklady:**
- `doc/hoppscotch/admin/Vats.json`
- `doc/hoppscotch/admin/Articles.json`
- `doc/hoppscotch/admin/ChilliPeppers.json`
- `doc/hoppscotch/public/Ratings.json`
- `doc/hoppscotch/Auth.json`

## Proces generování

1. **Zeptej se uživatele**, zda chce kolekci
2. **Vygeneruj kolekci** podle struktury výše
3. **Použij skutečná data** z validace pro body
4. **Zahrň všechny REST metody** z kontrolleru
5. **Přidej Enum endpointy** (pokud relevantní, prefixuj `Enum`)
6. **Ulož do správné složky** podle typu
7. **Informuj uživatele** o umístění souboru

## Příklad body pro requests

Body by mělo odpovídat validačním pravidlům z `{Model}Validation`.

### Store (POST):

```json
{
  "key": "example-key",
  "name": "Příklad názvu",
  "description": "Popis entity",
  "value": 100,
  "published": true,
  "validSince": "2023-01-01 00:00:00",
  "validTo": null,
  "image": 1,
  "category": 5
}
```

### Update PUT (kompletní data):

```json
{
  "key": "updated-key",
  "name": "Aktualizovaný název",
  "description": "Nový popis",
  "value": 200,
  "published": true,
  "validSince": "2023-01-01 00:00:00",
  "validTo": "2024-12-31 23:59:59",
  "image": 2,
  "category": 6
}
```

### Update PATCH (částečná data):

```json
{
  "name": "Jen změna názvu",
  "value": 150
}
```

## Enum endpointy

Pokud kontroler používá EnumController, přidej relevantní enum requesty:

### Enum - {Model} keys

**Endpoint:** `<<baseUri>><<path>>/enums/{model}-keys`
**Method:** GET
**Name:** `Enum - {Model} keys`

### Enum - {Model} pairs

**Endpoint:** `<<baseUri>><<path>>/enums/{model}-pairs`
**Method:** GET
**Name:** `Enum - {Model} pairs`

### Enum - {Model} types

**Endpoint:** `<<baseUri>><<path>>/enums/{model}-types`
**Method:** GET
**Name:** `Enum - {Model} types`

## Ukázka kompletní kolekce

Viz reference: `doc/hoppscotch/admin/Vats.json`

## Kontrola aktuálnosti

**⚠️ DŮLEŽITÉ:** Pomocí MCP Context7 zkontroluj dokumentaci k Hoppscotch a ověř:
- Nové funkce (examples, složky, atd.)
- Změny ve formátu kolekcí
- Změny v požadavcích na requesty

## Checklist

- [ ] Zeptat se uživatele, zda chce kolekci
- [ ] Určit typ controlleru (Admin/Public/Společný)
- [ ] Vygenerovat všechny REST requesty (index, show, store, update PUT, update PATCH, destroy)
- [ ] Přidat actions endpoint (pokud existuje)
- [ ] Přidat enum endpointy (pokud relevantní)
- [ ] Použít skutečná data z validace pro body
- [ ] Přidat description k parametrům
- [ ] Přidat standardní headers
- [ ] Přidat testovací skripty
- [ ] Nastavit proměnné jako `<<variable>>`
- [ ] Uložit do správné složky
- [ ] Informovat uživatele o umístění

**⚠️ Důležité:**
- **Environment** - proměnné musí být v Environment, ne v kolekci
- **Headers** - všechny requesty mají standardní headers
- **Test scripts** - 200 pro GET/PUT/PATCH/DELETE, 201 pro POST
- **Body** - podle validace, kompletní pro PUT, částečné pro PATCH
- **Parametry** - s description a ukázkovými hodnotami
- **Enum** - prefixuj slovem "Enum"

Reference: [Controller REST Methods](controller-rest-methods.md), [Validation Structure](validation-structure.md)