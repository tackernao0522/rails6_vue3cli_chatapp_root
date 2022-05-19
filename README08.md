# 7-1 ナビバーを作成する

+ `front $ touch src/components/Navbar.vue`を実行<br>

+ `front/src/components/Navbar.vue`を編集<br>

```vue:Navbar.vue
<template>
  <nav>
    <div>
      <p>こんにちは、XXさん</p>
      <p class="email">現在、...@...comでログイン中です</p>
    </div>
    <button>ログアウト</button>
  </nav>
</template>

<script>
export default {};
</script>

<style scoped>
nav {
  padding: 20px;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
nav p {
  margin: 2px auto;
  font-size: 16px;
  color: #444;
}
nav p.email {
  font-size: 14px;
  color: #999;
}
</style>
```

+ `front/src/Chatroom.vue`を編集<br>

```vue:Chatroom.vue
<template>
  <div class="container">
    <navbar />
  </div>
</template>

<script>
import Navbar from "../components/Navbar.vue";
export default {
  components: { Navbar },
};
</script>

<style scoped>
</style>
```

試してみる<br>

# 7-2 認証・ユーザー情報を保存・表示する

+ `front/view/LoginForm.vue`を編集<br>

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
  emits: ["redirectToChatRoom"],
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

        if (!this.error) {
          // eslint-disable-next-line no-console
          console.log({ res }); // 追加 (header情報が取得できることを確認する)
          this.$emit("redirectToChatRoom");
        }

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

```:console
data:
data:
email: "takaki55730317@gmail.com"
id: 1
name: "takaki"
provider: "email"
uid: "takaki55730317@gmail.com"
[[Prototype]]: Object
[[Prototype]]: Object
headers:
access-token: "xrYPzV3mbxa02HVR-9e1ug"
cache-control: "max-age=0, private, must-revalidate"
client: "uOA5HZ0q6ws4nEMVAg3TCQ"
content-type: "application/json; charset=utf-8"
expiry: "1654164270"
token-type: "Bearer"
uid: "takaki55730317@gmail.com"
```

+ `front/src/view/LoginForm.vue`を編集<br>

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
  emits: ["redirectToChatRoom"],
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

        if (!this.error) {
          // 編集
          window.localStorage.setItem(
            "access-token",
            res.headers["access-token"] // access-tokenは文字列にする必要がある
          );
          window.localStorage.setItem("client", res.headers.client);
          window.localStorage.setItem("uid", res.headers.uid);
          window.localStorage.setItem("name", res.data.data.name);
          // ここまで
          this.$emit("redirectToChatRoom");
        }

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

これでログインして`localStorage`に入っているか確認してみる(入っていればOK)<br>

## ナビバーに表示する

+ `front/src/components/Navbar.vue`を編集<br>

```vue:Navbar.vue
<template>
  <nav>
    <div>
      <p>こんにちは、{{ name }}さん</p> // 編集
      <p class="email">現在、{{ email }}でログイン中です</p> // 編集
    </div>
    <button>ログアウト</button>
  </nav>
</template>

<script>
export default {
  // 追加
  data() {
    return {
      name: window.localStorage.getItem("name"),
      email: window.localStorage.getItem("uid"),
    };
  },
  // ここまで
};
</script>

<style scoped>
nav {
  padding: 20px;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
nav p {
  margin: 2px auto;
  font-size: 16px;
  color: #444;
}
nav p.email {
  font-size: 14px;
  color: #999;
}
</style>
```

+ localhost:8080/chatroomにログインしてみる<br>


## サインアップ画面にも同じ処理を追加する

+ `front/src/components/SignupForm.vue`を編集<br>

```
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
  emits: ["redirectToChatRoom"],
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
          // 追加
          window.localStorage.setItem(
            "access-token",
            res.headers["access-token"]
          );
          window.localStorage.setItem("client", res.headers.client);
          window.localStorage.setItem("uid", res.headers.uid);
          window.localStorage.setItem("name", res.data.data.name);
          // ここまで
          this.$emit("redirectToChatRoom");
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

+ サインアップして試してみる<br>

## 7-3 共通メソッドを管理する

+ `front $ mkdir src/auth && touch $_/setItem.js`を実行<br>

+ `front/src/auth/setItem.js`を編集<br>

```js:setItem.js
const setItem = (headers, name) => {
  window.localStorage.setItem('access-token', headers['access-token'])
  window.localStorage.setItem('client', headers.client)
  window.localStorage.setItem('uid', headers.uid)
  window.localStorage.setItem('name', name)
}

export default setItem
```

+ `front/view/LoginForm.vue`を編集<br>

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
import setItem from "../auth/setItem";

export default {
  emits: ["redirectToChatRoom"],
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

        if (!this.error) {
          setItem(res.headers, res.data.data.name) // 編集
          this.$emit("redirectToChatRoom");
        }

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

+ `front/src/view/SignupForm.vue`を編集<br>

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
import setItem from "../auth/setItem";

export default {
  emits: ["redirectToChatRoom"],
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
          window.localStorage.setItem(
            "access-token",
            res.headers["access-token"]
          );
          setItem(res.headers, res.data.data.name);
          this.$emit("redirectToChatRoom");
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