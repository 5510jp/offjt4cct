# DB初歩

## 前提

- 基本的なJava言語仕様の理解（型、文字列、演算子、判定、制御構造、配列、インスタンスとメソッド）
- 以下を一通り実施して環境構築もされていること
  - [SpringBoot HelloWorld (Eclips)](./springboot_helloworld.md)
- 以下を理解できること
  - 簡単なSQL文の使い方
  - Java メソッドチェインの使い方
  - Java Optionalクラスの使い方
  - Java JUnitの使い方
  - Java Builderパターンの概要
  - Java SpringFramework JdbcTemplateの概要

## プロジェクト作成

1. 「ファイル」->「新規」->「Springスターター・プロジェクト」を選択

2. タイプを「Maven」、Javaバージョンを「8」にして「次へ」を選択

以降、名前を「animal-db-project」グループを「com.sample」としたとして説明します

3. 以下を選択して「完了」を選択

- Spring Web
- Spring Boot DevTools
- Spring Data JDBC
- H2 Database

4. プロジェクトが自動作成される

5. application.properties をYAML形式に変えておく

src/main/resources/application.properties

右クリックメニューから、リファクタリング->名前変更

ファイル名を application.yml へ変えてok


## H2DBを起動する

1. H2DBの設定を定義する

application.ymlに以下を記述

```
spring:
  application:
    name: sample-app
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:sampledb
    # username:
    # password:
```

- name ... 自プロジェクト名に変える
- username, password ... 今回はテスト利用するだけなのでDBユーザー・パスワードは不要

2. Tableスキーマや初期データをsqlファイルで定義する

src/main/resouce を右クリックし、新規->ファイルを選択して、schema.sql ファイルをつくる

以下のcreate文を記述する

```
CREATE TABLE animal (
	id	VARCHAR2(128)	DEFAULT NULL,
	name	VARCHAR2(128)	DEFAULT NULL,
	CONSTRAINT PERSON_PK PRIMARY KEY (id)
);
```

src/main/resouce を右クリックし、新規->ファイルを選択して、data.sql ファイルをつくる

以下のinsert文を記述する

```
INSERT INTO PERSON (ID, NAME) VALUES ('1', 'Dog');
INSERT INTO PERSON (ID, NAME) VALUES ('2', 'Cat');
```

3. SpringBootを起動してDB接続を試す

プロジェクトを右クリックしてメニューを開いて「実行」->「Spring Boot アプリケーション」を選択する

以下にアクセスするとH2DB管理コンソールが起動しているはず

http://localhost:8080/h2-console/login.jsp

application.yml へ記述した内容にそって項目を入力して、Connect

ANIMAL テーブルが作られていることが確認できる

フォームへ SELECT * FROM ANIMAL 入力してRun

data.sqlでINSERTされたデータが取得できる

## データを取り扱うObjectクラスの作成

1. src/main/java/com/sample/animal-db-project を右クリックし、新規->クラスを選択して、Animal.java ファイルをつくる

以下のclass定義を記述する

```
package com.sample.animal-db-project;

import java.util.Optional;

public class Animal {
  // Objectで扱うテーブル上のデータをメンバ変数として宣言する
  //  インスタンス化した時にデータが初期化され変更されない為にfinal宣言とする
  final private String id;
  final private String name;

  // メンバ変数を初期化するコンストラクタを定義する
  public Animal(String id, String name) {
    // 引数を自オブジェクトのメンバ変数へ代入している
    this.id = id;
    this.name = name;
  }

  // メンバ変数を参照するgetterメソッドを定義する
  // Animalはimmutableなオブジェクトなのでsetterは定義しない
  // 今回のDBはNULL値を許容しているので戻り値に Optional を使う
  public Optional<String> getId() {
    return Optional.ofNullable(this.id);
  }
  public Optional<String> getName() {
    return Optional.ofNullable(this.name);
  }


  // 扱い易くする為にBuilderクラスも定義する
  
  // Builderオブジェクトを取得するメソッド
  public static AnimalBuilder builder() {
    return new AnimalBuilder();
  }

  // AnimalBuilderクラス
  public static class AnimalBuilder {

    // Builderはmutableオブジェクトにするためfinal無し
    private String id;
    private String name;
    // 値がセットされたか判断する為のbooleanも
    private boolean id$set;
    private boolean name$set;

    // 値をセットできるようにメソッドを定義する
    //  Builder利用する際にメソッドチェーンが使えるようBuilder自身を戻り値とする
    public Animal.AnimalBuilder id(String id) {
      this.id = id;
      this.id$set = true;
      return this;
    }
    public Animal.AnimalBuilder name(String name) {
      this.name = name;
      this.name$set = true;
      return this;
    }

    // Animalオブジェクト生成メソッド
    public Animal build() {
      // 初期値を定義する今回はnull
      Animal defaultValue = new Animal(null, null);
      return build(defaultValue);
    }
    public Animal build(Animal defaultValue) {
      // 値がセットされてたらその値を使う
      // セットされてなかったらデフォルト値
      return new Animal(
        id$set ? this.id : defaultValue.getId().orElse(null),
        name$set ? this.name : defaultValue.getName().orElse(null)
      );
    }
  }
}
```

2. Animalクラスのテストを定義する

src/main/java/com/sample/animal-db-project を右クリックし、新規->クラスを選択して、AnimalTest.java ファイルをつくる

以下のclass定義を記述する

```
package com.sample.animal-db-project;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import static org.junit.jupiter.api.Assertions.assertEquals;

@SpringBootTest
public class AnimalTest {

  // メンバ変数idが仕様どおり動くか
  @Test
  void idTest() {
    // 初期値がnullであるか
    assertEquals(Animal.builder().build().getId().orElse(null),null);
    // nullをセットできるか
    assertEquals(Animal.builder().id(null).build().getId().orElse(null),null);
    // 値をセットできるか
    assertEquals(Animal.builder().id("123").build().getId().orElse(null),"123");
  }

  // メンバ変数nameが仕様どおり動くか
  @Test
  void nameTest() {
    // 初期値がnullであるか
    assertEquals(Animal.builder().build().getName().orElse(null),null);
    // nullをセットできるか
    assertEquals(Animal.builder().name(null).build().getName().orElse(null),null);
    // 値をセットできるか
    assertEquals(Animal.builder().name("Bard").build().getName().orElse(null),"Bard");
  }

}
```

src/main/java/com/sample/animal-db-project/AnimalTest.java を右クリックし実行->Javaアプリケーションを選択

テストが実行されて結果が表示される

問題無ければグリーンチェックマークが表示される


## データにアクセスするRepositoryクラスの作成

DBとの通信処理が他の層へ影響しないようにインターフェースとしてのクラスを定義する

1. AnimalRepositoryインターフェースを定義する

src/main/java/com/sample/animal-db-project を右クリックし、新規->インターフェースを選択して、AnimalRepository.java ファイルをつくる

以下のinterface定義を記述する

```
package com.sample.animal-db-project;

import java.util.List;

public interface AnimalRepository {
  int insert(Animal animal);
  List<Animal> select(String name);
}
```

2. AnimalRepositoryImplクラスを定義する

src/main/java/com/sample/animal-db-project を右クリックし、新規->クラスを選択して、AnimalRepositoryImpl.java ファイルをつくる

以下の実現クラス定義を記述する

```
package com.sample.animal-db-project;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.namedparam.SqlParameterSource;
import org.springframework.stereotype.Repository;

import java.sql.Timestamp;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

public clss AnimalRepositoryImpl implements AnimalRepository {

  // DBアクセサ今回はNamedParameterJdbcTemplateを使う
  // メンバ変数はfinalに 
  private final NamedParameterJdbcTemplate jdbcTemplate;

  // DIでメンバ変数に値をセットするコンストラクタを定義
  @Autowired
  public AnimalRepositoryImpl(NamedParameterJdbcTemplate jdbcTemplate) {
    this.jdbcTemplate = jdbcTemplate;
  }

  @Override
  public int insert(Animal animal) {
    if (animal == null) {
      return 0;
    }
    if (!animal.getId.isPresent()) {
      return 0;
    }
    String sql = "INSERT INTO /* SAMPLE_ANIMAL_0001 */" +
            "            animal(" +
            "            id" +
            "            ,name)" +
            " VLUES (" +
            "            :id" +
            "            ,:name" +
            ");";

    SqlParameterSource parameters = new MapSqlParameterSource("id", animal.getId().orElse(null))
            .addValue("name", animal.getName().orElse(null));

    return jdbcTemplate.update(sql, parameters);
  }

  @Override
  public List<Animal> select(String name) {
    Stirng sql = "SELECT     /* SMAPLE_ANIMAL_0002 */" +
            "          id" +
            "         ,name" +
            " FROM     animal" +
            " WHERE ";
    if (name == null) {
      sql = sql + " name IS NULL ";
    } else {
      sql = sql + " name=:name ";
    }
    sql = sql + ";";

    SqlParameterSource parameters = new MapSqlParameterSource("name", name");

    List<Map<String, Object>> result = jdbcTemplate.queryForList(sql, parameters);
    List<Animal> animalList = new ArrayList<Animal>();
    for(Map<String, Object> data : result) {
      Animal animal = Animal.builder()
              .id((String)data.get("id"))
              .name((String)data.get("name"))
              .build();
      animalList.add(animal);
    }

    return animalList;
  }
}
```


3. AnimalRepositoryImplクラスのテストを定義する

src/main/java/com/sample/animal-db-project を右クリックし、新規->クラスを選択して、AnimalRepositoryImplTest.java ファイルをつくる

以下のclass定義を記述する

```
package com.sample.animal-db-project;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;

@SpringBootTest
public class AnimalRepositoryImplTest {

  // テスト実行時にインスタンス化される
  @Autowierd
  private AnimalRepositoryImpl animalRepositoryImpl;

  @Test
  void selectTest() {
    List<Animal> animalList;

    // nullのデータが0件になるか
    animalList = animalRepositoryImpl.select(null);
    assertEquals(animalList.isEmpty(), true);

    // 存在しない名前のデータが0件になるか
    animalList = animalRepositoryImpl.select("Human");
    assertEquals(animalList.size(), 0);
  }

  @Test
  void insertTest() {
    // nullが渡されたら何もせず0を返すか
    int count = 0;
    try {
      count = animalRepositoryImpl.insert(null);
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  @Test
  void insertIdTest() {
    int count = 0;
    Animal animal;
    List<Animal> animalList;

    // テスト値を初期化
    animal = Animal.builder()
            .name("Monkey")
            .build();
    try {
      count = animalRepositoryImpl.insert(animal);
    } catch (Exception e) {
      e.printStackTrace();
    }

    // idが初期値=nullの場合にデータが入らないか
    assertEquals(count, 0);
    animalList = animalRepositoryImpl.select("Monkey");
    assertEquals(animalList.isEmpty(), true);

    // 値がセットできるか
    animal = Animal.builder()
            .id("12")
            .name("Monkey")
            .build();
    try {
      count = animalRepositoryImpl.insert(animal);
    } catch (Exception e) {
      e.printStackTrace();
    }
    assertEquals(count, 1);
    animalList = animalRepositoryImpl.select("Monkey");
    assertEquals(animal.get(0).getId().orElse(null), "12");
  }

  @Test
  void insertNameTest() {
    int count = 0;
    Animal animal;
    List<Animal> animalList;

    // テスト値を初期化
    animal = Animal.builder()
            .id("21")
            .build();
    try {
      count = animalRepositoryImpl.insert(animal);
    } catch (Exception e) {
      e.printStackTrace();
    }

    // nameが初期値=nullの場合にデータが入るか
    assertEquals(count, 1);
    animalList = animalRepositoryImpl.select(null);
    assertEquals(animal.get(0).getId().orElse(null), "21");
    assertEquals(animal.get(0).getName().orElse(null), null);

    // 値がセットできるか
    animal = Animal.builder()
            .id("25")
            .name("Rabbit")
            .build();
    try {
      count = animalRepositoryImpl.insert(animal);
    } catch (Exception e) {
      e.printStackTrace();
    }
    assertEquals(count, 1);
    animalList = animalRepositoryImpl.select("Rabbit");
    assertEquals(animal.get(0).getId().orElse(null), "25");
    assertEquals(animal.get(0).getName().orElse(null), "Rabbit");
  }

}
```

src/main/java/com/sample/animal-db-project/AnimalRepositoryImplTest.java を右クリックし実行->Javaアプリケーションを選択

テストが実行されて結果が表示される

問題無ければグリーンチェックマークが表示される
