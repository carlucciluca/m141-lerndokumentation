# MongoDB Datenbankdesign

## Embedding

**Vorteile:**  
* Mit einer Query können alle Daten geladen werden.
* Es werden keine Joins benötigt.
* Alle CRUD-Operationen von einem Document sind ACID-kompatibel.

**Nachteile:**  
* Eine Query auf grosse und viele Documents kann bei vielen doppelten Daten zu einem grossen Overhead führen und damit die Performance beeinflussen.

### Beispiel

```json
{
    "first_name": "Paul",
    "surname": "Miller",
    "cell": "447557505611",
    "city": "London",
    "location": [45.123, 47.232],
    "profession": ["banking", "finance", "trader"],
    "cars": [
        {
            "model": "Bentley",
            "year": 1973
        },
        {
            "model": "Rolls Royce",
            "year": 1965
        }
    ]
}
```

## Referencing

**Vorteile:**  
* Es sind kleinere Documents.
* Documents können kleiner als 16MB sein (das ist ein Limit).
* Kann nützlich sein, wenn die referenzierten Daten nicht bei jeder Query/nicht oft geladen werden müssen.
* Die Redundanzen in den Informationen werden reduziert.

**Nachteile:**  
* Abfragen werden komplizierter.

### Beispiel mit \$lookup

Collection ```orders``` erstellen:

```json
db.orders.insertMany( [
   { "_id" : 1, "item" : "almonds", "price" : 12, "quantity" : 2 },
   { "_id" : 2, "item" : "pecans", "price" : 20, "quantity" : 1 },
   { "_id" : 3  }
] )
```

Collection ```inventory``` erstellen:

```json
db.inventory.insertMany( [
   { "_id" : 1, "sku" : "almonds", "description": "product 1", "instock" : 120 },
   { "_id" : 2, "sku" : "bread", "description": "product 2", "instock" : 80 },
   { "_id" : 3, "sku" : "cashews", "description": "product 3", "instock" : 60 },
   { "_id" : 4, "sku" : "pecans", "description": "product 4", "instock" : 70 },
   { "_id" : 5, "sku": null, "description": "Incomplete" },
   { "_id" : 6 }
] )
```

Beide Collections verknüfpen/joinen:

```json
db.orders.aggregate( [
   {
     $lookup:
       {
         from: "inventory",
         localField: "item",
         foreignField: "sku",
         as: "inventory_docs"
       }
  }
] )
```

Ausgabe:

```json
{
   "_id" : 1,
   "item" : "almonds",
   "price" : 12,
   "quantity" : 2,
   "inventory_docs" : [
      { "_id" : 1, "sku" : "almonds", "description" : "product 1", "instock" : 120 }
   ]
}
{
   "_id" : 2,
   "item" : "pecans",
   "price" : 20,
   "quantity" : 1,
   "inventory_docs" : [
      { "_id" : 4, "sku" : "pecans", "description" : "product 4", "instock" : 70 }
   ]
}
{
   "_id" : 3,
   "inventory_docs" : [
      { "_id" : 5, "sku" : null, "description" : "Incomplete" },
      { "_id" : 6 }
   ]
}
```

## Beziehungen abbilden

### One-to-One

```json
{
 "_id": "ObjectId('AAA')",
 "name": "Joe Karlsson",
 "company": "MongoDB",
 "twitter": "@JoeKarlsson1",
 "twitch": "joe_karlsson",
 "tiktok": "joekarlsson",
 "website": "joekarlsson.com"
}
```

### One-to-Few

```json
{
 "_id": "ObjectId('AAA')",
 "name": "Joe Karlsson",
 "company": "MongoDB",
 "twitter": "@JoeKarlsson1",
 "twitch": "joe_karlsson",
 "tiktok": "joekarlsson",
 "website": "joekarlsson.com",
 "addresses": [
     { "street": "123 Sesame St", "city": "Anytown", "cc": "USA" },  
     { "street": "123 Avenue Q",  "city": "New York", "cc": "USA" }
 ]
}
```

### One-to-Many

**Products:**  

```json
{
   "name": "left-handed smoke shifter",
   "manufacturer": "Acme Corp",
   "catalog_number": "1234",
   "parts": ["ObjectID('AAAA')", "ObjectID('BBBB')", "ObjectID('CCCC')"]
}
```

**Parts:**  

```json
{
   "_id" : "ObjectID('AAAA')",
   "partno" : "123-aff-456",
   "name" : "#4 grommet",
   "qty": "94",
   "cost": "0.94",
   "price":" 3.99"
}
```

### Many-to-Many

**Users:**  

```json
{
   "_id": ObjectID("AAF1"),
   "name": "Kate Monster",
   "tasks": [ObjectID("ADF9"), ObjectID("AE02"), ObjectID("AE73")]
}
```

**Tasks:**  

```json
{
   "_id": ObjectID("ADF9"),
   "description": "Write blog post about MongoDB schema design",
   "due_date": ISODate("2014-04-01"),
   "owners": [ObjectID("AAF1"), ObjectID("BB3G")]
}
```

## Zusammenfassung der Regeln

1. Embedded Documents favorisieren, ausser es gibt gute Gründe dagegen.
2. Ist es nötig ein einzelnes Document auch einzeln anzeigen zu können, kann das ein Grund sein dieses Document nicht zu embedden.
3. Joins/Lookups wann immer möglich versuchen zu vermeiden. Hilft es allerdings ein besseres(performanteres) Schema zu erreichen - mit Referenzen arbeiten.
4. Nested Arrays sollten nicht "unbegrenzt" gross werden können.
5. Many-To-Many - hier muss mit Referenzen gearbeitet werden.
