# Debezium Server
## 環境構築
1. debezium-server以外を起動
    ```shell
    docker compose up -d mysql kafka zookeeper kafdrop
    ```
2. スナップショット取得のためdebezium-serverを起動
    memo `docker compose up -d`で一斉に起動するとタイミングのせいかスナップショットが取れないので分けて起動する
    ```shell
    cp /dev/null ./debezium/data/offsets.dat
    docker compose up -d debezium-server
    ```
3. スナップショットがKafkaに流れているのか確認
    ```shell
    open http://localhost:9000/

    or

    docker exec -it kafka kafka-topics.sh --list --bootstrap-server kafka:9092
    docker exec -it kafka kafka-console-consumer.sh --bootstrap-server kafka:9092 --from-beginning --topic mysql.test_cdc.customers
    ```
4. データ挿入
    ```shell
    docker compose exec mysql bash -c 'mysql -u mysqluser -pmysqlpassword test_cdc'
    mysql > INSERT INTO customers (name) VALUES ("fuga");
    ```
3. データ挿入した変更点がKafkaに流れているのか確認
    ```shell
    open http://localhost:9000/

    or

    docker exec -it kafka kafka-topics.sh --list --bootstrap-server kafka:9092
    docker exec -it kafka kafka-console-consumer.sh --bootstrap-server kafka:9092 --from-beginning --topic mysql.test_cdc.customers
    ```

## 参考リンク
### Debezium
#### Debezium Server
- https://debezium.io/documentation/reference/stable/operations/debezium-server.html
- https://github.com/debezium/debezium-examples/tree/main/debezium-server
- https://debezium.io/documentation/reference/stable/operations/debezium-server.html#_apache_kafka
    Kafkaにsinkする場合の資料

### Snowflake
#### Snowflake Sink Connect
- https://docs.confluent.io/cloud/current/connectors/cc-snowflake-sink/cc-snowflake-sink.html  
- https://github.com/snowflakedb/snowflake-kafka-connector/tree/master
#### Kafka Connector with Snowpipe Streaming
  - https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-kafka
#### KafkaConnector
- Snowflakeにsinkする場合の資料
  - https://docs.snowflake.com/en/user-guide/kafka-connector
- 一連の設定方法などが全部書いてある
  - https://docs.snowflake.com/en/user-guide/kafka-connector-install
- Kafkaコネクタファイルのダウンロード先
  - https://www.confluent.io/hub/snowflakeinc/snowflake-kafka-connector
  - https://mvnrepository.com/artifact/com.snowflake/snowflake-kafka-connector
- Kafkaコネクタの設定
  - https://docs.snowflake.com/en/user-guide/kafka-connector-install#configuring-the-kafka-connector
