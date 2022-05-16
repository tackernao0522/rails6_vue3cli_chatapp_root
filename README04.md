# 6-1 Vue.jsプロジェクトの作成

+ `下記の指定するファイルがあれば削除`<br>

1. src/components/HelloWorld.vue <br>
2. src/views/About.vue <br>
3. src/views/Home.vue <br>
4. src/assets/logo.png <br>

+ `front $ mkdir src/router && touch $_/index.js`を実行<br>

+ `front/src/App.vue`を編集<br>

```vue:App.vue
<template>
  <router-view />
</template>

<script>
export default {};
</script>

<style scoped>
</style>
```

+ `front/src/router/index.js`を編集<br>

```js:index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = []

const router = createRouter({
  history: createWebHistory(process.env.Base_URL),
  routes
})

export default router
```

+ エラーが出た場合は `root $ docker compose run --rm font npm install --save vue-router`を実行<br>
