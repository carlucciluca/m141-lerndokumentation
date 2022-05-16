# Übung - Indexierung und Performance

## Vorbereitung

Um Testdaten zu erzeugen, habe ich diese Funktion Importiert:

```
populatePhones = function(area, start, stop) {
    for(var i = start; i < stop; i++) {
    var country = 1 + ((Math.random() * 8) << 0);
    var num = (country * 1e10) + (area * 1e7) + i;
    var fullNumber = "+" + country + " " + area + "-" + i;
    db.phones.insert({
        _id: num,
        components: {
        country: country,
        area: area,
        prefix: (i * 1e-4) << 0,
        number: i
        },
        display: fullNumber
    });
    print("Inserted number " + fullNumber);
    }
    print("Done!");
}
```

Zusätzlich eine neue Datenbank erstellen:

```
use phone
```



## Daten generieren

Funktion aufrufen, um Testdaten zu generieren:
```
populatePhones(800, 5550000, 5650000) // Dauert eine Weile
```

Es werden nun ganz viele Telefonnummern erzeugt. Die Ortsnetzkennzahl (ONKz) ist dabei 800 und die Teilnehmerrufnummer (RufNr) ist dabei zwischen 5550000 und 5650000.

## Indices auslesen

**Aufgabe:**  
**Aufgabenstellung**

```
Code
```

**Lösung:**  
Text

## Austesten

**Aufgabe:**  
**Aufgabenstellung**

```
Code
```

**Lösung:**  
Text

## Index erfassen

**Aufgabe:**  
**Aufgabenstellung**

```
Code
```

**Lösung:**  
Text