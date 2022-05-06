# Anwendung von DB-Benutzern und URIs

## Vorbereitung

Applikation klonen:
```bash
git clone https://github.com/sbhaseen/express-dashboard-demo.git
```

> [!NOTE|style:callout]
> Die Demo-Applikation baut auf folgende Technologien auf:
> * [Express.js](https://expressjs.com/) - A Node.js web application framework
> * [Express-Validator](https://express-validator.github.io/docs/) - A set of validation middlewares for Express.js applications.
> * [Mongoose](https://mongoosejs.com/) - MongoDB Object Document Mapper
> * [(Express) Handlebars](https://handlebarsjs.com/) - A minimal views templating engine.
> * [Helmet.js](https://helmetjs.github.io/) - Helmet helps you secure your Express apps by setting various HTTP headers.
> * [Bootstrap (front-end library)](https://getbootstrap.com/) - Bootstrap is an open source toolkit for developing with HTML, CSS, and JS.
> * [Chart.js](https://www.chartjs.org/) - A JavaScript charting library.

## Inbetriebnahme

1. In das Verzeichnis des Repositories wechseln:
```
cd express-dashboard-demo
```

2. Dependecies installieren:
```
npm install
```

3. Applikation starten:
```
npm run dev
```

4. Die Applikation kann im Webbrowser über [http://localhost:5000](http://localhost:5000) erreicht werden.


## Testen

**Erfassen Sie auf der Applikation einzelne Datensätze (z.Bsp. eine neue Maschine).**

![Create Machine](../images/create_machine.png "Create Machine")

**Verifizieren Sie folgende Dinge:**

* **Über welchen Controller wird kommunziert?**  
  ```/express-dashboard-demo/controllers/machineController.js```

* **Über welches Model wird der Datensatz in der Datenbank gespeichert?**  
  ```/express-dashboard-demo/models/machine.js```

* **Wie sehen die Dokumente in der Datenbank aus?**  
  * Datenbank: mfgdashboard
  * Collections: machines
  * Erstellte Machine abfragen ```db.machines.find()```:
    ```
    { "_id" : ObjectId("627371b132d009219c568d60"), "name" : "TestMachine", "date_of_commissioning" : ISODate("2022-05-05T00:00:00Z"), "date_of_retirement" : ISODate("2023-05-05T00:00:00Z"), "__v" : 0 }
    ```

## Anwendung modifizieren

**Passen Sie die Applikation und Datenbank wie folgt an:**

1. Erfassen Sie auf der MongoDB-Instanz einen Systemadministrator
    ```
    // Zur Admin Datenbank wechseln
    use admin
    db.createUser(
    {
        user: "Admin",
        pwd: passwordPrompt(), // oder cleartext password. passwordPrompt() führt zu einem Prompt auf der Konsole
        roles: [
        { role: "userAdminAnyDatabase", db: "admin" },
        { role: "readWriteAnyDatabase", db: "admin" }
        ]
    }
    )
    ```

2. Erfassen Sie auf der MongoDB-Instanz einen Applikationsbenutzer für diese Applikation
    ```
    // Zur Admin Datenbank wechseln
    use admin
    db.createUser(
    {
        user: "DashboardUser",
        pwd: passwordPrompt(), // oder cleartext password. passwordPrompt() führt zu einem Prompt auf der Konsole
        roles: [
        { role: "readWrite", db: "mfgdashboard" }
        ]
    }
    )
    ```

3. Passen Sie die URI in der Datei server.js entsprechend an
  
URI vor der Anpassung:
```'mongodb://localhost:27017/mfgdashboard'```
  
URI nach der Anpassung:
```'mongodb://DashboardUser:vagrantM141!@localhost:27017/mfgdashboard?authSource=admin'```
