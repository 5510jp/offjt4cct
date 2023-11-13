# IntelliJで business-card-manegement を起動する

もし時間があれば以下を読まずに自力で調べながらやってみるのがおススメです


## プロジェクトをIDE開く

1. git cloneしてIntelliJ>File>Openよりディレクトリを選択して開く

2. IntelliJ>Build>BuildProjectでいったんビルドしてみる

2.1. 必要なJavaバージョンがありません、のエラーした場合

JavaSDKがありませんといったエラーメッセージを得た際は
まずbuild.gradleファイルを確認して必要なJavaSDKバージョンを確認する

2.2. SDKをダウンロードする

IntelliJ上で使うSDKバージョンの設定メニューからダウンロードもできるので、まず
File>ProjectStructure>Project>SDK
Add SDK
から前述で確認しておいたバージョンに沿うJavaSDKを得る

どのリビジョンでもほぼ代わらないがtemurinは大丈夫そう

2.3. gradleにもこのバージョンを反映しないとエラーする

```
Could not resolve org.springframework.boot:spring-boot-gradle-plugin:3.0.5 gradle
```

こうエラーが出た場合は以下を設定してからビルドし直す
File>Settings>Build, Execution, Deployment>Build Tools>Gradle
Gradel JVM
こちらもさきほど仕入れたSDKに変えておく

3. アプリケーションを実行

```
com/example/mnaganu/bcm/BusinessCardManagementApplication.java
```

mainメソッドの左にある再生マークを選択してアプリケーション実行

いったんSpringBoot起動まではいけるはず
API叩けるけどDBが無い旨のエラーする状態

## DBセットアップする

1. application.ymlでh2コンソール起動させる

```yaml
  h2:
    console:
      enabled: true
```

上記を書き加えて、アプリケーションをリスタート

2. ブラウザで以下URLへアクセスしymlに書かれてある通り入力してログイン

http://localhost:8080/h2-console/


3. DBセットアップ

DBはある状態だからTableをつくる
以下のSQLをRun実行する

```sql
CREATE TABLE `business_card` (
  `id` int NOT NULL AUTO_INCREMENT,
  `company_name` varchar(255) DEFAULT NULL COMMENT '会社名',
  `depaertment_name` varchar(255) DEFAULT NULL COMMENT '部署名',
  `post` varchar(255) DEFAULT NULL COMMENT '役職',
  `name` varchar(255) NOT NULL COMMENT '名前',
  `postal_code` varchar(8) DEFAULT NULL COMMENT '郵便番号',
  `address` varchar(255) DEFAULT NULL COMMENT '住所',
  `phone_number` varchar(255) DEFAULT NULL COMMENT '電話番号',
  `fax` varchar(255) DEFAULT NULL COMMENT 'fax',
  `note` varchar(255) DEFAULT NULL COMMENT '備考',
  `create_time` datetime DEFAULT NULL COMMENT 'データ作成日',
  `update_time` datetime DEFAULT NULL COMMENT 'データ更新日',
  `delete_time` datetime DEFAULT NULL COMMENT 'データ削除日',
  PRIMARY KEY (`id`)
)
```

## CRUD API実行確認しつつデータ投入

1. Create(作成)を試す

以下をPOSTメソッドで投入する

request_body.json
```json
{
    "companyName": "hogesha",
    "depaertmentName": "kanribu",
    "post": "bucho",
    "name": "tanaka ichiro",
    "postalCode": "000-0000",
    "address": "jusho",
    "phoneNumber": "000-0000-0000",
    "fax": "00-0000-0000",
    "note": "note"
}
```

```bash
$ curl -X POST -H "Content-Type: application/json" -d @request_body.json localhost:8080/api/v1/business-card
```


2. Read(参照)を試す

以下のとおりGETメソッドでリクエストする

全件
```
$ curl -X GET localhost:8080/api/v1/business-card
```

id指定の1件
```
$ curl -X GET localhost:8080/api/v1/business-card/1
```

3. Update(更新)を試す

以下をPUTメソッドで投入する

request_body.json
```json
{
    "id": 1,
    "companyName": "hogesha",
    "depaertmentName": "yakuin",
    "post": "shacho",
    "name": "tanaka ichiro",
    "postalCode": "000-0000",
    "address": "jusho",
    "phoneNumber": "000-0000-0000",
    "fax": "00-0000-0000",
    "note": "note2"
}
```

```
$ curl -X PUT -H "Content-Type: application/json" -d @request_body.json localhost:8080/api/v1/business-card
```

4. Delete(削除)を試す

以下のとおりDELETEメソッドでリクエストする

id指定で1件削除

```
$ curl -X DELETE -H localhost:8080/api/v1/business-card/1
```
