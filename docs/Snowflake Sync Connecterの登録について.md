# Snowflake Sync Connectorの登録について
バッチモードとストリーミングモードがあり、バッチモードではSnowpipeを使用してデータをバッチでロードし、ストリーミングモードではSnowpipe Streamingを使用してデータをストリーミングでロードします。

## Snowflake Sync Connectorの登録
登録方法に関しては、Kafka Connectに対してPOSTのAPIリクエストにconfgを付与して行うことで登録することができます。
今回はSnowflakeSinkConnectorを使用するため事前にKafka Connectの環境でSnowflakeSinkConnectorのJarをダウンロードしておく必要があります。

## 登録時のconfig例
動作させるために必要最低限なconfigを記載します。(必須値のみというわけではありません)
### バッチモード
```json
{
    "name": "snowflake-sink-connector-batch",
    "config": {
        "connector.class": "com.snowflake.kafka.connector.SnowflakeSinkConnector",
        "topics": "mysql.test_cdc.customers",
        "snowflake.url.name": "https://<your account>.snowflakecomputing.com",
        "snowflake.user.name": "jane.smith",
        "snowflake.private.key": "<private_key>",
        "snowflake.database.name":"kafka_db",
        "snowflake.schema.name": "kafka_schema",
        "tasks.max": 1,
        "snowflake.topic2table.map": "mysql.test_cdc.customers:customers_batch",
        "buffer.flush.time": 10,
        "buffer.count.records": 5,
        "buffer.size.bytes": 1000,
        "key.converter": "org.apache.kafka.connect.storage.StringConverter",
        "value.converter": "org.apache.kafka.connect.json.JsonConverter"
    }
}
```

### ストリーミングモード
```json
{
    "name": "snowflake-sink-connector-streaming",
    "config": {
        "connector.class": "com.snowflake.kafka.connector.SnowflakeSinkConnector",
        "topics": "mysql.test_cdc.customers",
        "snowflake.url.name": "https://<your account>.snowflakecomputing.com",
        "snowflake.user.name": "jane.smith",
        "snowflake.private.key": "<your private key>",
        "snowflake.database.name":"kafka_db",
        "snowflake.schema.name": "kafka_schema",
        "tasks.max": 1,
        "snowflake.topic2table.map": "mysql.test_cdc.customers:customers_streaming",
        "snowflake.role.name": "kafka_connector_role_1",
        "enable.streaming.client.optimization": true,
        "buffer.flush.time": 1,
        "buffer.count.records": 1,
        "buffer.size.bytes": 100,
        "snowflake.streaming.max.client.lag": 1,
        "snowflake.streaming.enable.single.buffer": true,
        "key.converter": "org.apache.kafka.connect.storage.StringConverter",
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "snowflake.ingestion.method": "SNOWPIPE_STREAMING",
        "snowflake.enable.schematization" : true
    }
}

```

## config例の設定値について
#### snowflake.user.name, snowflake.private.key
認証はユーザーによるIDとパスワード or IDとキーペア認証の2種類があります。
例えばパスワードとして違う値を入れたなどで認証が失敗する場合はコネクタのPOSTやPUT自体が失敗し、400エラーが返されます。

#### snowflake.role.name
バッチモードではユーザーに付与されているロールの権限が自動作成したテーブルに付与されますが、ストリーミングモードではroleを指定してその権限を付与することができます。

#### snowflake.database.name, snowflake.schema.name
Snowflakeのデータベースとスキーマを指定するこのオプションは必須値のため、事前にデータベースとスキーマを作成する必要があります。

#### snowflake.topic2table.map
kafkaのトピック:Snowflakeのテーブル名 のような指定の仕方で、指定したテーブルが存在しない場合は自動で作成します。

#### key.converter, value.converter
現状では使用していませんが、Avroという形式が使用できます。ただし、こちらを利用する場合は環境にスキーマレジストラを立ててKafka Connectと接続できるようにしておく必要があります。

## その他の設定値について
以下の重用そうな設定値については公式ドキュメントを参照してください。
- スタンドアローンモードと分散モード
- Converter と Transform
- Task
- Avroとスキーマレジストラ
- デッドレターキュー

## 参考リンク
- [Batch、Streaming共通設定値について](https://docs.snowflake.com/en/user-guide/kafka-connector-install#configuring-the-kafka-connector)
- [Streamingで使用する設定値について](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-kafka)
- [Snowpipe Streaming best practices](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-streaming-recommendation)
