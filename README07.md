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
