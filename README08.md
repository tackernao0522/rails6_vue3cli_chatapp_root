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
