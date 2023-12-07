# Java

## コンテンツ

### リサーチ手法について

- [調べ方（Java開発版）](./researchAndDevelopment.md)

### 環境について

- [Eclips基本](./setup_eclips.md)
- [IntelliJ基本（Git連携以降はあまり使わない）](https://tech-blog.rakus.co.jp/entry/20210421/intellij)
- [IntelliJショートカット](https://blog.jetbrains.com/ja/2020/03/12/2848/)

### ハンズオン

コードを自身で書いて動かし個人リポジトリへcommitして誰かに見てもらう進めかたを想定

#### はじめの一歩

- [SpringBoot HelloWorld (Eclips)](./springboot_helloworld.md)
- [DB初歩 (Eclips)](./springboot_db_intro.md)
- [DI](https://qiita.com/kazuki43zoo/items/7a0e96573e930ac934ed)
- [基本的なMVC](http://terasolunaorg.github.io/guideline/current/ja/Overview/FirstApplication.html)

  

#### BusineeCardManagement課題

- https://github.com/mnaganu/business-card-management

#### バッチ処理課題

- [Spring boot Javaプログラムとして動かす](https://github.com/mnaganu/data-copy-batch)
- [Spring Batch taskletを利用する様にしたもの](https://github.com/mnaganu/data-copy-batch-tasklet)
- [Spring Batch chunkを利用する様にしたもの](https://github.com/mnaganu/data-copy-batch-chunk)

##### 概説

- [パッケージ構成はDDDに近い方式をとっている](http://terasolunaorg.github.io/guideline/current/ja/Overview/ApplicationLayering.html)
- [アノテーションベースのコンテナー構成でxmlは使わない](https://spring.pleiades.io/spring-framework/reference/core/beans/annotation-config.html)

##### 課題

1. [IntelliJで business-card-manegement を起動する](./bcm_intellij_first-step.md)
2. [CodeGeneratorの使い方](https://github.com/kudkki/offjt4cct/blob/main/java/bcm_codegen-usage.md)
3. [Gradleビルド入門 (簡単な使い方、build.gradleの書き方、依存)](https://pleiades.io/help/idea/getting-started-with-gradle.html)
4. [Gradleビルド応用（publish）](https://pleiades.io/help/idea/add-a-gradle-library-to-the-maven-repository.html)
5. business-card-managementへ「入力値の検証＝バリデーション」機能を追加する（デバッガ、JUnitなどの使い方）
6. business-card-managementで接続DBを複数化する（分岐をifで処理しないDIを体感し理解する）
7. business-card-manegementの起動がエラーするアノテーション付け忘れなど（難解エラーメッセージを調査する）

### Java資格

- [Oracle Certified Java Programmer, Silver SE 11 対策](./ocjp_silver.md)
