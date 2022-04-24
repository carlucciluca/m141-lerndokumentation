# Sakila Datenbank - Stored Procedures, Views und Triggers

## Installation

1. Datei von [hier](https://downloads.mysql.com/docs/sakila-db.zip) herunterladen und entpacken:
```bash
unzip sakila-db.zip
```

2. Danach hat man folgende zwei Dateien:
   * **sakila-db/sakila-schema.sql** - Beinhaltet die Struktur der Datenbank
   * **sakila-db/sakila-data.sql** - Beinhaltet die Daten der Datenbank

3. In das neue Verzeichnis wechseln:
```bash
cd sakila-db
```
  
Danach die Datenbank importieren:
```bash
/usr/bin/mysql -u root -pvagrantM141! < sakila-schema.sql
/usr/bin/mysql -u root -pvagrantM141! < sakila-data.sql
```

## Testing

Testen Sie die Funktion der Datenbank mittels den folgenden Aufrufen:

```sql
USE sakila;
```

```sql
SHOW FULL TABLES;
```

```
+----------------------------+------------+
| Tables_in_sakila           | Table_type |
+----------------------------+------------+
| actor                      | BASE TABLE |
| actor_info                 | VIEW       |
| address                    | BASE TABLE |
| category                   | BASE TABLE |
| city                       | BASE TABLE |
| country                    | BASE TABLE |
| customer                   | BASE TABLE |
| customer_list              | VIEW       |
| film                       | BASE TABLE |
| film_actor                 | BASE TABLE |
| film_category              | BASE TABLE |
| film_list                  | VIEW       |
| film_text                  | BASE TABLE |
| inventory                  | BASE TABLE |
| language                   | BASE TABLE |
| nicer_but_slower_film_list | VIEW       |
| payment                    | BASE TABLE |
| rental                     | BASE TABLE |
| sales_by_film_category     | VIEW       |
| sales_by_store             | VIEW       |
| staff                      | BASE TABLE |
| staff_list                 | VIEW       |
| store                      | BASE TABLE |
+----------------------------+------------+
```

```sql
SELECT COUNT(*) FROM film;
```

```
+----------+
| COUNT(*) |
+----------+
|     1000 |
+----------+
```

```sql
SELECT COUNT(*) FROM film_text;
```

```
+----------+
| COUNT(*) |
+----------+
|     1000 |
+----------+
```

## Anzeigen

> [!NOTE|style:callout]
> Alle Stored Procedures, alle Views und Triggers sind in der Datei *sakila-data.sql* definiert.

1. Zeigen Sie alle Views der Datenbank sakila an:

```sql
SHOW FULL tables where table_type = 'VIEW';
```

```
+----------------------------+------------+
| Tables_in_sakila           | Table_type |
+----------------------------+------------+
| actor_info                 | VIEW       |
| customer_list              | VIEW       |
| film_list                  | VIEW       |
| nicer_but_slower_film_list | VIEW       |
| sales_by_film_category     | VIEW       |
| sales_by_store             | VIEW       |
| staff_list                 | VIEW       |
+----------------------------+------------+
```

2. Zeigen Sie alle Stored Procedures der Datenbank sakila an:

```sql
SHOW PROCEDURE STATUS WHERE db = 'sakila';
```

```
+--------+-------------------+-----------+----------------+---------------------+---------------------+---------------+--------------------------------------------------+----------------------+----------------------+--------------------+
| Db     | Name              | Type      | Definer        | Modified            | Created             | Security_type | Comment                                          | character_set_client | collation_connection | Database Collation |
+--------+-------------------+-----------+----------------+---------------------+---------------------+---------------+--------------------------------------------------+----------------------+----------------------+--------------------+
| sakila | film_in_stock     | PROCEDURE | root@localhost | 2022-03-24 07:58:03 | 2022-03-24 07:58:03 | DEFINER       |                                                  | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
| sakila | film_not_in_stock | PROCEDURE | root@localhost | 2022-03-24 07:58:03 | 2022-03-24 07:58:03 | DEFINER       |                                                  | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
| sakila | rewards_report    | PROCEDURE | root@localhost | 2022-03-24 07:58:03 | 2022-03-24 07:58:03 | DEFINER       | Provides a customizable report on best customers | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
+--------+-------------------+-----------+----------------+---------------------+---------------------+---------------+--------------------------------------------------+----------------------+----------------------+--------------------+
```

3. Zeigen Sie alle Triggers der Datenbank sakila an:

```sql
SHOW TRIGGERS IN sakila;
```

```
+----------------------+--------+----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------+------------------------+----------------------------------------------------------------------------------------------------------------------------------+----------------+----------------------+----------------------+--------------------+
| Trigger              | Event  | Table    | Statement                                                                                                                                                                                                                                                                                                              | Timing | Created                | sql_mode                                                                                                                         | Definer        | character_set_client | collation_connection | Database Collation |
+----------------------+--------+----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------+------------------------+----------------------------------------------------------------------------------------------------------------------------------+----------------+----------------------+----------------------+--------------------+
| customer_create_date | INSERT | customer | SET NEW.create_date = NOW()                                                                                                                                                                                                                                                                                            | BEFORE | 2022-03-24 07:59:48.33 | STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION | root@localhost | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
| ins_film             | INSERT | film     | BEGIN
    INSERT INTO film_text (film_id, title, description)
        VALUES (new.film_id, new.title, new.description);
  END                                                                                                                                                                                          | AFTER  | 2022-03-24 07:58:03.14 | STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION | root@localhost | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
| upd_film             | UPDATE | film     | BEGIN
    IF (old.title != new.title) OR (old.description != new.description) OR (old.film_id != new.film_id)
    THEN
        UPDATE film_text
            SET title=new.title,
                description=new.description,
                film_id=new.film_id
        WHERE film_id=old.film_id;
    END IF;
  END | AFTER  | 2022-03-24 07:58:03.17 | STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION | root@localhost | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
| del_film             | DELETE | film     | BEGIN
    DELETE FROM film_text WHERE film_id = old.film_id;
  END                                                                                                                                                                                                                                                     | AFTER  | 2022-03-24 07:58:03.19 | STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION | root@localhost | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
| payment_date         | INSERT | payment  | SET NEW.payment_date = NOW()                                                                                                                                                                                                                                                                                           | BEFORE | 2022-03-24 07:59:49.40 | STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION | root@localhost | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
| rental_date          | INSERT | rental   | SET NEW.rental_date = NOW()                                                                                                                                                                                                                                                                                            | BEFORE | 2022-03-24 07:59:49.98 | STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION | root@localhost | utf8mb4              | utf8mb4_0900_ai_ci   | utf8mb4_0900_ai_ci |
+----------------------+--------+----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+--------+------------------------+----------------------------------------------------------------------------------------------------------------------------------+----------------+----------------------+----------------------+--------------------+
```

## Anwenden

### Stored Procedures

Stored Procedures sind SQL-Anweisungen, die datenbankseitig gespeichert sind. Mit ihnrn lassen sich redundante Lese-, Einfüge- und Änderungsbefehle vermeiden und erhöhen somit die Geschwindigkeit. Zusätzlich bleiben dadurch Tabellen- und Spaltennamen verborgen.

**Aufgabe**  
Dokumentieren Sie den Create-Befehl der Stored Procedures *film_in_stock*.

Beschreibung:  
Diese SP sucht nach Filmkopien (eines bestimmten Films) die noch an Lager sind.

Parameter:
* p_film_id : Die ID des gesuchten Films
* p_store_id : Die ID des entsprechenden Lagers
* p_film_count : Die Anzahl der Filme

Return Values:  
Es wird eine Tabelle mit *inventory_id* Nummern generiert. Der Parameter *p_film_count* beinhaltet die totale Anzahl der Kopien.

**Stored Procedures erstellen**  
```sql
DELIMITER //

CREATE PROCEDURE film_in_stock (IN p_film_id INT, IN p_store_id INT, OUT p_film_count INT)
READS SQL DATA
BEGIN
    SELECT inventory_id
    FROM inventory
    WHERE film_id = p_film_id
    AND store_id = p_store_id
    AND inventory_in_stock(inventory_id);

    SELECT COUNT(*)
    FROM inventory
    WHERE film_id = p_film_id
    AND store_id = p_store_id
    AND inventory_in_stock(inventory_id)
    INTO p_film_count;
END //

DELIMITER ;
```

**Stored Procedures anzeigen**  
```sql
SHOW CREATE PROCEDURE film_in_stock;
```

**Stored Procedures ändern**  
```sql
ALTER PROCEDURE film_in_stock (characteristic ...);
```

**Stored Procedures löschen**  
```sql
DROP PROCEDURE IF EXISTS film_in_stock;
```

**Stored Procedures ausführen**  

```sql
CALL film_in_stock(1,1,@count);
```

Ausgabe:
```
+--------------+
| inventory_id |
+--------------+
|            1 |
|            2 |
|            3 |
|            4 |
+--------------+
```

```sql
SELECT @count;
```

Ausgabe:
```
+--------+
| @count |
+--------+
|      4 |
+--------+
```

### Views

Views sind Sichten auf Tabellen und andere Views. Mit Views lassen sich nur bestimmte Inhalte, welche mittels SELECT definiert wurden, anzeigen.

**Aufgabe**  
Dokumentieren Sie den Create-Befehl der View *actor_info*.

**View erstellen**  
```sql
CREATE OR REPLACE VIEW actor_info AS SELECT `a`.`actor_id` AS `actor_id`,`a`.`first_name` AS `first_name`,`a`.`last_name` AS `last_name`,group_concat(DISTINCT concat(`c`.`name`,': ',(SELECT group_concat(`f`.`title` ORDER BY `f`.`title` ASC separator ', ') FROM ((`film` `f` JOIN `film_category` `fc` ON((`f`.`film_id` = `fc`.`film_id`))) JOIN `film_actor` `fa` ON((`f`.`film_id` = `fa`.`film_id`))) WHERE ((`fc`.`category_id` = `c`.`category_id`) AND (`fa`.`actor_id` = `a`.`actor_id`)))) ORDER BY `c`.`name` ASC separator '; ') AS `film_info` FROM (((`actor` `a` LEFT JOIN `film_actor` `fa` ON((`a`.`actor_id` = `fa`.`actor_id`))) LEFT JOIN `film_category` `fc` ON((`fa`.`film_id` = `fc`.`film_id`))) LEFT JOIN `category` `c` ON((`fc`.`category_id` = `c`.`category_id`))) GROUP BY `a`.`actor_id`,`a`.`first_name`,`a`.`last_name`;
```

**View anzeigen**  
```sql
SHOW CREATE VIEW actor_info;
```

**View ändern**  
```sql
ALTER VIEW actor_info AS SELECT `a`.`actor_id` AS `actor_id`,`a`.`first_name` AS `first_name`,`a`.`last_name` AS `last_name`,group_concat(DISTINCT concat(`c`.`name`,': ',(SELECT group_concat(`f`.`title` ORDER BY `f`.`title` ASC separator ', ') FROM ((`film` `f` JOIN `film_category` `fc` ON((`f`.`film_id` = `fc`.`film_id`))) JOIN `film_actor` `fa` ON((`f`.`film_id` = `fa`.`film_id`))) WHERE ((`fc`.`category_id` = `c`.`category_id`) AND (`fa`.`actor_id` = `a`.`actor_id`)))) ORDER BY `c`.`name` ASC separator '; ') AS `film_info` FROM (((`actor` `a` LEFT JOIN `film_actor` `fa` ON((`a`.`actor_id` = `fa`.`actor_id`))) LEFT JOIN `film_category` `fc` ON((`fa`.`film_id` = `fc`.`film_id`))) LEFT JOIN `category` `c` ON((`fc`.`category_id` = `c`.`category_id`))) GROUP BY `a`.`actor_id`,`a`.`first_name`,`a`.`last_name`;
```

**View löschen**  
```sql
DROP VIEW IF EXISTS actor_info;
```

**View ausführen**  
```sql
SELECT * FROM actor_info;
```

### Triggers

Triggers sind Datenbank-Objekte, welche durch ein bestimmtes Ereignis ausgelöst werden können. So lassen sich Triggers für folgende Aktionen programmieren:
* Before Insert
* Before Update
* Before Delete
* After Insert
* After Update
* After Delete

**Trigger erstellen**  
Syntax:
```sql
CREATE TRIGGER trigger_name
{BEFORE | AFTER} {INSERT | UPDATE| DELETE }
ON table_name FOR EACH ROW
trigger_stmt
```

Beispiel:  
Wenn eine neue Filmkategorie hinzugefügt wird, wird auch ein neuer Eintrag bei den Verkäufen nach Filmkategorie hinzugefügt.
```sql
CREATE TRIGGER after_insert_category
AFTER INSERT
ON category FOR EACH ROW
INSERT INTO sales_by_film_category (category, total_sales) VALUES (new.name, 0);
```

**Trigger anzeigen**  
```sql
SHOW TRIGGERS IN sakila;
```
  
**Trigger löschen**  
```sql
DROP TRIGGER trigger_name;
```
