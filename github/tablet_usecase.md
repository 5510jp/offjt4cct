# Lenovo Tab M10 HD  で快適に使う

## 1. Termuxセットアップして使い方を覚える

1.1 Androidアプリをインストールして起動する

1.2 パッケージ環境を最新化して sshをインストールする
```
apt upgrade && apt update
pkg install openssl
apt install openssl-tool

pkg install vim
pkg install git
```

1.3 使い方を覚える

- 基本的にここによくまとまってます  
https://qiita.com/tasakii/items/e3415780c99aa07a963a

- 日本語入力したいときはツールバーを左へスワイプして入力プロンプトを表示すると自動的にかな入力にきりかわります

- Hacker's Keyboard と組み合わせるとかなり快適になります、特にTabキーで補完が使えるだけでかなり楽になる

## 2. ssh鍵を作って登録

2.1 作る

```
ssh-keygen -t rsa -b 4096

// パスフレーズを確認する
ssh-add id_rsa
```

- id_rsa ... 秘密鍵、しっかり管理して下さい
- id_rsa.pub ... 公開鍵、Githubへ登録します

2.2 登録する

https://github.com/settings/profile  
SSH and GPG keys  
New SSH key  
ここへ公開鍵をコピペして登録する

## 3. gitコマンドを使う

3.1 config最低限しておく

3.2 署名する

3.3 確認する

