# CodeGeneratorの使い方

BusinessCardManagementにはh2db起動時に自動でデータロードはされないが
コードジェネレーターを使うことで想定したデータを入れつつIFコードもつくれる

## 基本の使い方

`com/example/mnaganu/bcm/common/CodeGeneratorTest.java`

ユニットテストを実行することで

1. IntelliJで business-card-manegement を起動する

しておく

2. ユニットテスト実行する

マスターとするTableスキーマSQLが文字列として代入されている

https://github.com/mnaganu/business-card-management/blob/main/src/test/java/com/example/mnaganu/bcm/common/CodeGeneratorTest.java#L10-L25

CodeGeneratorTest
クラス名を選択した状態で Ctrl + Shift + F10 押下してテスト全実行する

3. 生成コードを取得する

consoleログへ以下のようなコードがまとめて吐かれるので
既存のコードと置き換えたりSQLは必要に応じてh2-consoleで叩くなどして用いる

```
# テーブルが生成される際のスキーマ情報オブジェクト一覧
CreateTableModel

# ひとつめのSQLは素のupdate文
UPDATE

# ふたつめのSQLはStringにアサインしてNamedPraseHolderを使う時に用いるudpate文
UPDATE

# ひとつめのSQLは素のinsert文
INSERT

# ふたつめのSQLはStringにアサインしてNamedPraseHolderを使う時に用いるinsert文
INSERT

# ひとつめのSQLは素のselect文
SELECT

# ふたつめのSQLはStringにアサインしてNamedPraseHolderを使う時に用いるselect文
SELECT

# みっつめのSQLはカラム名の前にプレフィックスを付けるときの素のselect文
SELECT

# よっつめのSQLはカラム名の前にプレフィックスを付けてStringにアサインしてNamedPraseHolderを使う時に用いるselect文
SELECT

# スキーマ情報から生成されたcom.example.mnaganu.bcm.domain.model.BusinessCardModelドメインモデルソースコード
public class BusinessCardModel

# スキーマ情報から生成されたcom.example.mnaganu.bcm.infrastructure.handler.BusinessCardRowCountCallbackHandlerインフラハンドラソースコード
public class BusinessCardRowCountCallbackHandler extends RowCountCallbackHandler

# スキーマ情報から生成されたcom.example.mnaganu.bcm.infrastructure.handler.BusinessCardRowCountCallbackHandlerインフラハンドラソースコード（テーブル名プレフィックス付き）
public class BusinessCardRowCountCallbackHandler extends RowCountCallbackHandler

# スキーマ情報から生成されたcom.example.mnaganu.bcm.infrastructure.mapper.BusinessCardRowMapperインフラマッパーソースコード
public class BusinessCardRowMapper implements RowMapper<BusinessCardModel>

# スキーマ情報から生成されたcom.example.mnaganu.bcm.infrastructure.mapper.BusinessCardRowMapperインフラマッパーソースコード（テーブル名プレフィックス付き）
public class BusinessCardRowMapper implements RowMapper<BusinessCardModel>
```

## スキーマ変更したいとき

まず、CodeGeneratorTest.java 上でスキーマ情報を修正してコード生成し
各種インフラ系・モデル系のソースコードを置き換えることですぐにカラム追加削除などが可能