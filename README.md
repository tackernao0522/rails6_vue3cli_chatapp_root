## Railsを動かすDockerfileを作成

+ `$ mkdir {api,front}`を実行<br>

+ `cd api`を実行<br>

+ `api $ touch {Dockerfile,Gemfile,Gemfile.lock}`を実行<br>

+ `api/Gemfile`を編集<br>

```:Gemfile
source 'https://rubygems.org'
gem 'rails', '~> 6.0.5'
```

+ `api/Dockerfile`を編集<br>

```:Dockerfile
# ベースイメージを指定する
# FROM ベースイメージ:タグ(タグはなくてもよいが最新のものが指定されることになる)
FROM ruby:2.6.4-alpine

# Dockerfile内で使用する変数を定義
# appという値が入る
ARG WORKDIR

ARG RUNTIME_PACKAGES="tzdata postgresql-dev postgresql git"

ARG DEV_PACKAGES="build-base curl-dev"

# 環境変数を定義(Dockerfile, コンテナ参照可能)
# Rails ENV["TZ"] => Asia/Tokyoが出力される
ENV HOME=/${WORKDIR} \
    LANG=C.UTF-8 \
    TZ=Asia/Tokyo

# Dockerfime内で指定した命令を実行する・・・RUN, COPY, ADD, ENTORYPOINT, CMD
# 作業ディレクトリを定義
# コンテナ/app/Railsアプリ
WORKDIR ${HOME}

# ホスト側の(PC)のファイルをコンテナにコピー
# COPY コピー元(ホスト) コピー先(コンテナ)
# Gemfile* ... Gemfileから始まるファイルを全指定(Gemfile, Gemfile, Gemfile.lock)
# コピー元(ホスト) ... Dockerfileがあるディレクトリ以下を指定(api) ../ NG
# コピー先(コンテナ) ... 絶対パス or 相対パス(./ ... 今いる(カレント)ディレクトリ)
COPY Gemfile* ./

# apk ... Alpine Linuxのコマンド
# apk update = パッケージの最新リストを取得
RUN apk update && \
  # apk upgrade = インストールパッケージを最新のものに
  apk upgrade && \
  # apk add = パッケージのインストールを実行
  # --no-cache = パッケージをキャッシュしない(Dokcerイメージを軽量化)
  apk add --no-cache ${RUNTIME_PACKAGES} && \
  # --virtual 名前(任意) = 仮想パッケージ
  apk add --virtual build-dependencies --no-cache ${DEV_PACKAGES} && \
  # Gemのインストールコマンド
  # -j4(jobs=4) = Gemインストールの高速化
  bundle install -j4 && \
  # パーケージを削除(Dokcerイメージを軽量化)
  apk del build-dependencies
# . ... Dockerfileがあるディレクトリ全てのファイル(サブディレクトリも含む)
COPY . ./

# ホスト(PC)       | コンテナ
# ブラウザ(外部)    | Rails
```

## Vue.jsを動かすDockerfileを作成

+ `$ cd front`を実行<br>

+ `front $ touch Dockerfile`を実行<br>

+ `front/Dockerfile`を編集<br>

```:Dockerfile
# FROM node:14.4.0-alpine 使用不可
FROM node:16.13.1-alpine

ARG WORKDIR
ARG API_URL

ENV HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo \
  # これを指定しないとブラウザからhttp://localhost へアクセスすることができない。
  # コンテナのNuxt.jsをブラウザから参照するためにip:0.0.0.0に紐付ける
  # https://ja.nuxtjs.org/faq/host-port/
  HOST=0.0.0.0 \
  API_URL=${API_URL}

WORKDIR ${HOME}

# ローカル上のpackageをコンテナにコピーしてインストールする
RUN apk update && npm install -g @vue/cli@4.5.13
# COPY package*.json ./
# RUN npm install

# コンテナにNuxtプロジェクトをコピー
COPY . ./
```

## .env, .gitignore, docker-compose.ymlを作成

+ `root touch {.gitignore,.env,docker-compose.yml}`を実行<br>

+ `root/.gitignore`ファイルを編集<br>

```:.gitignore
/.DS_Store
```

+ `root/.env`ファイルを編集<br>

```:.env
# commons
WORKDIR=app
API_PORT=3000
FRONT_PORT=8080

# db
POSTGRES_PASSWORD=password
```

+ `root $ docker-compose.yml`を編集<br>

```yml:docker-compose.yml
# composeファイルのバージョン指定
# Doc: https://docs.docker.com/compose/compose-file/compose-versioning/
version: "3.8"

services:
# サービス(= コンテナ)
  db:
    # ベースイメージを定義
    image: postgres:13.1-alpine
    # 環境変数を定義
    environment:
      # OSのタイムゾーン
      TZ: UTC
      # postgresのタイムゾーン
      PGTZ: UTC
      # データベースのパスワード
      POSTGRES_PASSWORD: $POSTGRES_PASSWORD
    # ホスト側のディレクトリをコンテナで使用する
    # volumes: ホストパス(絶対 or 相対) : コンテナパス(絶対)
    volumes:
      - "./api/tmp/db:/var/lib/postgresql/data"

  api:
    # ベースイメージとなるDockerfileを指定
    build:
      context: ./api
      # Dockerfileに変数を渡す
      args:
        WORKDIR: $WORKDIR
    environment:
      POSTGRES_PASSWORD: $POSTGRES_PASSWORD
      API_DOMAIN: "localhost:$FRONT_PORT"       # 追加
      BASE_URL: "http://localhost:$API_PORT"
    command: /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
    volumes:
      - "./api:/$WORKDIR"
    # サービスの依存関係を定義(起動の順番)
    # 公開したいポート番号:コンテナポート
    depends_on:
      - db
    # 公開用ポートを指定
    ports:
      - "$API_PORT:3000"

  front:
    build:
      context: ./front
      args:
        WORKDIR: $WORKDIR
        API_URL: "http://localhost:$API_PORT" # 追加
    # コンテナで実行したいコマンド(CMD)
    command: npm run serve
    volumes:
      - "./front:/$WORKDIR"
    ports:
      - "$FRONT_PORT:8080"
    depends_on:
      - api
```

## Rails アプリを立ち上げる

+ `root $ docker compose build`を実行<br>

+ `root $ docker images`を実行してイメージが出来ているか確認してみる<br>

+ `root $ docker compose run --rm api rails new . -f -B -d postgresql --skip-bundle --skip-turbolinks --skip-test --api`を実行<br>

+ `root $ docker compose build`を実行<br>

+ `api/config/database.yml`を編集<br>

```yml:database.yml
# PostgreSQL. Versions 9.3 and up are supported.
#
# Install the pg driver:
#   gem install pg
# On macOS with Homebrew:
#   gem install pg -- --with-pg-config=/usr/local/bin/pg_config
# On macOS with MacPorts:
#   gem install pg -- --with-pg-config=/opt/local/lib/postgresql84/bin/pg_config
# On Windows:
#   gem install pg
#       Choose the win32 build.
#       Install PostgreSQL and put its /bin directory on your path.
#
# Configure Using Gemfile
# gem 'pg'
#
default: &default
  adapter: postgresql
  encoding: unicode
  # ここから追記
  host: db
  username: postgres
  password: <%= ENV["POSTGRES_PASSWORD"] %>
  # ここまで
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>

development:
  <<: *default
  database: app_development

  # The specified database role being used to connect to postgres.
  # To create additional roles in postgres see `$ createuser --help`.
  # When left blank, postgres will use the default role. This is
  # the same name as the operating system user that initialized the database.
  #username: app

  # The password associated with the postgres role (username).
  #password:

  # Connect on a TCP socket. Omitted by default since the client uses a
  # domain socket that doesn't need configuration. Windows does not have
  # domain sockets, so uncomment these lines.
  #host: localhost

  # The TCP port the server listens on. Defaults to 5432.
  # If your server runs on a different port number, change accordingly.
  #port: 5432

  # Schema search path. The server defaults to $user,public
  #schema_search_path: myapp,sharedapp,public

  # Minimum log levels, in increasing order:
  #   debug5, debug4, debug3, debug2, debug1,
  #   log, notice, warning, error, fatal, and panic
  # Defaults to warning.
  #min_messages: notice

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: app_test

# As with config/credentials.yml, you never want to store sensitive information,
# like your database password, in your source code. If your source code is
# ever seen by anyone, they now have access to your database.
#
# Instead, provide the password as a unix environment variable when you boot
# the app. Read https://guides.rubyonrails.org/configuring.html#configuring-a-database
# for a full rundown on how to provide these environment variables in a
# production deployment.
#
# On Heroku and other platform providers, you may have a full connection URL
# available as an environment variable. For example:
#
#   DATABASE_URL="postgres://myuser:mypass@localhost/somedatabase"
#
# You can use this database configuration with:
#
#   production:
#     url: <%= ENV['DATABASE_URL'] %>
#
production:
  <<: *default
  database: app_production
  username: app
  password: <%= ENV['APP_DATABASE_PASSWORD'] %>
```

+ `root $ docker compose run --rm api rails db:create`を実行<br>

+ `root $ docker compose up api`を実行<br>

+ localhost:3000にアクセス(Railsの初期画面が表示されればOK)<br>

## Vue.js プロジェクトを立ち上げる

+ `root $ docker compose run --rm front vue create .`を実行<br>

```:terminal
Vue CLI v4.5.13
? Generate project in current directory? (Y/n) Y
```

```:terminal
Vue CLI v4.5.13
? Please pick a preset:
  Default ([Vue 2] babel, eslint)
❯ Default (Vue 3) ([Vue 3] babel, eslint)
  Manually select features
```

```:terminal
? Pick the package manager to use when installing dependencies:
  Use Yarn
❯ Use NPM
```

```:terminal
Vue CLI v4.5.13
✨  Creating project in /Users/Macのユーザー名/Documents/live-chat-vuejs.
⚙️  Installing CLI plugins. This might take a while...
```

+ `front $ touch vue.config.js`を実行<br>

+ `front/vue.config.js`を編集<br>

```js:vue.config.js
module.exports = {
  devServer: {
    port: 8080,
    disableHostCheck: true,
  },
};
```

+ `front/Dockerfile`を編集<br>

```:Dockerfile
# FROM node:14.4.0-alpine 使用不可
FROM node:16.13.1-alpine

ARG WORKDIR
ARG API_URL

ENV HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo \
  # これを指定しないとブラウザからhttp://localhost へアクセスすることができない。
  # コンテナのNuxt.jsをブラウザから参照するためにip:0.0.0.0に紐付ける
  # https://ja.nuxtjs.org/faq/host-port/
  HOST=0.0.0.0 \
  API_URL=${API_URL}

WORKDIR ${HOME}

# ローカル上のpackageをコンテナにコピーしてインストールする
RUN apk update && npm install -g @vue/cli@4.5.13
# 追記
COPY package*.json ./
RUN npm install

# コンテナにNuxtプロジェクトをコピー
COPY . ./
```

+ `root $ docker compose up front`を実行<br>

+ localhost:8080 へアクセスしてvue.jsの初期画面が表示されればOK<br>

+ `root $ docker compose run --rm front vue add router`を実行<br>

````:terminal
? Use history mode for router? (Requires proper server setup for index fallback in production) Yes でEnter
```

+ `roote $ docker compose up front`を実行<br>

+ localhost:8080 へアクセスすると真っ白な画面になる<br>
