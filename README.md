# Debezium Server
## 環境構築
### debeziumでCDCを行いKafkaにデータを流す
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
5. データ挿入した変更点がKafkaに流れているのか確認
    ```shell
    open http://localhost:9000/

    or

    docker exec -it kafka kafka-topics.sh --list --bootstrap-server kafka:9092
    docker exec -it kafka kafka-console-consumer.sh --bootstrap-server kafka:9092 --from-beginning --topic mysql.test_cdc.customers
    ```
###  kafka-connectを使ってKafkaのデータをSnowflakeに流す
1. kafka-connectを起動
    ```shell
    docker compose up -d kafka-connect
    docker logs kafka-connect -f # 起動に時間がかかるのでログを見て起動したか確認する "GET /connectors HTTP/1.1" 200" のようなログがでてきたら次の作業に移る
    ```
2. Snowflake sink connectorを登録
#### Snowpipe Streaming(ストリーミング)の場合はこちら
```shell
curl -XPOST http://localhost:8083/connectors -H "Content-Type: application/json" -d @kafka-connect/config/SF_streaming.json
curl -XGET http://localhost:8083/connectors # コネクタの登録ができているか確認
["snowflake-sink-connector"]
```
#### Snowpipe Batch(バッチ)の場合はこちら
```shell
curl -XPOST http://localhost:8083/connectors -H "Content-Type: application/json" -d @kafka-connect/config/SF_batch.json
curl -XGET http://localhost:8083/connectors # コネクタの登録ができているか確認
["snowflake-sink-connector"]
```

※もし失敗してやり直したい場合はdocker storageのデータを削除する

3. データ挿入してsnowflakeにデータが流れているか確認
    ```shell
    docker compose exec mysql bash -c 'mysql -u mysqluser -pmysqlpassword test_cdc'
    mysql > INSERT INTO customers (name) VALUES ("snowflake");
    open https://app.snowflake.com/ # 見たいテーブルまで行き、テーブルの権限を確認し問題なければ DataPreviewで確認する
    ```

## TODO
- kafka connectのスタンドアローンモードと分散モード
- Avroとスキーマレジストラ

## 参考リンク
### Debezium
#### Debezium Server
- https://debezium.io/documentation/reference/stable/operations/debezium-server.html
- https://github.com/debezium/debezium-examples/tree/main/debezium-server
- https://debezium.io/documentation/reference/stable/operations/debezium-server.html#_apache_kafka
    Kafkaにsinkする場合の資料

### Snowflake
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
- 使っているDockerImage参考
  - https://hub.docker.com/r/confluentinc/cp-kafka-connect
#### Kafka Connector with Snowpipe Streaming
  - https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-kafka
  - https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-recommendation
#### Snowflake Sink Connect
- https://docs.confluent.io/cloud/current/connectors/cc-snowflake-sink/cc-snowflake-sink.html  
- https://github.com/snowflakedb/snowflake-kafka-connector/tree/master
- https://docs.confluent.io/confluent-cli/current/command-reference/connect/cluster/confluent_connect_cluster_create.html
- コネクターの追加
  - https://docs.confluent.io/platform/current/connect/references/restapi.html#post--connectors
