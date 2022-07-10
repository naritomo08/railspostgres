## 参考URL
https://qiita.com/na-777/items/fcccb48b4f3b549abe61

## 事前準備

自分の端末の公開鍵をgithubの個人設定に登録してあること。

## 環境構築手順

1.本レポジトリをクロンする。

```bash
$ git clone git@github.com:naritomo08/railspostgres.git railsdocker
$ cd railsdocker
```

2.関連するdockerイメージ,コンテナを削除する。

3.rails newコマンドをweb上で実行
```
$ docker-compose build
$ docker-compose run --no-deps web rails new . --webpack --force --database=postgresql
```

4.railsのディレクトリができているかチェック
```
$ ls -l
```

5.所有者がrootになっているファイルの所有者を現在のユーザに書き換え
```
$ sudo chown -R $USER:$USER .
```

6.rails new で新しいGemfileができたので再ビルド
```
$ docker-compose build
```

7.webpackerのインストール
```
$ docker-compose run web rails webpacker:install
```

8.DBの設定を変更
```
$ vi config/database.yml

default: &default
  adapter: postgresql
  encoding: unicode
  host: db #追加
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
```

10.別タブを開き下記のコマンドを実行してDBを作成
```
$ docker-compose run web rake db:create
```

## ログインURL
```
http://localhost:3000
```

## 以降の起動・停止

```
docker-compose up -d
docker-compose down
```