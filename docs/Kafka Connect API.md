# Kafka Connect API
## [Connect Cluster](https://docs.confluent.io/platform/current/connect/references/restapi.html#kconnect-cluster)

### GET /

```shell
curl -XGET http://localhost:8083/
```
```json
{"version":"7.8.0-ccs","commit":"3843870745f85813","kafka_cluster_id":"Qf4-49g3Qx61x7Q731m59A"}
```

## [Connectors](https://docs.confluent.io/platform/current/connect/references/restapi.html#connectors)

### GET /connectors
アクティブなコネクタのリストを取得する

```shell
curl -XGET http://localhost:8083/connectors
```
```json
["snowflake-sink-connector-streaming","snowflake-sink-connector-batch"]
```
オプションでstatusも取得できる
```shell
curl -XGET "http://localhost:8083/connectors?expand=status"
```
```json
{"snowflake-sink-connector-batch":{"status":{"name":"snowflake-sink-connector-batch","connector":{"state":"RUNNING","worker_id":"connect:8083"},"tasks":[{"id":0,"state":"RUNNING","worker_id":"connect:8083"}],"type":"sink"}},"snowflake-sink-connector-streaming":{"status":{"name":"snowflake-sink-connector-streaming","connector":{"state":"RUNNING","worker_id":"connect:8083"},"tasks":[{"id":0,"state":"RUNNING","worker_id":"connect:8083"}],"type":"sink"}}}
```

?expand=infoもある

### POST /connectors
新しいコネクタを作成し、成功した場合は現在のコネクタ情報を返します。 再バランス処理中の場合、またはコネクタが既に存在する場合は返します。
```shell
curl -XPOST http://localhost:8083/connectors -H "Content-Type: application/json" -d @kafka-connect/config/SF_streaming.json
```

### GET /connectors/(string:name)
コネクタに関する情報を取得します。

応答 JSON オブジェクト:	
- name (文字列) – 作成されたコネクタの名前
- config ( map ) – コネクタの設定パラメータ
- タスク(配列) – コネクタによって生成されたアクティブなタスクのリスト
- tasks[i].connector (文字列) – タスクが属するコネクタの名前
- tasks[i].task ( int ) – コネクタ内のタスクID
```shell
curl -XGET http://localhost:8083/connectors/snowflake-sink-connector-batch
```
※jsonは接続情報が乗るため貼らない
### GET /connectors/(string:name)/config
コネクタの構成を取得します
```shell
curl -XGET http://localhost:8083/connectors/snowflake-sink-connector-batch/config
```
※jsonは接続情報が乗るため貼らない
### PUT /connectors/(string:name)/config
指定された構成を使用して新しいコネクタを作成するか、既存のコネクタの構成を更新します。変更が行われた後のコネクタに関する情報を返します
```shell
curl -XPUT http://localhost:8083/connectors/snowflake-sink-connector-batch/config \
-H "Content-Type: application/json" \
-d $(jq  --monochrome-output --compact-output .config kafka-connect/config/SF_batch.json)
```
### GET /connectors/(string:name)/status
以下の情報を含むコネクタの現在のステータスを取得します。

- 実行中か再起動中か、失敗しているか一時停止しているか
- どの作業者に割り当てられているか
- 失敗した場合のエラー情報
- すべてのタスクの状態
```shell
curl -XGET http://localhost:8083/connectors/snowflake-sink-connector-batch/status
```
```json
{"name":"snowflake-sink-connector-batch","connector":{"state":"RUNNING","worker_id":"connect:8083"},"tasks":[{"id":0,"state":"RUNNING","worker_id":"connect:8083"}],"type":"sink"}
```
### POST /connectors/(string:name)/restart
コネクタを再起動します
クエリパラメーターのオプションがある
```shell
curl -XPOST http://localhost:8083/connectors/snowflake-sink-connector-batch/restart
```

### PUT /connectors/(string:name)/pause
コネクタとそのタスクを一時停止します。これにより、コネクタが再開されるまでメッセージの処理が停止します
stateがPAUSEDになる
```shell
curl -XPUT http://localhost:8083/connectors/snowflake-sink-connector-batch/pause
```

### PUT /connectors/(string:name)/resume
一時停止されたコネクタを再開するか、コネクタが一時停止されていない場合は何も行いません。
stateがSTOPPEDからRUNNINGになる
```shell
curl -XPUT http://localhost:8083/connectors/snowflake-sink-connector-batch/resume
```

### PUT /connectors/(string:name)/stop
コネクタを停止しますが、コネクタは削除しません。コネクタのすべてのタスクは完全にシャットダウンされます。停止したコネクタを再開すると、コネクタは割り当てられたワーカーで起動します。
stateがSTOPPEDになる
```shell
curl -XPUT http://localhost:8083/connectors/snowflake-sink-connector-batch/stop
```
### DELETE /connectors/(string:name)/
コネクタの削除
```shell
curl -XDELETE http://localhost:8083/connectors/snowflake-sink-connector-batch
```

## [Tasks](https://docs.confluent.io/platform/current/connect/references/restapi.html#tasks)
### GET /connectors/(string:name)/tasks
### GET /connectors/(string:name)/tasks/(int:taskid)/status
### POST /connectors/(string:name)/tasks/(int:taskid)/restart

## [Topics](https://docs.confluent.io/platform/current/connect/references/restapi.html#topics)
### GET /connectors/(string:name)/topics
コネクタ トピック名のリストを返します
```shell
curl -XGET http://localhost:8083/connectors/snowflake-sink-connector-batch/topics
```
```json
{"snowflake-sink-connector":{"topics":["mysql.test_cdc.customers"]}}
```
### PUT /connectors/(string:name)/topics/reset
コネクタの作成以降、またはアクティブなトピックのセットが最後にリセットされてからコネクタが使用しているトピック名のセットをリセットします

## [Offsets](https://docs.confluent.io/platform/current/connect/references/restapi.html#offsets)
### GET /connectors/{connector}/offsets
```shell
curl -XGET http://localhost:8083/connectors/snowflake-sink-connector-batch/offsets
```
```json
{"offsets":[{"partition":{"kafka_partition":0,"kafka_topic":"mysql.test_cdc.customers"},"offset":{"kafka_offset":32}}]}
```

### PATCH /connectors/{connector}/offsets
コネクタのオフセットを変更します。コネクタが存在し、停止状態になっている必要があることに注意してください
### DELETE /connectors/{connector}/offsets
コネクタのオフセットをリセットします。コネクタが存在し、停止 状態になっている必要があることに注意してください。
### PUT /connectors/{connector}/stop
コネクタを停止し、そのタスクをシャットダウンしますが、削除しないでください。

## [Connector plugins](https://docs.confluent.io/platform/current/connect/references/restapi.html#connector-plugins)

### GET /connector-plugins/
Kafka Connect クラスターにインストールされているコネクタ プラグインと (オプションで) 変換のリストを返します
```shell
curl -XGET http://localhost:8083/connector-plugins/
```
```json
[{"class":"com.snowflake.kafka.connector.SnowflakeSinkConnector","type":"sink","version":"3.1.0"},{"class":"org.apache.kafka.connect.mirror.MirrorCheckpointConnector","type":"source","version":"7.8.0-ccs"},{"class":"org.apache.kafka.connect.mirror.MirrorHeartbeatConnector","type":"source","version":"7.8.0-ccs"},{"class":"org.apache.kafka.connect.mirror.MirrorSourceConnector","type":"source","version":"7.8.0-ccs"}]
```

### PUT /connector-plugins/(string:name)/config/validate
提供された構成値を構成定義に対して検証します。この API は構成ごとに検証を実行し、検証中に推奨値とエラー メッセージを返します
```shell
curl -XPUT http://localhost:8083/connector-plugins/SnowflakeSinkConnector/config/validate/ \
-H "Content-Type: application/json" \
-d $(jq  --monochrome-output --compact-output .config kafka-connect/config/SF_batch.json) | jq .error_count
```

```json
1
```



