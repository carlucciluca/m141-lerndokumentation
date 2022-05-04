# Command-Line Fun

{
name: Regex (String der mit F beginnt),
population: {lesserthan: 30000}
},
{
hier wird definiert, welche spalten angezeigt werden (0 = nicht anzeigen, 1 = anzeigen)
}

```
db.towns.find( { 
    name: /^F/, 
    population : { $lt : 30000 } 
    }, 
    { _id: 0, 
    name : 1, 
    population : 1 }
    )
```
