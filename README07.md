# 6-9 ログイン処理を実装する

+ `front/src/components/LoginForm.vue`を編集<br>

```vue:LoginForm.vue
<template>
  <div>
    <h2>ログイン</h2>
    <form @submit.prevent="login"> // 編集
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
      <button>ログインする</button>
      <div class="error">{{ error }}</div> // 追加
    </form>
  </div>
</template>

<script>
import axios from "axios"; // 追加
export default {
  data() {
    return {
      email: "",
      password: "",
      error: null, // 追加
    };
  },
  // 追加
  methods: {
    async login() {
      try {
        this.error = null;

        const res = await axios.post("http://localhost:3000/auth/sign_in", {
          email: this.email,
          password: this.password,
        });
        if (!res) {
          throw new Error("メールアドレスかパスワードが違います");
        }

        console.log({ res });

        return res;
      } catch (error) {
        console.log({ error });
        this.error = "メールアドレスがパスワードが違います";
      }
    },
  },
  // ここまで
};
</script>
```

+ api ターミナルに `Unpermitted parameter: :session`と出てる場合<br>

`api/config/initializers/wrap_parameters.rb`を編集<br>

```rb:wrap_parameters.rb
# Be sure to restart your server when you modify this file.

# This file contains settings for ActionController::ParamsWrapper which
# is enabled by default.

# Enable parameter wrapping for JSON. You can disable this by setting :format to an empty array.
ActiveSupport.on_load(:action_controller) do
  wrap_parameters format: [] # 編集
end

# To enable root element in JSON for ActiveRecord objects.
# ActiveSupport.on_load(:active_record) do
#   self.include_root_in_json = true
# end
```

+ `api/app/controllers/auth/registration_controller.rb`を編集<br>

```rb:registration_controller.rb
class Auth::RegistrationsController < DeviseTokenAuth::RegistrationsController

  private

  def sign_up_params
    params.permit(:name, :email, :password, :password_confirmation) # 最初に戻す
  end
end
```

## Front Vue AxiosのbaseURLの設定(production, development)

+ `front mkdir src/api && touch $_/index.js`を実行<br>

+ `fonrt/src/api/index.js`を編集<br>

```js:index.js
import axios from "axios";

export default () => {
  console.log(process.env.VUE_APP_API_ORIGIN)
  const instance1 = axios.create({
    baseURL: `${process.env.VUE_APP_API_ORIGIN}`,
  })
  return instance1;
};
```

+ `front $ touch .env.development`を実行<br>

+ `front/.env.development`を編集<br>

```:.env.development
NODE_ENV='development'
VUE_APP_API_ORIGIN='http://localhost:3000'
```

+ `front $ touch .env.production`を実行<br>

+ `front/.env.production`を編集<br>

```:.env.production
NODE_ENV='production'
VUE_APP_API_ORIGIN='https://rails-vue3cli-api.herokuapp.com'
```

# Front heroku.ymlの作成

+ `front $ touch heroku.yml`を実行<br>

+ `front/heroku.yml`を編集<br>

```yml:heroku.yml
setup:
  config:
    NODE_ENV: production
build:
  docker:
    web: Dockerfile
  config:
    WORKDIR: app
    API_URL: 'https://rails-vue3cli-api.herokuapp.com'
run:
  web: npm run build
```

+ frontディレクトリを git commitしておく<br>

## Vue.js(font)をHerokuにアプリを作成しプロジェクトをPushする (デプロイ編)

+ `front $ heroku create rails-vue3cli-front --maniset`を実行<br>

+ `font $ heroku stack`を実行して container になっているか確認<br>

```:terminal
=== ⬢ rails-vue3cli-front Available Stacks
* container
  heroku-18
  heroku-20
```

+ `front $ heroku config`を実行して環境変数を確認<br>

```:terminal
=== rails-vue3cli-front Config Vars
NODE_ENV: production
```

+ `front $ git push heroku main`を実行<br>

+ `root $ cd api`を実行<br>

+ `api heroku config:set API_DOMAIN=rails-vue3cli-front.herokuapp.com`を実行<br>

+ `api heroku config`を実行して確認<br>

```:terminal
=== rails-vue3cli-api Config Vars
API_DOMAIN:               rails-vue3cli-front.herokuapp.com
DATABASE_URL:             postgres://lkcemzvldthfud:bf294aed21561cfe6041e52a76a33c00f69c7aefee890efdd544e46e121584c9@ec2-54-86-224-85.compute-1.amazonaws.com:5432/daon5gds3o4te4
RACK_ENV:                 production
RAILS_ENV:                production
RAILS_LOG_TO_STDOUT:      enabled
RAILS_MASTER_KEY:         dbd7ff5441b828998b19239769b0f033
RAILS_SERVE_STATIC_FILES: enabled
```

## Herokuへデプロイするための最終設定

+ `root $ docker compose run --rm front npm install express`を実行<br>

+ `front/package.json`を編集<br>

```josn:package.json
{
  "name": "app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint",
    "start": "node server.js" // 追加
  },
  "dependencies": {
    "axios": "^0.27.2",
    "core-js": "^3.6.5",
    "express": "^4.18.1",
    "vue": "^3.0.0",
    "vue-axios": "^3.4.1",
    "vue-router": "^4.0.0-0"
  },
  "devDependencies": {
    "@vue/cli-plugin-babel": "~4.5.13",
    "@vue/cli-plugin-eslint": "~4.5.13",
    "@vue/cli-plugin-router": "~4.5.13",
    "@vue/cli-service": "~4.5.13",
    "@vue/compiler-sfc": "^3.0.0",
    "babel-eslint": "^10.1.0",
    "eslint": "^6.7.2",
    "eslint-plugin-vue": "^7.0.0"
  }
}
```

+ `front/Dockerfile`を修正<br>

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
  API_URL=${VUE_APP_API_ORIGIN}

WORKDIR ${HOME}

# ローカル上のpackageをコンテナにコピーしてインストールする
RUN apk update && npm install -g @vue/cli@4.5.13
COPY package*.json ./
RUN npm install

# コンテナにNuxtプロジェクトをコピー

COPY . ./

# 本番環境にアプリを構築
# RUN npm run build // 削除

# CMD ["-g", "daemon off;"]
```

+ `front/heroku.yml`を修正<br>

```yml:heroku.yml
setup:
  config:
    NODE_ENV: production
build:
  docker:
    web: Dockerfile
  config:
    WORKDIR: app
    API_URL: "https://rails-vue3cli-api.herokuapp.com"
run:
  web: npm run build && npm run start # 編集
```

+ `front $ touch server.js`を実行<br>

+ `front/server.js`を編集<br>

```js:server.js
const express = require('express');
const port = process.env.PORT || 3000;
const app = express();
app.use(express.static(__dirname + "/dist/"));
app.listen(port);
```

+ `front/src/api/index.js`を編集<br>

```js:index.js
import axios from "axios";

export default () => {
  // eslint-disable-next-line no-console
  console.log(process.env.VUE_APP_API_ORIGIN)
  const API_URL = axios.create({
    baseURL: `${process.env.VUE_APP_API_ORIGIN}`,
  })
  return API_URL;
};
```

これでHerokuにデプロイできるはず<br>


## 6-10 ページ遷移の処理を追加する

+ `fornt $ touch src/views/Chatroom.vue`を編集<br>

```vue:Chatroom.vue
<template>
  <div>チャットルームです</div>
</template>

<script>
export default {};
</script>

<style scoped>
</style>
```

+ `front/src/views/index.js`を編集<br>

```js:index.js
import { createRouter, createWebHistory } from 'vue-router'
import WelcomePage from '../views/WelcomePage'
import Chatroom from '../views/Chatroom'

const routes = [
  {
    path: '/',
    name: 'WelcomePage',
    component: WelcomePage
  },
  {
    path: '/chatroom',
    name: 'Chatroom',
    component: Chatroom
  }
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router
```

+ localhost:8080/chatroom にアクセスしてみる<br>

# Welcome.vueにリダイレクト処理を追加する

+ `front/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <div class="container welcome">
    <p>ようこそ!</p>
    <div v-if="shouldShowLoginForm">
      <login-form @redirectToChatRoom="redirectToChatRoom" />  // 編集
      <p class="change-form">
        初めての方は<span @click="shouldShowLoginForm = false">こちら</span
        >をクリック
      </p>
    </div>
    <div v-if="!shouldShowLoginForm">
      <signup-form @rredirectTohatRoom="redirectTohatRoom" /> // 編集
      <p class="change-form">
        アカウントをお持ちの方は<span @click="shouldShowLoginForm = true"
          >こちら</span
        >をクリック
      </p>
    </div>
  </div>
</template>

<script>
import LoginForm from "../components/LoginForm.vue";
import SignupForm from "../components/SignupForm.vue";
export default {
  components: { LoginForm, SignupForm },
  data() {
    return {
      shouldShowLoginForm: true,
    };
  },
  // 追加
  methods: {
    redirectTohatRoom() {
      this.$router.push({ name: "Chatroom" });
    },
  },
  // ここまで
};
</script>

<style>
.welcome {
  text-align: center;
  padding: 20px 0;
}
.welcome form {
  width: 300px;
  margin: 20px auto;
}
.welcome label {
  display: block;
  margin: 20px 0 10px;
}
.welcome input {
  width: 100%;
  padding: 12px 20px;
  margin: 8px auto;
  border-radius: 4px;
  border: 1px solid #eee;
  outline: none;
  box-sizing: border-box;
}
.welcome span {
  font-weight: bold;
  text-decoration: underline;
  cursor: pointer;
}
.welcome button {
  margin: 20px auto;
}
</style>
```

## SignupForm.vueにリダイレクト処理を追加する

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
import axios from "../api/index";
export default {
  emits: ["redirectToChatRoom"], // 親のメソッドを実行するため
  data() {
    return {
      name: "",
      email: "",
      password: "",
      passwordConfirmation: "",
      error: null,
    };
  },
  methods: {
    async signUp() {
      this.error = null;
      try {
        const res = await axios().post("/auth", {
          name: this.name,
          email: this.email,
          password: this.password,
          password_confirmation: this.passwordConfirmation,
        });
        if (!res) {
          throw new Error("アカウントを登録できませんでした");
        }

        if (!this.error) {
          this.$emit("redirectToChatRoom"); // 親のメソッドを実行
        }
        // eslint-disable-next-line no-console
        console.log({ res });
        return res;
      } catch (error) {
        this.error = "アカウントを登録できませんでした";
      }
    },
  },
};
</script>
```

## LoginForm.vue にリダイレクト処理を追加する

+ `front/src/components/LoginForm.vue`を編集<br>

```vue:LoginForm.vue
<template>
  <div>
    <h2>ログイン</h2>
    <form @submit.prevent="login">
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
      <div class="error">{{ error }}</div>
      <button>ログインする</button>
    </form>
  </div>
</template>

<script>
import axios from "../api/index";
export default {
  emits: ["redirectToChatRoom"], // 追加
  data() {
    return {
      email: "",
      password: "",
      error: null,
    };
  },
  methods: {
    async login() {
      try {
        this.error = null;

        const res = await axios().post("/auth/sign_in", {
          email: this.email,
          password: this.password,
        });

        if (!res) {
          throw new Error("メールアドレスかパスワードが違います");
        }

        // 追加
        if (!this.error) {
          this.$emit("redirectToChatRoom");
        }
        // ここまで

        // eslint-disable-next-line no-console
        console.log({ res });

        return res;
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log({ error });
        this.error = "メールアドレスかパスワードが違います";
      }
    },
  },
};
</script>
```

+ テストしてみる<br>
