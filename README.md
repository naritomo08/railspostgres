## 参考URL
[Windows10のWSL2(Ubuntu)でDockerを使ったRails環境構築](https://qiita.com/na-777/items/fcccb48b4f3b549abe61)

[Elixirをdocker環境で立ち上げてみる。](https://qiita.com/naritomo08/items/fecf4ace7b9ca9078102)

## 事前準備

windows11+wsl2+Ubuntu22+DockerCompose+vscodeでの環境を構築してること。

## 環境構築手順

### 不要なdockerイメージ,ボリューム,コンテナを削除する。

### 本レポジトリをクロンする。

```bash
$ git clone git@github.com:naritomo08/railspostgres.git railspostgres
$ cd railspostgres
```

### railsアプリの新規作成準備

```bash
$ mkdir src
$ cp Gemfile src/
$ cp Gemfile.lock src/
```
### rails newコマンドをrailpapp上で実行

```bash
$ docker-compose build
$ docker-compose run --no-deps railpapp rails new . --webpack --force --database=postgresql
```

### railsのディレクトリができているかチェック

```bash
$ ls -l
```

### 所有者がrootになっているファイルの所有者を現在のユーザに書き換え

```bash
$ sudo chown -R $USER:$USER .
```

### rails new で新しいGemfileができたので再ビルド

```bash
$ docker-compose build
```

### webpackerのインストール

```bash
$ docker-compose run railpapp rails webpacker:install
```

### DBの設定を変更

```bash
$ vi src/config/database.yml

default: &default
  adapter: postgresql
  encoding: unicode
  host: postgresdb #追加
  username: postgres #追加
  password: password #追加
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: myapp_development

test:
  <<: *default
  database: myapp_test
  
production:
  <<: *default
  database: myapp_production
  username: myapp
  password: <%= ENV['MYAPP_DATABASE_PASSWORD'] %>
```

### コンテナ立ち上げ

```bash
$ docker-compose up
途中で立ち上がらないなどのエラーが出ないこと。
```

### 別タブを開き下記のコマンドを実行してDBを作成

```bash
$ docker exec -ti railspostgres_railpapp_1 bash
$ rake db:create
```

本手順でアプリ新規作り直しから実施した場合、
すべてのコンソールを立ち上げなおし、
すべてのコンテナを削除後立ち上げなおすこと。

既存のアプリの場合、すべてのコンソールを立ち上げなおし
下記手順を使用し立ち上げなおしを実施すること。

## ログインURL


### Rubyサイト

http://localhost:3000

### adminer(DB管理ツール)

http://127.0.0.1:8081

* ログイン情報
  - データベース種類: Postgresql
  - サーバ: postgresdb
  - ユーザ名: postgres
  - パスワード:password

mailhog(メールサーバ)

http://127.0.0.1:8025

## コンテナ起動

```bash
docker-compose up -d
```

## コンテナ停止

```bash
docker-compose stop
```

## コンテナ削除

```bash
docker-compose down
```

## 起動中のコンテナに入る

1. appコンテナ

```bash
$ docker-compose exec railpapp bash
```

2. DBコンテナ

```bash
$ docker-compose exec postgresdb bash
```
## Gemfileを更新した場合

以下のコマンドでコンテナ再ビルド/作り直しを行うこと。

```bash
docker-compose down
docker-compose build
docker-compose up -d
```
