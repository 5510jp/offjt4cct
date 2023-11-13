**this document is WorkInProgress**

# SpringBoot run with AWS Lambda

AWS試したいけど費用が気になる

無料でためせる安心そうなRuntimeとしてのLambda

aws-samを使ったCD

Github Actionsを使ってCI 

## 前提

- javaアプリケーションを任意にビルドできる
- javaアプリケーションを任意にパッケージングできる

## ゴール

### ファイル

- SpringBootアプリケーション本体ソースコード
- LambdaエンドポイントProxy Javaソースコード
- Lambdaで実行可能な形式でビルドをする為の設定
- aws samデプロイ仕様の設定
- aws samが実行するaws上のpipeline設定
- GithubActionsの設定

## SpringBootアプリケーション

普通にアプリを用意して下さい、手元に無ければ適当にHelloWorldでももってくる

LambdaをHandllerとしてProxyセットアップする必要がある

詳細な手順は[こちら](https://github.com/awslabs/aws-serverless-java-container/wiki/Quick-start---Spring-Boot2#manual-setup--converting-existing-projects)

1. 必要なパッケージをimportする

pom.xml

```
<dependency>
    <groupId>com.amazonaws.serverless</groupId>
    <artifactId>aws-serverless-java-container-springboot2</artifactId>
    <version>1.9.1</version>
</dependency>
```

2. ハンドラークラスを作る

2.1. Application.class のところをSpringBootアプリケーションのrootクラスに書き換える

2.2. StreamLambdaHandler.java ファイルとしてSpringBootアプリケーションパッケージrootの一つ上に配置しておく

package 宣言はSpringBootアプリケーションよりひとつ上の階層になり  
Applicationをimportする


```java
// こんな感じのイメージ
package com.Demo;
import com.Demo.SpringBootApplication.Application;

public class StreamLambdaHandler implements RequestStreamHandler {
    private static SpringBootLambdaContainerHandler<AwsProxyRequest, AwsProxyResponse> handler;
    static {
        try {
            handler = SpringBootLambdaContainerHandler.getAwsProxyHandler(Application.class);
            // If you are using HTTP APIs with the version 2.0 of the proxy model, use the getHttpApiV2ProxyHandler
            // method: handler = SpringBootLambdaContainerHandler.getHttpApiV2ProxyHandler(Application.class);
        } catch (ContainerInitializationException e) {
            // if we fail here. We re-throw the exception to force another cold start
            e.printStackTrace();
            throw new RuntimeException("Could not initialize Spring Boot application", e);
        }
    }

    @Override
    public void handleRequest(InputStream inputStream, OutputStream outputStream, Context context)
            throws IOException {
        handler.proxyStream(inputStream, outputStream, context);
    }
}
```

詳細な手順はこれ以外にも色々書かれてるが最低限これだけあれば動かせる


## Lambda

1. Lambdaはパッケージのフォーマットをzipかimageかにしないといけない

SpringBootではzipが簡単そうなので以下のファイルを用意する

src/assembly/bin.xml

```
<assembly xmlns="http://maven.apache.org/ASSEMBLY/2.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/ASSEMBLY/2.0.0 http://maven.apache.org/xsd/assembly-2.0.0.xsd">
    <id>lambda-package</id>
    <formats>
        <format>zip</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <fileSets>
        <!-- copy runtime dependencies with some exclusions -->
        <fileSet>
            <directory>${project.build.directory}${file.separator}lib</directory>
            <outputDirectory>lib</outputDirectory>
            <excludes>
                <exclude>tomcat-embed*</exclude>
            </excludes>
        </fileSet>
        <!-- copy all classes -->
        <fileSet>
            <directory>${project.build.directory}${file.separator}classes</directory>
            <includes>
                <include>**</include>
            </includes>
            <outputDirectory>${file.separator}</outputDirectory>
        </fileSet>
    </fileSets>
</assembly>
```

## aws-sam

samではここまで用意したSpringBoot Javaをzipパッケージしたものを用いたLambdaProxyでの実行をセットアップする


## Github Actions
