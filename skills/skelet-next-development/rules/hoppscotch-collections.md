---
title: Hoppscotch kolekce generování
impact: MEDIUM
impactDescription: Hoppscotch kolekce umožňují rychlé testování API endpointů
tags: hoppscotch, api-testing, collections, rest
---

## Hoppscotch kolekce generování

**Impact: MEDIUM**

Po vytvoření nového kontrolleru lze **nepovinně** vygenerovat JSON kolekci pro import do Hoppscotch, která umožní okamžité testování nových API endpointů.

**⚠️ DŮLEŽITÉ:** Response examples jsou **povinné tam, kde demonstrují různé stavy nebo parametry** (index s filtry, show s withRelations, apod.). Jednoduché operace (delete, restore) nemusí mít examples.

**Verze:** Hoppscotch v11  
**Formát:** JSON export (Array of Collections) s folder strukturou  
**Umístění:** 
- Admin: `doc/hoppscotch/admin/{ModelName}.json`
- Public: `doc/hoppscotch/public/{ModelName}.json`
- Enum: `doc/hoppscotch/enum/{ModelName}.json`

**⚠️ DŮLEŽITÉ:** Každý modul má **vlastní soubor**, ale requests jsou organizované do **folder struktury** podle typu API (Admin/Public/Enum).

## Kdy použít

Po vytvoření kontrolleru se **zeptaj uživatele**:

> "Chceš, abych vygeneroval Hoppscotch kolekci pro testování těchto endpointů?"

## ⚠️ KRITICKY DŮLEŽITÉ - Response Examples

**Response examples MUSÍ být přidány tam, kde demonstrují různé stavy nebo parametry!**

### Kdy PŘIDAT examples:

✅ **POVINNÉ examples:**
- **index** → 2-3 examples (všechny, publikované, různé filtry)
- **show** → 1+ examples pokud má `with*` parametry (withRelations, withMeasurements, ...)
- **store** → 1 example (úspěšné vytvoření)
- **update PUT** → 1 example (kompletní aktualizace)
- **update PATCH** → 1 example (částečná aktualizace)
- **actions** → 1 example pro každý typ akce (restore, delete, deleteForce)

### Kdy VYNECHAT examples:

❌ **NEPOVINNÉ (prázdné `{}` je OK):**
- **destroy** → jednoduchá operace, není co demonstrovat
- **restore** → jednoduchá operace, není co demonstrovat
- **show** → pokud NEMÁ žádné `with*` parametry

**Příklad správného použití:**
```json
// Index - MUSÍ mít examples (různé filtry)
{
  "name": "index - List articles",
  "endpoint": "...",
  "responses": {
    "Všechny články": {...},
    "Publikované": {...}
  }
}

// Show s with* parametry - MUSÍ mít examples
{
  "name": "show - Get article detail",
  "endpoint": "...",
  "params": [
    {"key": "withRelations", "value": "1", "active": false}
  ],
  "responses": {
    "Základní detail": {...},
    "S relacemi": {...}
  }
}

// Destroy - prázdné OK (nic zajímavého)
{
  "name": "destroy - Delete article",
  "endpoint": "...",
  "responses": {}
}

// Restore - prázdné OK (nic zajímavého)
{
  "name": "restore - Restore article",
  "endpoint": "...",
  "responses": {}
}
```

## Struktura exportu v11

Každý soubor je **array kolekcí**, kde top-level kolekce představuje typ API (Admin/Public/Enum).
Uvnitř jsou **folders** pro jednotlivé moduly.

### Příklad: Admin controller (doc/hoppscotch/admin/Contracts.json)

```json
[
  {
    "v": 11,
    "id": "cml6gl372014q41pexhukrq48",
    "name": "Admin",
    "folders": [
      {
        "v": 11,
        "id": "cml6gkf7x014h41pebr1b2hvs",
        "name": "Contracts",
        "folders": [],
        "requests": [
          {
            "v": "17",
            "name": "index - List contracts",
            "method": "GET",
            "endpoint": "<<baseUri>><<path>>/admin/contracts",
            ...
          }
        ],
        "auth": {
          "authType": "inherit",
          "authActive": true
        },
        "headers": [],
        "variables": [],
        "description": null
      }
    ],
    "requests": [],
    "auth": {
      "authType": "inherit",
      "authActive": true
    },
    "headers": [],
    "variables": [],
    "description": null
  }
]
```

### Příklad: Public controller (doc/hoppscotch/public/Articles.json)

```json
[
  {
    "v": 11,
    "id": "cml6gluh6015041pexmr842em",
    "name": "Public",
    "folders": [
      {
        "v": 11,
        "id": "cml6grzun015141peihhwxmln",
        "name": "Articles",
        "folders": [],
        "requests": [...]
      }
    ],
    "requests": [],
    "auth": {
      "authType": "inherit",
      "authActive": true
    },
    "headers": [],
    "variables": [],
    "description": null
  }
]
```

### Příklad: Enum endpoints (doc/hoppscotch/enum/ContractStatus.json)

```json
[
  {
    "v": 11,
    "id": "unique_enum_collection_id",
    "name": "Enums",
    "folders": [
      {
        "v": 11,
        "id": "unique_folder_id",
        "name": "Contract Status",
        "folders": [],
        "requests": [
          {
            "v": "17",
            "name": "Get contract status enum",
            "method": "GET",
            "endpoint": "<<baseUri>><<path>>/enums/contract-status",
            ...
          }
        ]
      }
    ],
    "requests": [],
    "auth": {
      "authType": "inherit",
      "authActive": true
    },
    "headers": [],
    "variables": [],
    "description": null
  }
]
```

## Formát requestu v17

Každý request má následující strukturu:

```json
{
  "v": "17",
  "auth": {
    "authType": "inherit",
    "authActive": true
  },
  "body": {
    "body": "",
    "contentType": "application/json"
  },
  "name": "index - List contracts",
  "method": "GET",
  "params": [...],
  "headers": [...],
  "endpoint": "<<baseUri>><<path>>/admin/contracts",
  "responses": {
    "Plánované": {
      "body": "",
      "code": 200,
      "name": "Plánované",
      "status": "OK",
      "headers": [],
      "originalRequest": {...}
    }
  },
  "testScript": "pw.test(\"Status code is 200\", ()=> { pw.expect(pw.response.status).toBe(200); });",
  "description": null,
  "preRequestScript": "",
  "requestVariables": [],
  "id": "unique_request_id"
}
```

### Naming Convention

Používej formát: `{method} - {Description}`

**Příklady:**
- `index - List contracts`
- `show - Get contract detail`
- `store - Create new contract`
- `update - Update contract (PUT)`
- `update - Partial update (PATCH)`
- `destroy - Delete contract`
- `actions - Bulk operations`

### Response Examples

**Vytvářej tam, kde demonstrují různé stavy nebo parametry!**

Response examples mají smysl pro:
- **index** - různé filtry (např. "Všechny články", "Publikované", "Bez překladu")
- **show s parametry** - různé `with*` parametry (např. "Základní", "S relacemi", "S měřením")
- **store/update** - ukázka úspěšného requestu s daty
- **actions** - různé typy akcí (restore, delete, deleteForce)

Response examples **nemají smysl** pro:
- **destroy** - jednoduchá operace
- **restore** - jednoduchá operace
- **show bez parametrů** - jen jeden stav

**Doporučené použití:**
- Různé filtry (např. "Plánované" vs "Dokončené")
- Různé parametry (např. "S měřením" vs "Bez měření")
- Různé stavy dat (např. "Prázdný seznam" vs "S daty")

## Response examples - kdy je vytvářet

### ✅ PŘIDEJ examples (demonstrují různé stavy)

#### 1. index (GET) - 2-3 examples

**Důvod:** Ukázat různé filtry a možnosti vyhledávání
```json
"responses": {
  "Všechny články": {
    "body": "",
    "code": 200,
    "name": "Všechny články",
    "status": "OK",
    "headers": [],
    "originalRequest": {
      "v": "6",
      "method": "GET",
      "params": [],
      "endpoint": "<<baseUri>><<path>>/admin/articles",
      ...
    }
  },
  "Publikované články": {
    "body": "",
    "code": 200,
    "name": "Publikované články",
    "status": "OK",
    "headers": [],
    "originalRequest": {
      "v": "6",
      "method": "GET",
      "params": [
        {
          "key": "filter[published][value]",
          "value": "1",
          "active": true,
          "description": "Filter by published status"
        }
      ],
      "endpoint": "<<baseUri>><<path>>/admin/articles",
      ...
    }
  }
}
```

#### 2. show (GET) - 1+ examples POUZE pokud má with* parametry

**Důvod:** Ukázat rozdíl mezi základním detailem a detailem s relacemi/měřením

**PŘIDEJ examples pokud:**
- Endpoint má `withRelations`, `withMeasurements`, `withGallery` apod.

**VYNECH examples pokud:**
- Endpoint je jednoduchý detail bez parametrů

**Příklad:**
```json
"responses": {
  "Detail článku": {
    "body": "",
    "code": 200,
    "name": "Detail článku",
    "status": "OK",
    "headers": [],
    "originalRequest": {
      "v": "6",
      "method": "GET",
      "params": [],
      "endpoint": "<<baseUri>><<path>>/admin/articles/1",
      ...
    }
  }
}
```

#### 3. store (POST) - 1 example

**Důvod:** Ukázat kompletní request body s daty

**Příklad:**
```json
"responses": {
  "Úspěšné vytvoření": {
    "body": "",
    "code": 201,
    "name": "Úspěšné vytvoření",
    "status": "Created",
    "headers": [],
    "originalRequest": {
      "v": "6",
      "method": "POST",
      "body": {
        "body": "{...ukázková data...}",
        "contentType": "application/json"
      },
      "endpoint": "<<baseUri>><<path>>/admin/articles",
      ...
    }
  }
}
```

#### 4. update PUT (PUT) - 1 example

**Důvod:** Ukázat kompletní request body (createOrReplace)

**Příklad:**
```json
"responses": {
  "Úspěšná aktualizace": {
    "body": "",
    "code": 200,
    "name": "Úspěšná aktualizace",
    "status": "OK",
    "headers": [],
    "originalRequest": {
      "v": "6",
      "method": "PUT",
      "body": {
        "body": "{...kompletní data...}",
        "contentType": "application/json"
      },
      "endpoint": "<<baseUri>><<path>>/admin/articles/1",
      ...
    }
  }
}
```

#### 5. update PATCH (PATCH) - 1 example

**Důvod:** Ukázat částečný request body (rozdíl od PUT)

**Příklad:**
```json
"responses": {
  "Částečná aktualizace": {
    "body": "",
    "code": 200,
    "name": "Částečná aktualizace",
    "status": "OK",
    "headers": [],
    "originalRequest": {
      "v": "6",
      "method": "PATCH",
      "body": {
        "body": "{\"title\": \"...\", \"priority\": 5}",
        "contentType": "application/json"
      },
      "endpoint": "<<baseUri>><<path>>/admin/articles/1",
      ...
    }
  }
}
```

#### 6. actions (POST) - 1 example pro každý typ akce

**Důvod:** Ukázat různé typy hromadných operací (restore, delete, deleteForce)

**Příklad:**
```json
"responses": {
  "Hromadné obnovení": {
    "body": "",
    "code": 200,
    "name": "Hromadné obnovení",
    "status": "OK",
    "headers": [],
    "originalRequest": {
      "v": "6",
      "method": "POST",
      "body": {
        "body": "{\"type\": \"restore\", \"ids\": [1, 2, 3]}",
        "contentType": "application/json"
      },
      "endpoint": "<<baseUri>><<path>>/admin/articles/actions",
      ...
    }
  }
}
```

### ❌ VYNECH examples (jednoduché operace, nic zajímavého)

#### destroy (DELETE) - `responses: {}`

**Důvod:** Jednoduchá operace, žádné různé stavy k demonstraci

**Správně:**
```json
{
  "name": "destroy - Delete article",
  "method": "DELETE",
  "endpoint": "<<baseUri>><<path>>/admin/articles/1",
  "responses": {},  // Prázdné OK - nic zajímavého
  ...
}
```

#### restore (PATCH/POST) - `responses: {}`

**Důvod:** Jednoduchá operace, žádné různé stavy k demonstraci

**Správně:**
```json
{
  "name": "restore - Restore article",
  "method": "PATCH",
  "endpoint": "<<baseUri>><<path>>/admin/articles/1/restore",
  "responses": {},  // Prázdné OK - nic zajímavého
  ...
}
```

#### show bez parametrů - `responses: {}`

**Důvod:** Jen jeden stav, žádné `with*` parametry

**Správně:**
```json
{
  "name": "show - Get article detail",
  "method": "GET",
  "endpoint": "<<baseUri>><<path>>/admin/articles/1",
  "params": [],  // Žádné with* parametry
  "responses": {},  // Prázdné OK - jen jeden stav
  ...
}
```

---

## Požadované requesty pro REST API

### 1. index (GET) - Seznam záznamů

**Name:** `index - List {resource}`  
**Endpoint:** `<<baseUri>><<path>>/admin/{resource}`

**Parametry** (všechny neaktivní pro ukázku):
```json
"params": [
  {
    "key": "trash",
    "value": "0",
    "active": false,
    "description": "Show soft-deleted records (0 or 1)"
  },
  {
    "key": "page[size]",
    "value": "20",
    "active": false,
    "description": "Page size for pagination"
  },
  {
    "key": "page[number]",
    "value": "1",
    "active": false,
    "description": "Page number"
  },
  {
    "key": "filter[id][]",
    "value": "1",
    "active": false,
    "description": "Filter by ID"
  },
  {
    "key": "filter[title][value]",
    "value": "example",
    "active": false,
    "description": "Filter by title (search)"
  },
  {
    "key": "filter[status][value]",
    "value": "planned",
    "active": false,
    "description": "Filter by status (planned, inProgress, completed, onHold)"
  },
  {
    "key": "sort[0][field]",
    "value": "id",
    "active": false,
    "description": "Sort field (id, key, title, status, createdAt)"
  },
  {
    "key": "sort[0][order]",
    "value": "desc",
    "active": false,
    "description": "Sort order (asc or desc)"
  }
]
```

**Response Examples (volitelné):**
Můžeš přidat response examples pro různé filtry:
- "Plánované" - s aktivním filtrem `filter[status][value]=planned`
- "Dokončené" - s aktivním filtrem `filter[status][value]=completed`
- "Prázdný seznam" - bez záznamů

### 2. show (GET) - Detail záznamu

**Name:** `show - Get {resource} detail`  
**Endpoint:** `<<baseUri>><<path>>/admin/{resource}/1`

**Response Examples (volitelné):**
- "S měřením" - s parametrem `withMeasurements=true`
- "Základní" - bez dodatečných parametrů

### 3. store (POST) - Vytvoření nového záznamu

**Name:** `store - Create new {resource}`  
**Endpoint:** `<<baseUri>><<path>>/admin/{resource}`

**Body:** JSON s ukázkovými daty podle validace

```json
{
  "title": "Nová stavba",
  "text": "Podrobný popis stavby",
  "status": "planned",
  "image": null,
  "attachment": null,
  "video": null,
  "constructionStartAt": "2026-02-01",
  "constructionEndAt": "2026-12-31",
  "note": "Interní poznámka"
}
```

**Test script:**
```javascript
pw.test("Status code is 201", ()=> {
    pw.expect(pw.response.status).toBe(201);
});
```

### 4. update PUT (PUT) - Přepsání záznamu

**Name:** `update - Update {resource} (PUT)`  
**Endpoint:** `<<baseUri>><<path>>/admin/{resource}/1`

**Body:** JSON s **kompletními** daty (createOrReplace)

### 5. update PATCH (PATCH) - Částečná aktualizace

**Name:** `update - Partial update (PATCH)`  
**Endpoint:** `<<baseUri>><<path>>/admin/{resource}/1`

**Body:** JSON s **částečnými** daty (update)

### 6. destroy (DELETE) - Smazání záznamu

**Name:** `destroy - Delete {resource}`  
**Endpoint:** `<<baseUri>><<path>>/admin/{resource}/1`

### 7. actions (POST) - Hromadné akce

**Name:** `actions - Bulk operations`  
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
pw.test("Status code is 200", ()=> {
    pw.expect(pw.response.status).toBe(200);
});
```

### Test pro create (POST):

```javascript
pw.test("Status code is 201", ()=> {
    pw.expect(pw.response.status).toBe(201);
});
```

## Umístění souborů

Každý modul má **vlastní JSON soubor** s folder strukturou uvnitř.

**Pro Admin controllery:**
```
doc/hoppscotch/admin/{ModelName}.json
```

**Pro Public controllery:**
```
doc/hoppscotch/public/{ModelName}.json
```

**Pro Enum endpointy:**
```
doc/hoppscotch/enum/{EnumName}.json
```

**Příklady:**
- `doc/hoppscotch/admin/Vats.json`
- `doc/hoppscotch/admin/Articles.json`
- `doc/hoppscotch/admin/Contracts.json`
- `doc/hoppscotch/public/Articles.json`
- `doc/hoppscotch/public/Ratings.json`
- `doc/hoppscotch/enum/ContractStatus.json`

### Struktura uvnitř souboru

Každý soubor obsahuje:
1. **Array kolekcí** (top-level)
2. **Kolekce** s názvem typu (Admin/Public/Enums)
3. **Folders** - jeden folder pro modul
4. **Requests** - všechny REST endpointy modulu

## Proces generování

1. **Zeptej se uživatele**, zda chce kolekci
2. **Urči typ API** (Admin/Public/Enum)
3. **Vytvoř folder strukturu:**
   - Top-level kolekce s názvem typu (Admin/Public/Enums)
   - Folder s názvem modulu (např. "Contracts", "Articles")
   - Requests uvnitř folderu
4. **Vygeneruj všechny REST metody** z kontrolleru
5. **Použij skutečná data** z validace pro body
6. **Přidej response examples TAM, KDE JSOU UŽITEČNÉ:**
   - ✅ index (2-3 examples pro různé filtry)
   - ✅ show s `with*` parametry (1+ examples)
   - ✅ store/update (1 example s daty)
   - ✅ actions (1 example pro každý typ)
   - ❌ destroy/restore (prázdné `{}` OK)
7. **Zahrň Enum endpointy** (pokud relevantní, do složky Enums)
8. **Ulož do správné složky** podle typu
9. **Informuj uživatele** o umístění souboru a kde byly přidány examples

### Generování response examples

Response examples jsou **POVINNÉ** pro všechny requesty. Vytvoř je pro:
- **index** - různé filtry (např. "Plánované", "Dokončené", "Všechny")
- **show** - různé parametry (např. "S měřením", "Základní", "S relacemi")
- **store/update** - různé validační scénáře

**Formát example:**
```json
"responses": {
  "Plánované": {
    "body": "",
    "code": 200,
    "name": "Plánované",
    "status": "OK",
    "headers": [],
    "originalRequest": {
      "v": "6",
      "auth": { ... },
      "body": { ... },
      "name": "index - List contracts",
      "method": "GET",
      "params": [
        {
          "key": "filter[status][value]",
          "value": "planned",
          "active": true,
          "description": "Filter by status"
        }
      ],
      "headers": [ ... ],
      "endpoint": "<<baseUri>><<path>>/admin/contracts",
      "requestVariables": []
    }
  }
}
```

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

Pokud kontroler používá EnumController, vytvoř **samostatný soubor** v `doc/hoppscotch/enum/{EnumName}.json`.

### Struktura Enum souboru

```json
[
  {
    "v": 11,
    "id": "unique_enum_collection_id",
    "name": "Enums",
    "folders": [
      {
        "v": 11,
        "id": "unique_folder_id",
        "name": "Contract Status",
        "folders": [],
        "requests": [
          {
            "v": "17",
            "auth": {
              "authType": "inherit",
              "authActive": true
            },
            "body": {
              "body": null,
              "contentType": null
            },
            "name": "Get contract status enum",
            "method": "GET",
            "params": [],
            "headers": [
              {
                "key": "X-Lang",
                "value": "<<locale>>",
                "active": true,
                "description": "Application language"
              },
              {
                "key": "X-Channel",
                "value": "<<X-Channel>>",
                "active": true,
                "description": "Channel/tenant ID"
              },
              {
                "key": "Accept",
                "value": "application/json",
                "active": true,
                "description": ""
              },
              {
                "key": "Authorization",
                "value": "Bearer <<authToken>>",
                "active": true,
                "description": ""
              }
            ],
            "endpoint": "<<baseUri>><<path>>/enums/contract-status",
            "responses": {},
            "testScript": "pw.test(\"Status code is 200\", ()=> { pw.expect(pw.response.status).toBe(200); });",
            "description": null,
            "preRequestScript": "",
            "requestVariables": [],
            "id": "unique_request_id"
          }
        ],
        "auth": {
          "authType": "inherit",
          "authActive": true
        },
        "headers": [],
        "variables": [],
        "description": null
      }
    ],
    "requests": [],
    "auth": {
      "authType": "inherit",
      "authActive": true
    },
    "headers": [],
    "variables": [],
    "description": null
  }
]
```

### Naming Convention pro Enum

**Name:** `Get {enum description}`

**Příklady:**
- `Get contract status enum`
- `Get article category enum`
- `Get user role enum`

## Ukázka kompletní kolekce

Viz reference: 
- `doc/hoppscotch/admin/Contracts.json` - Admin kolekce s response examples
- `doc/hoppscotch/public/Articles.json` - Public kolekce
- `.github/skills/skelet-next-development/hoppscotch-example.json` - Kompletní ukázka

### Minimální funkční struktura

```json
[
  {
    "v": 11,
    "id": "cml6gl372014q41pexhukrq48",
    "name": "Admin",
    "folders": [
      {
        "v": 11,
        "id": "cml6gkf7x014h41pebr1b2hvs",
        "name": "Contracts",
        "folders": [],
        "requests": [
          {
            "v": "17",
            "auth": {
              "authType": "inherit",
              "authActive": true
            },
            "body": {
              "body": "",
              "contentType": "application/json"
            },
            "name": "index - List contracts",
            "method": "GET",
            "params": [],
            "headers": [
              {
                "key": "X-Data-Lang",
                "value": "<<dataLocale>>",
                "active": true,
                "description": "Data language (cs_CZ, en_US, de_DE, ...)"
              },
              {
                "key": "X-Lang",
                "value": "<<locale>>",
                "active": true,
                "description": "Application language (cs_CZ, en_US, de_DE, ...)"
              },
              {
                "key": "X-Channel",
                "value": "<<X-Channel>>",
                "active": true,
                "description": "Channel/tenant ID (main, brno, 1, ...)"
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
              }
            ],
            "endpoint": "<<baseUri>><<path>>/admin/contracts",
            "responses": {},
            "testScript": "pw.test(\"Status code is 200\", ()=> { pw.expect(pw.response.status).toBe(200); });",
            "description": null,
            "preRequestScript": "",
            "requestVariables": [],
            "id": "cml6gkftn014p41pe7erl65tt"
          }
        ],
        "auth": {
          "authType": "inherit",
          "authActive": true
        },
        "headers": [],
        "variables": [],
        "description": null
      }
    ],
    "requests": [],
    "auth": {
      "authType": "inherit",
      "authActive": true
    },
    "headers": [],
    "variables": [],
    "description": null
  }
]
```

## Kontrola aktuálnosti

**⚠️ DŮLEŽITÉ:** Pomocí MCP Context7 zkontroluj dokumentaci k Hoppscotch a ověř:
- Nové funkce (examples, složky, atd.)
- Změny ve formátu kolekcí
- Změny v požadavcích na requesty

## Checklist

### Před generováním
- [ ] Zeptat se uživatele, zda chce kolekci
- [ ] Určit typ controlleru (Admin/Public/Enum)

### Struktura souboru
- [ ] Vytvořit array kolekcí (top-level)
- [ ] Vytvořit kolekci s názvem typu (Admin/Public/Enums)
- [ ] Vytvořit folder s názvem modulu
- [ ] Nastavit `"v": 11` pro kolekci a folder
- [ ] Nastavit `"v": "17"` pro requesty

### Requesty
- [ ] Vygenerovat všechny REST requesty (index, show, store, update PUT, update PATCH, destroy)
- [ ] Přidat actions endpoint (pokud existuje)
- [ ] Použít naming convention: `{method} - {Description}`
- [ ] Použít skutečná data z validace pro body
- [ ] Přidat description k parametrům v angličtině
- [ ] Přidat standardní headers
- [ ] Přidat testovací skripty

### Response Examples (tam, kde jsou užitečné)
- [ ] **index**: 2-3 examples pro různé filtry (všechny, publikované, specifický filtr)
- [ ] **show s with* parametry**: 1+ examples (základní, s relacemi, s měřením)
- [ ] **show bez parametrů**: prázdné `{}` OK
- [ ] **store**: 1 example s kompletním request body
- [ ] **update PUT**: 1 example s kompletním request body
- [ ] **update PATCH**: 1 example s částečným request body
- [ ] **destroy**: prázdné `{}` OK (jednoduchá operace)
- [ ] **restore**: prázdné `{}` OK (jednoduchá operace)
- [ ] **actions**: 1 example pro každý typ akce (restore, delete, deleteForce)
- [ ] Každý example má `originalRequest` s aktivními parametry
- [ ] Examples demonstrují **užitečné rozdíly**, ne jen pro formu

### Environment a proměnné
- [ ] Nastavit proměnné jako `<<variable>>`
- [ ] NEPŘIDÁVAT variables do kolekce (zůstane prázdné pole)
- [ ] Auth na úrovni kolekce: `"authType": "inherit"`
- [ ] Auth na úrovni folderu: `"authType": "inherit"`

### Finalizace
- [ ] Uložit do správné složky (admin/public/enum)
- [ ] Informovat uživatele o umístění
- [ ] Připomenout potřebu nastavit Environment v Hoppscotch

**⚠️ Důležité:**
- **Samostatné soubory** - každý modul má vlastní JSON soubor
- **Folder struktura** - requests jsou ve folderu podle modulu
- **Environment** - proměnné musí být v Environment, ne v kolekci
- **Headers** - všechny requesty mají standardní headers
- **Test scripts** - 200 pro GET/PUT/PATCH/DELETE, 201 pro POST
- **Body** - podle validace, kompletní pro PUT, částečné pro PATCH
- **Parametry** - s description v angličtině a ukázkovými hodnotami
- **Naming** - formát `{method} - {Description}`
- **Response examples** - přidávej TAM, KDE JSOU UŽITEČNÉ (index, show s with*, store/update, actions). Jednoduché operace (destroy, restore) mohou mít prázdné `{}`

Reference: [Controller REST Methods](controller-rest-methods.md), [Validation Structure](validation-structure.md)
