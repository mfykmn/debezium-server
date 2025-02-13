# Snowflakeの認証情報での疎通確認
Snowflake Sync Connectorでは認証にIDとパスワード or IDとキーペア認証の2種類のどちらかを選ぶ必要([参照](./Snowflake%20Sync%20Connecterの登録について.md))があります。

先にSnowflakeの認証情報を格納させて疎通確認を行いたい場合にどのようなリクエストが行えるかを確認してみます。

## [SQL REST API](https://docs.snowflake.com/ja/developer-guide/snowflake-rest-api/reference)

疎通に適したAPIは見当たらなかったので、何かしらのAPIを叩いて認証エラーが返ってこなければ疎通できているということにする。
こちらの方法ではキーペアでの認証を行える。
[JWTトークンを作成](https://docs.snowflake.com/en/developer-guide/sql-api/authenticating#label-sql-api-authenticating-key-pair)し認証を行う。

```shell
curl -i -X POST 
-H "Content-Type: application/json" 
-H "Authorization: Bearer <jwt>" 
-H "Accept: application/json" 
-H "User-Agent: myApplicationName/1.0" 
-H "X-Snowflake-Authorization-Token-Type: KEYPAIR_JWT" 
-d "@request-body.json" 
"https://<account_identifier>.snowflakecomputing.com/api/v2/statements"
```


## [Snow CD](https://docs.snowflake.com/ja/user-guide/snowcd)
ユーザーがSnowflakeへのネットワーク接続を診断およびトラブルシューティングするのに役立つCLIツール
ネットワーク接続の診断なのでこれは使えるかもしれないがIDとキーペア認証ができるかについては調査が必要。

```shell
snowcd allowlist.json \
--proxyHost <ホスト名> \
--proxyPort <ポート番号> \
--proxyUser <ユーザー名> \
--proxyPassword <パスワード> \
--logLevel trace \
--logPath test.log
```

## 参考リンク
- [SnowflakeのSQL REST APIを呼び出してみた](https://note.com/bunsekiya_tech/n/n1fb256dd0e08)
