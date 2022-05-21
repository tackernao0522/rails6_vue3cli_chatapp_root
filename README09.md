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
