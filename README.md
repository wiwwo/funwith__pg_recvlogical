Startup
```
$ docker-compose up -d
```

Play with it:
```
$ docker exec -it funwith__pg_recvlogical_pg_master_1 psql -h funwith__pg_recvlogical_pg_master_1 -Umyuser -dpostgres -p5432
```

Copy-paste this in terminal:

```
drop table if exists table2_with_pk;
drop table if exists table2_without_pk;
CREATE TABLE table2_with_pk (a SERIAL, b VARCHAR(30), c TIMESTAMP NOT NULL, PRIMARY KEY(a, c));
CREATE TABLE table2_without_pk (a SERIAL, b NUMERIC(5,2), c TEXT);

SELECT 'init' FROM pg_create_logical_replication_slot('test_slot', 'wal2json');

BEGIN;
INSERT INTO table2_with_pk (b, c) VALUES('Backup and Restore', now());
INSERT INTO table2_with_pk (b, c) VALUES('Tuning', now());
INSERT INTO table2_with_pk (b, c) VALUES('Replica', now());
UPDATE table2_with_pk SET b = 'Replication' WHERE a=3;
DELETE FROM table2_with_pk WHERE a < 3;

INSERT INTO table2_without_pk (b, c) VALUES(2.34, 'Tapir');

UPDATE table2_without_pk SET c = 'Anta' WHERE c = 'Tapir';
COMMIT;



SELECT data FROM pg_logical_slot_get_changes('test_slot', NULL, NULL, 'format-version', '2');
SELECT 'stop' FROM pg_drop_replication_slot('test_slot');
```


Example "screenshot":

```
$ docker exec -it pg_recvlogical_pg_master_1 psql -h pg_recvlogical_pg_master_1 -Umyuser -dpostgres -p5432
psql (13.4 (Debian 13.4-1.pgdg110+1))
Type "help" for help.

postgres=# CREATE TABLE table2_with_pk (a SERIAL, b VARCHAR(30), c TIMESTAMP NOT NULL, PRIMARY KEY(a, c));
CREATE TABLE table2_without_pk (a SERIAL, b NUMERIC(5,2), c TEXT);

SELECT 'init' FROM pg_create_logical_replication_slot('test_slot', 'wal2json');

BEGIN;
INSERT INTO table2_with_pk (b, c) VALUES('Backup and Restore', now());
INSERT INTO table2_with_pk (b, c) VALUES('Tuning', now());
INSERT INTO table2_with_pk (b, c) VALUES('Replication', now());
DELETE FROM table2_with_pk WHERE a < 3;

INSERT INTO table2_without_pk (b, c) VALUES(2.34, 'Tapir');
-- it is not added to stream because there isn't a pk or a replica identity
UPDATE table2_without_pk SET c = 'Anta' WHERE c = 'Tapir';
COMMIT;
CREATE TABLE
CREATE TABLE
 ?column?
----------
 init
(1 row)

BEGIN
INSERT 0 1
INSERT 0 1
INSERT 0 1
DELETE 2
INSERT 0 1
UPDATE 1
COMMIT
postgres=#
postgres=#
postgres=# SELECT data FROM pg_logical_slot_get_changes('test_slot', NULL, NULL, 'format-version', '2');
WARNING:  no tuple identifier for UPDATE in table "public"."table2_without_pk"
                                                                                                                                     data

-----------------------------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------------
 {"action":"B"}
 {"action":"I","schema":"public","table":"table2_with_pk","columns":[{"name":"a","type":"integer","value":1},{"name":"b","type":"character varying(30)","va
lue":"Backup and Restore"},{"name":"c","type":"timestamp without time zone","value":"2021-09-30 07:29:34.799976"}]}
 {"action":"I","schema":"public","table":"table2_with_pk","columns":[{"name":"a","type":"integer","value":2},{"name":"b","type":"character varying(30)","va
lue":"Tuning"},{"name":"c","type":"timestamp without time zone","value":"2021-09-30 07:29:34.799976"}]}
 {"action":"I","schema":"public","table":"table2_with_pk","columns":[{"name":"a","type":"integer","value":3},{"name":"b","type":"character varying(30)","va
lue":"Replication"},{"name":"c","type":"timestamp without time zone","value":"2021-09-30 07:29:34.799976"}]}
 {"action":"D","schema":"public","table":"table2_with_pk","identity":[{"name":"a","type":"integer","value":1},{"name":"c","type":"timestamp without time zo
ne","value":"2021-09-30 07:29:34.799976"}]}
 {"action":"D","schema":"public","table":"table2_with_pk","identity":[{"name":"a","type":"integer","value":2},{"name":"c","type":"timestamp without time zo
ne","value":"2021-09-30 07:29:34.799976"}]}
 {"action":"I","schema":"public","table":"table2_without_pk","columns":[{"name":"a","type":"integer","value":1},{"name":"b","type":"numeric(5,2)","value":2
.34},{"name":"c","type":"text","value":"Tapir"}]}
 {"action":"C"}
(8 rows)

postgres=# SELECT 'stop' FROM pg_drop_replication_slot('test_slot');
 ?column?
----------
 stop
(1 row)

postgres=#
```
