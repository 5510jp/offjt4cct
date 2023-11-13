# SpringBoot HelloWorld


## 概要

セットアップして環境が動くかどうか確かめる

簡単なプログラムを作成して動作確認を行う


## プロジェクト作成

1. 「ファイル」->「新規」->「Springスターター・プロジェクト」を選択

2. タイプを「Maven」、Javaバージョンを「8」にして「次へ」を選択

3. 「Spring Web」を選択して「完了」を選択

4. プロジェクトが自動作成される


## コントローラーを実装

クライアントからのリクエストを受け取ってレスポンスを返す Controller の機能を実装する

プロジェクトから以下のファイルを編集する

`src/main/java/com/example/demo/DemoApplication.java`


```Java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@SpringBootApplication
@Controller
public class DemoApplication {

    @RequestMapping("/")
    @ResponseBody
    String helloWorld(){
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```

#### @Controller

クラスに対して@Controllerあるいは@RestControllerというアノテーションを記述することで、Spring Boot はそのクラスを Controller として認識します

@RestController では View に遷移せず、メソッドの戻り値がそのままレスポンスコンテンツになります

@Controller では後述する @ResponseBody というアノテーションを使用することで同様の処理ができますが、 @RestController は @ResponseBody を記述する必要はありません

今回は @Controller を記述します

#### @RequestMapping

@RequestMapping というアノテーションを記述することでクライアントからのリクエストに対してメソッドやハンドラをマッピングします

一般的には特定のURLリクエストに対してマッピングを行いますが後述する属性を使えばGETやPOSTといったメソッドなどで条件付けすることも可能です

@RequestMapping はクラスとメソッドどちらにも使用できます

クラスに対して使用した場合は設定された値は親パスとして機能しこの場合メソッドはデフォルトで親パスへのリクエストにマッピングされます

メソッドに使用した場合は親パスぱらの相対パスと絶対パスの両方が設定できます

#### @ResponseBody

@ResponseBody というアノテーションをメソッドに記述すると戻り値がそのままレスポンスボディの内容になります


## 実行

1. プロジェクトを右クリックしてメニューを開いて「実行」->「Spring Boot アプリケーション」を選択する

2. ブラウザでいかにアクセスするとHello Worldと表示される

http://localhost:8080/
