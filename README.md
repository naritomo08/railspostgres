## 参考URL
https://qiita.com/na-777/items/fcccb48b4f3b549abe61

## 事前準備

自分の端末の公開鍵をgithubの個人設定に登録してあること。

## 環境構築手順

1.不要なdockerイメージ,ボリューム,コンテナを削除する。

2.本レポジトリをクロンする。

```bash
$ git clone git@github.com:naritomo08/railspostgres.git railspostgres
$ cd railspostgres
$ git config --local user.name "naritomo"
$ git config --local user.email naritomo08@gmail.com
```

2.1 railsアプリの新規作成からする場合

```bash
$ mkdir src
$ cp Gemfile src/
$ cp Gemfile.lock src/
```
手順3以降を実施する。

2.2　railsアプリがすでにある場合

```bash
$ git clone git@github.com:naritomo08/railspostgresapp.git src
$ docker-compose build
$ cd src
$ git config --local user.name "naritomo"
$ git config --local user.email naritomo08@gmail.com
```

手順9に飛んでサービスが立ち上がるか確認する。

3.rails newコマンドをrailpapp上で実行

```
$ docker-compose build
$ docker-compose run --no-deps railpapp rails new . --webpack --force --database=postgresql
```

4.railsのディレクトリができているかチェック

```
$ ls -l
```

5.所有者がrootになっているファイルの所有者を現在のユーザに書き換え
*状況によってはいらない。

```
$ sudo chown -R $USER:$USER .
```

6.rails new で新しいGemfileができたので再ビルド

```
$ docker-compose build
```

7.webpackerのインストール

```
$ docker-compose run railpapp rails webpacker:install
```

8.DBの設定を変更

```
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

9.コンテナ立ち上げ

```
$ docker-compose up
途中で立ち上がらないなどのエラーが出ないこと。
```

10.別タブを開き下記のコマンドを実行してDBを作成

```
$ docker exec -ti railspostgres_railpapp_1 bash
$ rake db:create
```

本手順でアプリ新規作り直しから実施した場合、
すべてのコンソールを立ち上げなおし、
すべてのコンテナを削除後立ち上げなおすこと。

既存のアプリの場合、すべてのコンソールを立ち上げなおし
下記手順を使用し立ち上げなおしを実施すること。

## ログインURL

```
http://localhost:3000
```

## コンテナ起動

```
docker-compose up -d
```

## コンテナ停止

```
docker-compose stop
```

## コンテナ削除

```
docker-compose down
```

## 起動中のコンテナに入る

1. appコンテナ

```bash
$ docker exec -ti railspostgres_railpapp_1 bash
```

2. DBコンテナ

```bash
$ docker exec -ti railspostgres_postgresdb_1 bash
```
## Gemfileを更新した場合

以下のコマンドでコンテナ再ビルド/作り直しを行うこと。

```bash
docker-compose down
docker-compose build
docker-compose up -d
```
