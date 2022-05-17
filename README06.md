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
