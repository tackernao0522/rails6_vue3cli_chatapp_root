# 6-5 ログイン画面を作成する

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <div class="container welcome">
    <p>ようこそ！</p>
  </div>
</template>

<script>
export default {};
</script>

<style scoped>
.welcome {
  text-align: center;
  padding: 20px 0;
}
</style>
```

## グローバルCSSを適用する

+ `front $ mkdir src/assets && touch $_/main.css`を実行<br>

+ `front/src/assets/main.css`を編集<br>

```css:main.css
body {
  background: #51b392;
  color: #444;
}
.container {
  width: 90%;
  max-width: 960px;
  margin: 80px auto;
  border-radius: 3px;
  box-shadow: 2px 4px 6px rgba(28, 6, 49, 0.1);
  background: white;
}
.error {
  color: #ff3f80;
  font-size: 14px;
}
button {
  text-decoration: none;
  background: #51b392;
  color: white;
  font-weight: bold;
  border: 0;
  border-radius: 3px;
  padding: 10px 20px;
  cursor: pointer;
}
```

+ `front/src/main.js`を編集<br>

```js:main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
// 追加
import './assets/main.css'

createApp(App).use(router).mount('#app')
```

## ログインコンポーネントの作成

+ `front touch src/components/LoginForm.vue`を実行<br>

+ `src/components/LoginForm.vue`を編集<br>

```vue:LoginForm.vue
<template>
  <div>
    <h2>ログイン</h2>
    <form>
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
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      email: "",
      password: "",
    };
  },
};
</script>
```

## ログイン画面を読み込む

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <div class="container welcome">
    <p>ようこそ！</p>
    // 追加
    <login-form />
  </div>
</template>

<script>
// ここから追加
import LoginForm from "../components/LoginForm.vue";
export default {
  components: { LoginForm },
};
// ここまで
</script>

<style scoped>
.welcome {
  text-align: center;
  padding: 20px 0;
}
</style>
```

## WelcomePage.vueにCSSを適用する

+ `font/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <div class="container welcome">
    <p>ようこそ！</p>
    <login-form />
  </div>
</template>

<script>
import LoginForm from "../components/LoginForm.vue";
export default {
  components: { LoginForm },
};
</script>

<style> // scopedを削除
.welcome {
  text-align: center;
  padding: 20px 0;
}
// ここから追加
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
// ここまで
</style>
```

## LoginForm.vueを試してみる

+ `front/src/components/LoginForm.vue`を編集<br>

```vue:LoginForm.vue
<template>
  <div>
    <h2>ログイン</h2>
    <form>
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
    </form>
    Eメール: {{ email }} # 確認したら削除
  </div>
</template>

<script>
export default {
  data() {
    return {
      email: "",
      password: "",
    };
  },
};
</script>
```

# 6-6 サインアップ画面の作成

+ `front touch src/components/SignupForm.vue`を実行<br>

```vue:SignupForm.vue
<template>
  <div>
    <h2>アカウントを登録</h2>
    <form>
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
        placeholder="パスワード(確認用"
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
};
</script>
```

## サインアウト画面を読み込む

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <div>
    <h2>アカウントを登録</h2>
    <form>
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
};
</script>
```

# 6-7 画面の切り替えをする

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <div class="container welcome">
    <p>ようこそ！</p>
    // ここから編集
    <div v-if="shouldShowLoginForm">
      <login-form />
    </div>
    <div v-if="!shouldShowLoginForm">
      <signup-form />
    </div>
    // ここまで
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
// 追加
.change-form {
  font-size: 14px;
  margin: 10px;
}
// ここまで
</style>
```

## `shouldShowLoginForm`を切り返すボタンを用意する

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <div class="container welcome">
    <p>ようこそ！</p>
    <div v-if="shouldShowLoginForm">
      <login-form />
      // 追加
      <p class="change-form">
        初めての方は<span @click="shouldShowLoginForm = false">こちら</span
        >をクリック
      </p>
      // ここまで
    </div>
    <div v-if="!shouldShowLoginForm">
      <signup-form />
      // 追加
      <p class="change-form">
        アカウントをお持ちの方は<span @click="shouldShowLoginForm = true"
          >こちら</span
        >をクリック
      </p>
      // ここまで
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
.change-form {
  font-size: 14px;
  margin: 10px;
}
</style>
```