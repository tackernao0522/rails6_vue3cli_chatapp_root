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
<template>
  <div class="chat-window">
    <div v-if="messages" class="messages">
      <ul v-for="message in messages" :key="message.id">
        // 編集
        <li
          :class="{
            received: message.email !== uid,
            sent: message.email === uid,
          }"
        >
        // ここまで
          <span class="name">{{ message.name }}</span>
          <span class="message">{{ message.content }}</span>
          <span class="created-at">{{ message.created_at }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  props: ["messages"],
  data() {
    return {
      uid: localStorage.getItem("uid"),
    };
  },
};
</script>

<style scoped>
.chat-window {
  background: white;
  padding: 30px 20px;
  border-bottom: 1px solid #eee;
}
// 追加
ul {
  list-style: none;
  margin: 0;
  padding: 0;
}
ul li {
  display: inline-block;
  clear: both;
}
.received .message {
  background: #eee;
  padding: 10px;
  display: inline-block;
  border-radius: 30px;
  margin-bottom: 2px;
  max-width: 400px;
}
.received {
  float: left;
}
.sent {
  float: right;
}
.sent .message {
  background: #677bb4;
  color: white;
  padding: 10px;
  display: inline-block;
  border-radius: 30px;
  margin-bottom: 2px;
  max-width: 400px;
}
.name {
  position: block;
  color: #999;
  font-size: 12px;
  margin-bottom: 20px;
  margin-left: 4px;
}
.messages {
  max-height: 400px;
  overflow: auto;
}
// ここまで
</style>
```

## 8-2 チャット入力機能を実装する

+ `front touch src/components/NewChatForm.vue`を実行<br>

+ `front/src/components/NewChatForm.vue`を編集<br>

```vue:NewChatForm.vue
<template>
  <form>
    <textarea
      placeholder="メッセージを入力してEnterを押してください"
      v-model="message"
    ></textarea>
  </form>
</template>

<script>
export default {
  data() {
    return {
      messags: "",
    };
  },
};
</script>

<style scoped>
form {
  margin: 10px;
}
textarea {
  width: 100%;
  max-width: 100%;
  margin-bottom: 6px;
  padding: 10px;
  box-sizing: border-box;
  border: 0;
  border-radius: 20px;
  font-family: inherit;
  outline: none;
}
</style>
```

+ `front/src/views/ChatroomPage.vue`を編集<br>

```vue:ChatroomPage.vue
<template>
  <div class="container">
    <navbar-menu />
    <chat-window :messages="messages" />
    <new-chat-form /> // 追加
  </div>
</template>

<script>
import axios from "../api/index";
import ChatWindow from "../components/ChatWindow.vue";
import NavbarMenu from "../components/NavbarMenu.vue";
import NewChatForm from "../components/NewChatForm.vue"; // 追加

export default {
  components: { NavbarMenu, ChatWindow, NewChatForm },　// 編集
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
};
</script>

<style scoped>
</style>
```
## Aaction Cableライブラリを導入する

+ `root $ docker compose run --rm front npm install --save actioncable`を実行<br>

+ `front/src/views/ChatroomPage.vue`を編集<br>

```vue:ChatroomPage.vue
<template>
  <div class="container">
    <navbar-menu />
    <chat-window :messages="messages" />
    <new-chat-form />
  </div>
</template>

<script>
import axios from "../api/index";
import ChatWindow from "../components/ChatWindow.vue";
import NavbarMenu from "../components/NavbarMenu.vue";
import NewChatForm from "../components/NewChatForm.vue";
import ActionCable from 'actioncable'; // 追加

export default {
  components: { NavbarMenu, ChatWindow, NewChatForm },
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
  // 編集
  mounted() {
    const cable = ActionCable.createConsumer("ws://localhost:3000/cable");
    this.messageChannel = cable.subscriptions.create("RoomChannel", {
      connected: () => {
        this.getMessages();
      },
      received: () => {
        this.getMessages();
      },
    });
  },
  beforeUnmount() {
    this.messageChannel.unsubscribe();
  },
  // ここまで
};
</script>

<style scoped>
</style>
```

## メッセージを送信できるようにする

+ `front/components/NewChatForm.vue`を編集<br>

```vue:NewChatForm.vue
<template>
  <form>
    <textarea
      placeholder="メッセージを入力してEnterを押してください"
      v-model="message"
      @keypress.enter.prevent="handleSubmit" // 追加
    ></textarea>
  </form>
</template>

<script>
export default {
  emits: ["connectCable"],
  data() {
    return {
      messags: "",
    };
  },
  // 追加
  methods: {
    handleSubmit() {
      this.$emit("connectCable", this.messags);
      this.message = "";
    },
  },
  // ここまで
};
</script>

<style scoped>
form {
  margin: 10px;
}
textarea {
  width: 100%;
  max-width: 100%;
  margin-bottom: 6px;
  padding: 10px;
  box-sizing: border-box;
  border: 0;
  border-radius: 20px;
  font-family: inherit;
  outline: none;
}
</style>
```

## connectCableを実行する

+ `front/src/ChatroomPage.vue`を編集<br>

```vue:ChatroomPage.vue
<template>
  <div class="container">
    <navbar-menu />
    <chat-window :messages="messages" />
    <new-chat-form @connectCable="connectCable" />
  </div>
</template>

<script>
import axios from "../api/index";
import ActionCable from "actioncable";
import ChatWindow from "../components/ChatWindow.vue";
import NewChatForm from "../components/NewChatForm.vue";
import NavbarMenu from "../components/NavbarMenu.vue";

export default {
  components: { NavbarMenu, ChatWindow, NewChatForm },
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
        this.error = "メッセージ一覧を取得できませんでした";
      }
    },
    connectCable(message) {
      this.messageChannel.perform("receive", {
        message: message,
        email: window.localStorage.getItem("uid"),
      });
    },
  },
  mounted() {
    if (process.env.VUE_APP_API_ORIGIN == "http://localhost:3000") {
      const cable = ActionCable.createConsumer(`ws://localhost:3000/cable`);
      this.messageChannel = cable.subscriptions.create("RoomChannel", {
        connected: () => {
          this.getMessages();
        },
        received: () => {
          this.getMessages();
        },
      });
    } else {
      const cable = ActionCable.createConsumer(`wss://rails-vue3cli-api.herokuapp.com/cable`);
      this.messageChannel = cable.subscriptions.create("RoomChannel", {
        connected: () => {
          this.getMessages();
        },
        received: () => {
          this.getMessages();
        },
      });
    }
  },
  beforeUnmount() {
    this.messageChannel.unsubscribe();
  },
};
</script>

<style scoped>
</style>
```