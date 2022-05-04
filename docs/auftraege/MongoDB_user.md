# MongoDB-Benutzer

## Benutzer erstellen

1. Mongo Shell öffnen:
```bash
mongo
```

2. Benutzer erstellen:
```
use admin
```
  
```
db.createUser(
  {
    user: "vagrant",
    pwd: "vagrantM141!",
    roles: [
        { role: "userAdminAnyDatabase", db: "admin" },
        { role: "readWrite", db: "cities" }
    ]
  }
)
```

3. Authentifizierung in der Konfigurationsdatei ```/etc/mongod.conf``` aktivieren:
```
security:
    authorization: "enabled"
```

4. MongoDB Dienst neustarten:
```bash
sudo systemctl restart mongod
```

## Benutzer bearbeiten

1. Datenbank "admin" öffnen:
```
use admin
```

2. Benutzer konfigurieren:
```
db.updateUser(
   "<username>",
   {
     customData : { <any information> },
     roles : [
       { role: "<role>", db: "<database>" } | "<role>",
       ...
     ],
     pwd: passwordPrompt(),      // Or  "<cleartext password>"
     authenticationRestrictions: [
        {
          clientSource: ["<IP>" | "<CIDR range>", ...],
          serverAddress: ["<IP>", | "<CIDR range>", ...]
        },
        ...
     ],
     mechanisms: [ "<SCRAM-SHA-1|SCRAM-SHA-256>", ... ],
     passwordDigestor: "<server|client>"
   },
   writeConcern: { <write concern> }
)
```
