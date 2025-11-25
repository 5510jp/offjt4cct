# 【究極のクリーン環境】JavaもGradleもPCに入れない！VS Code「Dev Containers」実践ガイド

皆さんは、新しいプロジェクトに参加したり、PCを新調したりした際に、環境構築でこんな悩みを抱えたことはありませんか？

😵「プロジェクトごとにJavaのバージョンが違って、切り替えが面倒…」

🥴「AさんのPCでは動くけど、BさんのPCではビルドエラーになる…」

😵‍💫「いろいろインストールしすぎて、いつの間にかPCの中がぐちゃぐちゃ…」

今回は、そんな悩みを一発で解決する 「究極の環境構築」 をご紹介します。

使うのは VS Code と コンテナ技術（Rancher Desktop） だけ。 なんと、ホストPC（手元のパソコン）にはJavaもGradleもDBもインストールしません。 すべてをコンテナの中に閉じ込めることで、いつでも捨てられる「汚れない開発環境」を作る方法を解説します。

**intellij** でも可能です（有料版）

## 1. 「Dev Containers」とは？ (WHY)
今回導入するのは、VS Codeの機能である 「Dev Containers (開発コンテナ)」 です。

通常はPCに直接インストールする開発ツール（Java, Gradleなど）を、すべてDockerコンテナの中に閉じ込め、VS Codeを使ってコンテナの中に入り込んで開発を行う仕組みです。

* PCが汚れない:  
  必要なツールはコンテナ内にあるため、PC本体はクリーンなままです。

* 全員同じ環境:  
 設定ファイル（コード）を共有するだけで、チーム全員が100%同じ環境で開発できます。

* セットアップが爆速:  
 「コマンド一発」で環境構築が完了します。

## 2. 事前準備 (TOOLS)
まずは土台となるツールを2つだけインストールしてください。

Rancher Desktop: (Docker Desktopの代替として利用します)

重要: インストール後の設定で、Container Engineは必ず dockerd (moby) を選択してください。

PATH設定は「Automatic」にします。

Visual Studio Code: 最新版をインストールしてください。

VS Code拡張機能:

Dev Containers (Microsoft製): これが今回の主役です。必ずインストールしてください。

## 3. 実践：設定ファイルの作成 (HOW)
プロジェクトのルートディレクトリ（空のフォルダでOKです）に .devcontainer というフォルダを作成し、その中に2つのファイルを配置します。

ディレクトリ構成:
```Plaintext
project-root/
├── .devcontainer/
│   ├── devcontainer.json
│   └── Dockerfile
```

### 1. Dockerfile の作成
今回は、最初から Gradle と Java がインストール済みの公式イメージ を使用します。これで環境構築の手間がゼロになります。

📄 .devcontainer/Dockerfile

```Dockerfile
# GradleとJDK17が入った公式イメージを使用
FROM gradle:8.5-jdk17

# 必要に応じてツールを追加（gitやvimなど）
RUN apt-get update && apt-get install -y \
    git \
    vim \
    && rm -rf /var/lib/apt/lists/*

# 作業ディレクトリの設定
WORKDIR /workspace
```

### 2. devcontainer.json の作成
VS Codeの設定ファイルです。

📄 .devcontainer/devcontainer.json

```JSON
{
  "name": "Spring Boot Dev Container",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "vscjava.vscode-java-pack",
        "vmware.vscode-spring-boot",
        "mhutchie.git-graph"
      ]
    }
  },
  // アプリケーションのポート(8080)をホスト側に転送
  "forwardPorts": [8080],
  
  // Gradleコンテナはデフォルトユーザーが 'gradle' なので、権限トラブル回避のため指定
  "remoteUser": "gradle" 
}
```

## 4. コンテナの起動
設定ファイルができたら、魔法の時間です。

VS Codeでプロジェクトを開く:

先ほどのファイルが含まれるフォルダを開きます。

コンテナを起動する:

画面左下にある緑色の >< アイコンをクリックします（または F1 キー）。

メニューから 「Dev Containers: Reopen in Container」 を選択します。

初回はDockerイメージのダウンロードとビルドが走ります。しばらく待って、左下の緑色エリアに 「Dev Container: ...」 と表示されれば、あなたはもうコンテナの中にいます！

## 5. サンプルアプリで動作確認 (Hello World)
環境が整ったので、実際にSpring Bootのコードを書いて動かしてみましょう。 ここからの作業はすべてVS Code上（＝コンテナ内）で行います。

### Step 1: ビルド定義の作成
プロジェクト直下に build.gradle を作成します。

📄 build.gradle

```Groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

### Step 2: Javaソースコードの作成
以下のフォルダ構成になるようにディレクトリを作成し、Javaファイルを作成してください。 src/main/java/com/example/demo/DemoApplication.java

📄 src/main/java/com/example/demo/DemoApplication.java

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @GetMapping("/")
    public String hello() {
        return "Hello World! This works inside Docker!";
    }
}
```

### Step 3: アプリケーションの実行
VS Codeのメニュー「Terminal > New Terminal」を開き、以下のコマンドを実行します。 （コンテナ内にはGradleがインストール済みなので、すぐ実行できます）

```bash
gradle bootRun
```

しばらくしてログが流れ、以下のような表示が出れば起動成功です。 Tomcat started on port 8080 (http) ...

Step 4: ブラウザからアクセス
お使いのブラウザ（Chromeなど）を開き、以下のアドレスにアクセスしてください。

Running: http://localhost:8080

画面に 「Hello World! This works inside Docker!」 と表示されましたか？ おめでとうございます！ PCにJavaを一切インストールすることなく、Spring Bootアプリの開発・実行環境が整いました。

---

📝 コラム：コンテナを消したらソースコードも消えるの？
「コンテナは使い捨て」と聞くと、「一生懸命書いたコードも消えてしまうのでは？」と不安になりますよね。 結論から言うと、ソースコードは消えません！

Dev Containers は、あなたのPCにあるフォルダをコンテナの中に 「マウント（投影）」 して使っています。

PC上のファイル: 実体

コンテナ内のファイル: 鏡に映った像（ただし編集可能）

のようなイメージです。 コンテナ内で「保存」を実行すれば、即座にPC上のファイルも更新されます。万が一、コンテナが壊れて作り直すことになっても、ソースコードはあなたのPC上に安全に残っているので安心してください。

---

## 6. まとめ (NEXT STEP)
この「Dev Containers」方式を採用すると、新人さんが入ってきたときのセットアップ手順は劇的にシンプルになります。

1. VS Code と Rancher Desktop を入れる。

1. リポジトリを git clone する。

1. VS Codeで開いて「Reopen in Container」ボタンを押す。

1. 〜 完了（コーヒーを飲んで待つだけ） 〜

「環境構築で1日終わる」なんてことはもうありません。 ぜひ、次回のプロジェクトからこの構成を取り入れて、快適な開発ライフを送ってください！
