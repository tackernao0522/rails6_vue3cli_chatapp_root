# 8-1 メッセージ一覧を表示する

## コンポーネントを作成する

+ `front touch src/components/ChatWindow.vue`を実行<br>

+ `front/src/components/ChatWindow.vue`を編集<br>

```vue:ChatWindow.vue
<template>
  <div class="chat-window">チャットウィンドウです</div>
</template>

<script>
export default {};
</script>

<style scoped>
.chat-window {
  background: white;
  padding: 30px 20px;
  border-bottom: 1px solid #eee;
}
</style>
```

+ `front/src/views/ChatroomPage.vue`を編集<br>

```vue:ChatroomPage.vue
<template>
  <div class="container">
    <navbar-menu />
    <chat-window /> // 追加
  </div>
</template>

<script>
import ChatWindow from "../components/ChatWindow.vue"; // 追加
import NavbarMenu from "../components/NavbarMenu.vue";

export default {
  components: { NavbarMenu, ChatWindow }, // 編集
};
</script>

<style scoped>
</style>
```

## メッセージの一覧を取得する

+ `front/views/ChatroomPage.vue`を編集<br>

```vue:ChatroomPage.vue
<template>
  <div class="container">
    <navbar-menu />
    <chat-window :messages="messages" /> // 編集
  </div>
</template>

<script>
import axios from "../api/index"; // 追加
import ChatWindow from "../components/ChatWindow.vue";
import NavbarMenu from "../components/NavbarMenu.vue";

export default {
  components: { NavbarMenu, ChatWindow }, // 編集
  // 追加
  data() {
    return {
      messages: [],
    };
  },
  methods: {
    async getMessages() {
      try {
        const res = await axios().get("/messages", {
          headers: {
            uid: window.localStorage.getItem("uid"),
            "access-token": window.localStorage.getItem("access-token"),
            client: window.localStorage.getItem("client"),
          },
        });
        if (!res) {
          new Error("メッセージ一覧を取得できませんでした");
        }
        this.messages = res.data;
      } catch (err) {
        // eslint-disable-next-line no-console
        console.log(err);
      }
    },
  },
  mounted() {
    this.getMessages();
  },
  // ここまで
};
</script>

<style scoped>
</style>
```

参考(mounted): https://qiita.com/chan_kaku/items/7f3233053b0e209ef355<br>

https://v3.ja.vuejs.org/guide/composition-api-lifecycle-hooks.html<br>

## コンポーネントでデータを受け取る

+ `front/src/components/ChatWindow.vue`を編集<br>

```vue:ChatWindow.vue
<template>
  <div class="chat-window">
    {{ messages }} // 編集
  </div>
</template>

<script>
export default {
  props: ["messages"], // 追加
};
</script>

<style scoped>
.chat-window {
  background: white;
  padding: 30px 20px;
  border-bottom: 1px solid #eee;
}
</style>
```

+ ログインしてChatroomPageを確認してみる<br>

# メッセージ一覧を綺麗に表示する

## messagesをループさせる

+ `front/components/ChatWindow.vue`を編集<br>

```vue:ChatWindow.vue
<template>
  <div class="chat-window">
    // 編集
    <div v-if="messages" class="messages">
      <ul v-for="message in messages" :key="message.id">
        <li>
          <span class="name">{{ message.name }}</span>
          <span class="message">{{ message.content }}</span>
          <span class="created-at">{{ message.created_at }}</span>
        </li>
      </ul>
    </div>
    // ここまで
  </div>
</template>

<script>
export default {
  props: ["messages"],
};
</script>

<style scoped>
.chat-window {
  background: white;
  padding: 30px 20px;
  border-bottom: 1px solid #eee;
}
</style>
```

## CSSを適用する

+ `front/src/components/ChatWindow.vue`を編集<br>

```vue:ChatWindow.vue

```