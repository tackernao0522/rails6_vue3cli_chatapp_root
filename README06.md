# 6-8 サインアップ処理を実装する

+ `front/src/components/SignupForm.vue`を編集<br>

```vue:SignupForm.vue
<template>
  <div>
    <h2>アカウントを登録</h2>
    // 編集 @submit.prevent="signUp"を追記する
    <form @submit.prevent="signUp">
      <input type="text" required placeholder="名前" v-model="name" />
      <input
        type="email"
        required
        placeholder="メールアドレス"
        v-model="email"
      />
      <input
        type="password"
        required
        placeholder="パスワード"
        v-model="password"
      />
      <input
        type="password"
        required
        placeholder="パスワード(確認用)"
        v-model="passwordConfirmation"
      />
      <button>登録する</button>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: "",
      email: "",
      password: "",
      passwordConfirmation: "",
    };
  },
  // 追加
  methods: {
    async signUp() {
      console.log(
        this.name,
        this.email,
        this.password,
        this.passwordConfirmation
      );
    },
  },
  // ここまで
};
</script>
```

## Axiosをインストールする

+ `root $ docker compose run --rm front npm install axios vue-axios@3.2.4`を実行<br>

## Axiosと使ってサインアップAPIと通信する

+ `front/src/components/SignupForm.vue`を編集<br>

```vue:SignupForm.vue
<template>
  <div>
    <h2>アカウントを登録</h2>
    <form @submit.prevent="signUp">
      <input type="text" required placeholder="名前" v-model="name" />
      <input
        type="email"
        required
        placeholder="メールアドレス"
        v-model="email"
      />
      <input
        type="password"
        required
        placeholder="パスワード"
        v-model="password"
      />
      <input
        type="password"
        required
        placeholder="パスワード(確認用)"
        v-model="passwordConfirmation"
      />
      <button>登録する</button>
    </form>
  </div>
</template>

<script>
// 追加
import axios from "axios";
export default {
  data() {
    return {
      name: "",
      email: "",
      password: "",
      passwordConfirmation: "",
    };
  },
  // 編集
  methods: {
    async signUp() {
      try {
        const res = await axios.post("http://localhost:3000/auth", {
          name: this.name,
          email: this.email,
          password: this.password,
          password_confirmation: this.passwordConfirmation,
        });
        console.log({ res });
        return res;
      } catch (error) {
        console.log({ error });
      }
    },
  },
  // ここまで
};
</script>
```

### terminalに下記のようなエラーが出る場合

```:terminal
Unpermitted parameter: :registration
```

+ `api/app/controller/auth/registrations_controller.rb`を編集<br>

```rb:registrations_controller.rb
class Auth::RegistrationsController < DeviseTokenAuth::RegistrationsController

  private

  def sign_up_params
    params.require(:registration).permit(:name, :email, :password, :password_confirmation) # 修正
  end
end
```

## エラー処理を実装する

+ `front/src/components/SignupForm.vue`を編集<br>

```vue:SignupForm.vue
<template>
  <div>
    <h2>アカウントを登録</h2>
    <form @submit.prevent="signUp">
      <input type="text" required placeholder="名前" v-model="name" />
      <input
        type="email"
        required
        placeholder="メールアドレス"
        v-model="email"
      />
      <input
        type="password"
        required
        placeholder="パスワード"
        v-model="password"
      />
      <input
        type="password"
        required
        placeholder="パスワード(確認用)"
        v-model="passwordConfirmation"
      />
      <div class="error">{{ error }}</div>
      <button>登録する</button>
    </form>
  </div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      name: "",
      email: "",
      password: "",
      passwordConfirmation: "",
      // 追加
      error: null,
    };
  },
  methods: {
    async signUp() {
      // 追加
      this.error = null;
      try {
        const res = await axios.post("http://localhost:3000/auth", {
          name: this.name,
          email: this.email,
          password: this.password,
          password_confirmation: this.passwordConfirmation,
        });
        // 追加
        if (!res) {
          throw new Error("アカウントを登録できませんでした");
        }
        console.log({ res });
        return res;
      } catch (error) {
        this.error = "アカウントを登録できませんでした"; // 編集
      }
    },
  },
};
</script>
```

## Rails Puma の設定と heorku.ymlの作成 〜デプロイ準備編〜

+ `api/config/puma.rb`を編集<br>

```rb:puma.rb
# Doc: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#config

# Specifies the number of `workers` to boot in clustered mode.
# Workers are forked web server processes. If using threads and workers together
# the concurrency of the application would be max `threads` * `workers`.
# Workers do not work on JRuby or Windows (both of which do not support
# processes).
#
workers Integer(ENV.fetch("WEB_CONCURRENCY") { 2 })

# Puma can serve each request in a thread from an internal thread pool.
# The `threads` method setting takes two numbers: a minimum and maximum.
# Any libraries that use thread pools should be configured to match
# the maximum value specified for Puma. Default is set to 5 threads for minimum
# and maximum; this matches the default thread size of Active Record.
#
max_threads_count = Integer(ENV.fetch("RAILS_MAX_THREADS") { 5 })
min_threads_count = Integer(ENV.fetch("RAILS_MIN_THREADS") { max_threads_count })
threads min_threads_count, max_threads_count

# Use the `preload_app!` method when specifying a `workers` number.
# This directive tells Puma to first boot the application and load code
# before forking the application. This takes advantage of Copy On Write
# process behavior so workers use less memory.
#
preload_app!

rackup      DefaultRackup

# Specifies the `port` that Puma will listen on to receive requests; default is 3000.
#
port        ENV.fetch("PORT") { 3000 }

# Specifies the `environment` that Puma will run in.
#
environment ENV.fetch("RACK_ENV") { "development" }

on_worker_boot do
  # Worker specific setup for Rails 4.1+
  # See: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#on-worker-boot
  ActiveRecord::Base.establish_connection
end
```

+ `root $ touch api/heroku.yml`を実行<br>

+ `api/heroku.yml`を編集<br>

```yml:heroku.yml
# アプリ環境を定義する場所
setup:
  # アプリ作成時にアドオンを自動で追加する
  addons:
    - plan: heroku-postgresql
  # 環境変数を指定する
  config:
    # Rackへ現在の環境を示す
    RACK_ENV: production
    # Railsへ現在の環境を示す
    RAILS_ENV: production
    # log出力のフラグ(enabled => 出力する)
    RAILS_LOG_TO_STDOUT: enabled
    # publicディレクトリからの静的ファイルを提供してもらうかのフラグ(enabled => 提供してもらう)
    RAILS_SERVE_STATIC_FILES: enabled
# ビルドを定義する場所
build:
  # 参照するDockerfileの場所を定義(相対パス)
  docker:
    web: Dockerfile
  # Dockerfileに渡す環境変数を指定
  config:
    WORKDIR: app
# プロセスを定義
run:
  # Bundlerでインストールされたgemを使用してコマンドを実行
  web: bundle exec puma -C config/puma.rb
```

+ `root/docker-compose.yml`を編集<br>

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
      - "$FRONT_PORT:3000"
    depends_on:
      - api
```

+ `root $ docker images`を実行<br>

```:terminal
REPOSITORY                   TAG           IMAGE ID       CREATED              SIZE
rails-vuecli-chatapp_front   latest        29bc8e52cf0d   About a minute ago   665MB
rails-vuecli-chatapp_api     latest        6f88b165165b   2 minutes ago        301MB
postgres                     13.1-alpine   8c6053d81a45   15 months ago        159MB
```

+ `root $ docker inspect -f="{{ .Config.Cmd }}" 6f88b165165b`を実行<br>

```:terminal
[irb]
```

+ `root $ docker compose run --rm api sh`を実行(api コンテナに入ることができる)<br>

+ `/app # /bin/sh -c "echo TEST"`を実行<br>

```:terminal
TEST
```

+ `/app # /bin/sh -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"`を実行<br>

```terminal
=> Booting Puma
=> Rails 6.0.5 application starting in development
=> Run `rails server --help` for more startup options
[10] Puma starting in cluster mode...
[10] * Version 4.3.12 (ruby 2.6.4-p104), codename: Mysterious Traveller
[10] * Min threads: 5, max threads: 5
[10] * Environment: development
[10] * Process workers: 2
[10] * Preloading application
[10] * Listening on tcp://0.0.0.0:3000
[10] Use Ctrl-C to stop
[10] - Worker 0 (pid: 17) booted, phase: 0
[10] - Worker 1 (pid: 18) booted, phase: 0
```

+ `/app # exit`でコンテナから抜ける<br>


