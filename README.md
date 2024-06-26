# ccx-notification-writer
CCX notification writer service

## Usage

```
  -authors
        show authors
  -check-kafka
        check connection to Kafka
  -db-cleanup
        perform database cleanup
  -db-drop-tables
        drop all tables from database
  -db-init
        perform database initialization
  -show-configuration
        show configuration
  -version
        show version
```

## Metrics

### Exposed metrics

* `notification_writer_check_last_checked_timestamp`
    - The total number of messages with last checked timestamp
* `notification_writer_check_schema_version`
    - The total number of messages with successfull schema check
* `notification_writer_consumed_messages`
    - The total number of messages consumed from Kafka
* `notification_writer_consuming_errors`
    - The total number of errors during consuming messages from Kafka
* `notification_writer_marshal_report`
    - The total number of marshaled reports
* `notification_writer_parse_incoming_message`
    - The total number of parsed messages
* `notification_writer_shrink_report`
    - The total number of shrinked reports
* `notification_writer_stored_messages`
    - The total number of messages stored into database
* `notification_writer_stored_bytes`
    - The total number of bytes stored into database

### Retriewing metrics

```
curl localhost:8080/metrics | grep ^notification_writer
```

## Database

PostgreSQL database is used as a storage.

### Check PostgreSQL status

```
service postgresql status
```

### Start PostgreSQL database

```
sudo service postgresql start
```

### Login into the database

```
psql --user postgres
```

List all databases:

```
\l
```

Select the right database:

```
\c notification
```

List of tables:

```
\dt

               List of relations
 Schema |        Name        | Type  |  Owner
--------+--------------------+-------+----------
 public | new_reports        | table | postgres
 public | notification_types | table | postgres
 public | reported           | table | postgres
 public | states             | table | postgres
(4 rows)
```

## Database schema

### Table `new_reports`

```
   Column     |            Type             | Modifiers
--------------+-----------------------------+-----------
 org_id       | integer                     | not null
 account_id   | integer                     | not null
 cluster      | character(36)               | not null
 report       | character varying           | not null
 updated_at   | timestamp without time zone | not null
 kafka_offset | bigint                      | not null default 0
Indexes:
    "new_reports_pkey" PRIMARY KEY, btree (org_id, cluster, updated_at)
    "report_kafka_offset_btree_idx" btree (kafka_offset)
```

### Table `states`

```
          Table "public.states"
 Column  |       Type        | Modifiers
---------+-------------------+-----------
 id      | integer           | not null
 value   | character varying | not null
 comment | character varying |
Indexes:
    "states_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "reported" CONSTRAINT "fk_state" FOREIGN KEY (state) REFERENCES states(id)
```

### Table `reported`

```
      Column       |            Type             | Modifiers
-------------------+-----------------------------+-----------
 org_id            | integer                     | not null
 account_id        | integer                     | not null
 cluster           | character(36)               | not null
 notification_type | integer                     | not null
 state             | integer                     | not null
 report            | character varying           | not null
 updated_at        | timestamp without time zone | not null
Indexes:
    "reported_pkey" PRIMARY KEY, btree (org_id, cluster)
Foreign-key constraints:
    "fk_notification_type" FOREIGN KEY (notification_type) REFERENCES notification_types(id)
    "fk_state" FOREIGN KEY (state) REFERENCES states(id)
```

### Table `notification_types`

```
  Column   |       Type        | Modifiers
-----------+-------------------+-----------
 id        | integer           | not null
 value     | character varying | not null
 frequency | character varying | not null
 comment   | character varying |
Indexes:
    "notification_types_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "reported" CONSTRAINT "fk_notification_type" FOREIGN KEY (notification_type) REFERENCES notification_types(id)
```
