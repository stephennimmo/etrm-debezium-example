# etrm-debezium-example

# Setup

podman and podman-compose - https://podman.io
```
brew install podman
brew install podman-compose
or..
brew install --cask podman-desktop
```
kcctl - https://github.com/kcctl/kcctl
```
brew install kcctl/tap/kcctl
```

```
kcctl config set-context local --cluster=http://localhost:8083
```

# Instructions

```
podman-compose up -d
```

## Create the Debezium Kafka Connector
```
kcctl apply -f etrm-connector.json
```

# Open Kafka Console Consumer to view messages
```
podman exec -it kafka /bin/bash
...
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --whitelist '.*etrm.*'
```

## Open Database Console and Tests

```
podman exec -it postgres /bin/bash
...
psql -d etrm -U admin
```

Insert into just trade header
```
insert into trade_header (start_date, end_date, execution_timestamp, trade_type_id) VALUES (CURRENT_DATE - 30, CURRENT_DATE + 5, CURRENT_TIMESTAMP, 1);
```

Insert into trade_header and trade_leg in single transaction (most likely)
```
BEGIN;
WITH new_trade AS (
    insert into trade_header (start_date, end_date, execution_timestamp, trade_type_id) VALUES (CURRENT_DATE - 30, CURRENT_DATE + 5, CURRENT_TIMESTAMP, 1)
        RETURNING trade_id
)

INSERT INTO trade_leg (trade_id, payer_id, receiver_id, price, price_currency, quantity, quantity_uom_id)
VALUES ((select trade_id from new_trade), 1, 2, 2.84, 1, 10000, 1),
       ((select trade_id from new_trade), 2, 1, 2.84, 1, 10000, 1);
COMMIT;
```

## Cleanup
```
podman-compose down
```

# Links

https://hub.docker.com/_/postgres

# Backlog

JsonConverter
https://gist.githubusercontent.com/yildirimabdullah/eb54f3386a37acbcb07f68c1bb13a15e/raw/a928622039ffb08284bba8674b52c1eb2fc1a653/create-connector.sh

We were producing JSON messages with schemas enabled; this creates larger Kafka records than needed, in particular if schema changes are rare. Hence we decided to disable message schemas by setting key.converter.schemas.enabled and value.converter.schemas.enabled to false to reduce the size of each payload considerably hence saving on network bandwidth and serialization/deserialization costs. The only downside is that we now need to maintain the schema of those messages in an external schema registry.
https://debezium.io/blog/2020/02/25/lessons-learned-running-debezium-with-postgresql-on-rds/


https://debezium.io/documentation/reference/stable/operations/debezium-ui.html





