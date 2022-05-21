## 7-4 ログアウト機能の実装

+ `front/src/components/NavbarMenu.vue`を編集<br>

```vue:NavbarMenu.vue
<template>
  <nav>
    <div>
      <p>こんにちは、{{ name }}さん</p>
      <p class="email">現在、{{ email }}でログイン中です</p>
    </div>
    <button @click="logout">ログアウト</button>
  </nav>
</template>

<script>
import axios from "../api/index";

export default {
  data() {
    return {
      name: window.localStorage.getItem("name"),
      email: window.localStorage.getItem("uid"),
    };
  },
  methods: {
    async logout() {
      try {
        const res = await axios().delete("/auth/sign_out", {
          headers: {
            uid: this.email,
            "access-token": window.localStorage.getItem("access-token"),
            client: window.localStorage.getItem("client"),
          },
        });

        // eslint-disable-next-line no-console
        console.log({ res });

        return res;
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log({ error });
      }
    },
  },
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

+ ログアウトしてみてconsoleを確認してみる<br>

```:console
request: XMLHttpRequest {onreadystatechange: null, readyState: 4, timeout: 0, withCredentials: false, upload: XMLHttpRequestUpload, …}
status: 200
statusText: "OK"
```

+ `front/node_modules/sockjs-client/dist/sock.js`を編集<br>

```js:sock.js
  try {
    // self.xhr.send(payload); この部分をコメントアウト
  } catch (e) {
    self.emit('finish', 0, '');
    self._cleanup(false);
  }
```

+ `front/node_modules/webpack-dev-server/client/index.js`を編集<br>

```js:index.js
close: function close() {
    // log.error('[WDS] Disconnected!');　この部分絵おコメントアウト
    sendMessage('Close');
  }
```

## ローカルストレージから情報を削除する

+ `front/src/components/NavbarMenu.vue`を編集<br>

```vue:NavbarMenu.vue
<template>
  <nav>
    <div>
      <p>こんにちは、{{ name }}さん</p>
      <p class="email">現在、{{ email }}でログイン中です</p>
    </div>
    <button @click="logout">ログアウト</button>
  </nav>
</template>

<script>
import axios from "../api/index";

export default {
  data() {
    return {
      name: window.localStorage.getItem("name"),
      email: window.localStorage.getItem("uid"),
    };
  },
  methods: {
    async logout() {
      try {
        const res = await axios().delete("/auth/sign_out", {
          headers: {
            uid: this.email,
            "access-token": window.localStorage.getItem("access-token"),
            client: window.localStorage.getItem("client"),
          },
        });

        // 追加
        // eslint-disable-next-line no-console
        console.log("ログアウトしました");
        window.localStorage.removeItem("access-token");
        window.localStorage.removeItem("client");
        window.localStorage.removeItem("uid");
        window.localStorage.removeItem("name");
        // ここまで

        return res;
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log({ error });
      }
    },
  },
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

+ ログアウトしてlocalStorageから削除されるか確認してみる<br>

## ログアウトに成功したらリダイレクトさせる

+ `front/components/NavbarMenu.vue`を編集<br>

```vue:NavbarMenu.vue
<template>
  <nav>
    <div>
      <p>こんにちは、{{ name }}さん</p>
      <p class="email">現在、{{ email }}でログイン中です</p>
    </div>
    <button @click="logout">ログアウト</button>
  </nav>
</template>

<script>
import axios from "../api/index";

export default {
  data() {
    return {
      name: window.localStorage.getItem("name"),
      email: window.localStorage.getItem("uid"),
    };
  },
  methods: {
    async logout() {
      try {
        const res = await axios().delete("/auth/sign_out", {
          headers: {
            uid: this.email,
            "access-token": window.localStorage.getItem("access-token"),
            client: window.localStorage.getItem("client"),
          },
        });

        // eslint-disable-next-line no-console
        console.log("ログアウトしました");
        window.localStorage.removeItem("access-token");
        window.localStorage.removeItem("client");
        window.localStorage.removeItem("uid");
        window.localStorage.removeItem("name");

        // 追加
        this.$router.push({ name: "WelcomePage" });

        return res;
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log({ error });
      }
    },
  },
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

+ ログアウトしてログインページにリダイレクトするか確認する<br>

## エラーを表示する

+ `front/components/NavbarMenu.vue`を編集<br>

```vue:NavbarMenu.vue
<template>
  <nav>
    <div>
      <p>こんにちは、{{ name }}さん</p>
      <p class="email">現在、{{ email }}でログイン中です</p>
      <div class="error">{{ error }}</div> // 追加
    </div>
    <button @click="logout">ログアウト</button>
  </nav>
</template>

<script>
import axios from "../api/index";

export default {
  data() {
    return {
      name: window.localStorage.getItem("name"),
      email: window.localStorage.getItem("uid"),
      error: null, // 追加
    };
  },
  methods: {
    async logout() {
      this.error = null; // 追加

      try {
        const res = await axios().delete("/auth/sign_out", {
          headers: {
            uid: this.email,
            "access-token": window.localStorage.getItem("access-token"),
            client: window.localStorage.getItem("client"),
          },
        });

        // 編集
        if (!res) {
          new Error("ログアウトできませんでした");
        }

        if (!this.error) {
          // eslint-disable-next-line no-console
          console.log("ログアウトしました");
          window.localStorage.removeItem("access-token");
          window.localStorage.removeItem("client");
          window.localStorage.removeItem("uid");
          window.localStorage.removeItem("name");

          this.$router.push({ name: "WelcomePage" });
        }
        // ここまで

        return res;
      } catch (error) {
        this.error = "ログアウトできませんでした";
      }
    },
  },
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

## 7-5 アクセスできるページの制御する

### ナビゲーションガードを設定する

+ `front/src/router/index.js`を編集<br>

```js:index.js
import { createRouter, createWebHistory } from 'vue-router'
import WelcomePage from '../views/WelcomePage'
import ChatroomPage from '../views/ChatroomPage'

// 追加
// eslint-disable-next-line no-unused-vars
const requireAuth = async(to, from, next) => {
  // eslint-disable-next-line no-console
  console.log('requiredAuthが呼ばれています!')

  next()
}
// ここまで

const routes = [
  {
    path: '/',
    name: 'WelcomePage',
    component: WelcomePage
  },
  {
    path: '/chatroom-page',
    name: 'ChatroomPage',
    component: ChatroomPage,

    // 追加
    beforeEnter: requireAuth
  }
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router
```

+ ログインしてconsoleを確認してみる<br>

## バリデーション処理を追加する

+ `front $ touch src/auth/validate.js`を実行<br>

+ front/src/auth/validate.js`を編集<br>

```js:validate.js
import axios from '../api/index'

const validate = async () => {
  const uid = window.localStorage.getItem('uid')
  const client = window.localStorage.getItem('client')
  const accessToken = window.localStorage.getItem('access-token')

  try {
    const res = await axios().get('/auth/validate_token', {
      headers: {
        uid: uid,
        'accsess-token': accessToken,
        client: client
      }
    })

    return res
  } catch (err) {
    console.log(err)
  }
}

const useValidate = () => {
  return { validate }
}

export default useValidate
```

## `index.js`ファイルで`validate`メソッドを読み込む

+ `front/src/router/index.js`を編集<br>

```js:index.js
import { createRouter, createWebHistory } from 'vue-router'
import WelcomePage from '../views/WelcomePage'
import ChatroomPage from '../views/ChatroomPage'
import useValidate from '../auth/validate' // 追加

const { validate } = useValidate() // 追加

// 編集
// eslint-disable-next-line no-unused-vars
const requireAuth = async (to, from, next) => {
  const uid = window.localStorage.getItem('uid')
  const client = window.localStorage.getItem('client')
  const accessToken = window.localStorage.getItem('access-token')

  if (!uid || !client || !accessToken) {
    // eslint-disable-next-line no-console
    console.log('ログインしていません')
    next({ name: 'WelcomePage' })
    return
  }

  await validate()
  // ここまで

  next()
}

const routes = [
  {
    path: '/',
    name: 'WelcomePage',
    component: WelcomePage
  },
  {
    path: '/chatroom-page',
    name: 'ChatroomPage',
    component: ChatroomPage,

    beforeEnter: requireAuth
  }
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router
```

+ ログインしていない状態で localhost:8080/chatroom-page にアクセスしてみてログインページに留まったままが確認する<br>

## あえてエラーを発生させる

+ `front/auth/validate.js`を編集<br>

```js:validate.js
import axios from '../api/index'

const validate = async () => {
  const uid = null // 編集
  const client = window.localStorage.getItem('client')
  const accessToken = window.localStorage.getItem('access-token')

  try {
    const res = await axios().get('/auth/validate_token', {
      headers: {
        uid: uid,
        'accsess-token': accessToken,
        client: client
      }
    })

    return res
  } catch (err) {
    console.log(err)
  }
}

const useValidate = () => {
  return { validate }
}

export default useValidate
```

+ 上記の状態でログインしてみる<br>

```:console
[HMR] Waiting for update signal from WDS...
index.js?365c:5 http://localhost:3000
LoginForm.vue?61b1:57 {res: {…}}
index.js?a18c:16 ログインしていません
```

## リダイレクトの処理を修正する

+ `font/auth/validate.js`を編集<br>

```js:validate.js
import { ref } from 'vue' // 追加
import axios from '../api/index'

const error = ref(null) // 追加

const validate = async () => {
  error.value = null // 追加

  const uid = null
  const client = window.localStorage.getItem('client')
  const accessToken = window.localStorage.getItem('access-token')

  try {
    const res = await axios().get('/auth/validate_token', {
      headers: {
        uid: uid,
        'accsess-token': accessToken,
        client: client
      }
    })

    // 追加
    if(!res) {
      throw new Error('認証に失敗しました')
    }
    // ここまで

    return res
  } catch (err) {
    // 編集
    error.value = '認証にしました'

    window.localStorage.removeItem('uid')
    window.localStorage.removeItem('access-token')
    window.localStorage.removeItem('client')
    window.localStorage.removeItem('name')
    // ここまで
  }
}

const useValidate = () => {
  return { error, validate } // 編集
}

export default useValidate
```

※ 参考(ref): https://qiita.com/hareku/items/41c42554e7718aa17483<br>

+ `front/router/index.js`を編集<br>

```js:index.js
import { createRouter, createWebHistory } from 'vue-router'
import WelcomePage from '../views/WelcomePage'
import ChatroomPage from '../views/ChatroomPage'
import useValidate from '../auth/validate'

const { error, validate } = useValidate() // 編集

// eslint-disable-next-line no-unused-vars
const requireAuth = async (to, from, next) => {
  const uid = window.localStorage.getItem('uid')
  const client = window.localStorage.getItem('client')
  const accessToken = window.localStorage.getItem('access-token')

  if (!uid || !client || !accessToken) {
    // eslint-disable-next-line no-console
    console.log('ログインしていません')
    next({ name: 'WelcomePage' })
    return
  }

  await validate()

  // 編集
  if (error.value) {
    // eslint-disable-next-line no-console
    console.log('認証に失敗しました')
    next({ name: 'WelcomePage' })
  } else {
    next()
  }
  // ここまで
}

const routes = [
  {
    path: '/',
    name: 'WelcomePage',
    component: WelcomePage
  },
  {
    path: '/chatroom-page',
    name: 'ChatroomPage',
    component: ChatroomPage,

    beforeEnter: requireAuth
  }
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router
```

+ ログインしてもチャットルームにリダイレクトされていないか確認する<br>

## ログインを成功させる

+ `front/auth/validate.js`を編集<br>

```js:validate.js
import { ref } from 'vue'
import axios from '../api/index'

const error = ref(null)

const validate = async () => {
  error.value = null

  const uid = window.localStorage.getItem('uid') // 編集
  const client = window.localStorage.getItem('client')
  const accessToken = window.localStorage.getItem('access-token')

  try {
    const res = await axios().get('/auth/validate_token', {
      headers: {
        uid: uid,
        'access-token': accessToken,
        client: client
      }
    })

    if(!res) {
      throw new Error('認証に失敗しました')
    }

    return res
  } catch (err) {
    error.value = '認証にしました'

    window.localStorage.removeItem('uid')
    window.localStorage.removeItem('access-token')
    window.localStorage.removeItem('client')
    window.localStorage.removeItem('name')
  }
}

const useValidate = () => {
  return { error, validate }
}

export default useValidate
```

+ `ログインするとチャットルームへリダイレクトするか確認する<br>

## ログインしている時にウェルカムページ にアクセスできないようにする

+ `front/src/router/index.js`を編集<br>

```js:index.js
import { createRouter, createWebHistory } from 'vue-router'
import WelcomePage from '../views/WelcomePage'
import ChatroomPage from '../views/ChatroomPage'
import useValidate from '../auth/validate'

const { error, validate } = useValidate()

// eslint-disable-next-line no-unused-vars
const requireAuth = async (to, from, next) => {
  const uid = window.localStorage.getItem('uid')
  const client = window.localStorage.getItem('client')
  const accessToken = window.localStorage.getItem('access-token')

  if (!uid || !client || !accessToken) {
    // eslint-disable-next-line no-console
    console.log('ログインしていません')
    next({ name: 'WelcomePage' })
    return
  }

  await validate()

  if (error.value) {
    // eslint-disable-next-line no-console
    console.log('認証に失敗しました')
    next({ name: 'WelcomePage' })
  } else {
    next()
  }
}

// 追加
const noRequireAuth = async (to, from, next) => {
  const uid = window.localStorage.getItem('uid')
  const client = window.localStorage.getItem('client')
  const accessToken = window.localStorage.getItem('access-token')

  if (!uid && !client && !accessToken) {
    next()
    return
  }

  await validate()

  if (!error.value) {
    next({ name: 'ChatroomPage' })
  } else {
    next()
  }
}
// ここまで追加

const routes = [
  {
    path: '/',
    name: 'WelcomePage',
    component: WelcomePage,
    beforeEnter: noRequireAuth
  },
  {
    path: '/chatroom-page',
    name: 'ChatroomPage',
    component: ChatroomPage,

    beforeEnter: requireAuth
  }
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router
```

+ ログインしてチャットルームページにいる状態で localhost:8080にアクセスしてみてチャットルームページにリダイレクトされるか確認する<br>
