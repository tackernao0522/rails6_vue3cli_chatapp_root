# 9-1 いいね機能を実装する

+ `front/src/components/ChatWindow.vue`を編集<br>

```vue:ChatWindow.vue
<template>
  <div class="chat-window">
    <div v-if="messages" class="messages">
      <ul v-for="message in messages" :key="message.id">
        <li
          :class="{
            received: message.email !== uid,
            sent: message.email == uid,
          }"
        >
          <span class="name">{{ message.name }}</span>
          // 編集
          <div class="message" @dblclick="createLike(message.id)">
            {{ message.content }}
            {{ message.likes.length }}
          </div>
          // ここまで
          <span class="created-at">{{ message.created_at }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from "../api/index";

export default {
  props: ["messages"],
  data() {
    return {
      uid: localStorage.getItem("uid"),
    };
  },
  // 追加
  methods: {
    async createLike(messageId) {
      try {
        const res = await axios().post(
          `/messages/${messageId}/likes`,
          {},
          {
            headers: {
              uid: this.uid,
              "access-token": window.localStorage.getItem("access-token"),
              client: window.localStorage.getItem("client"),
            },
          }
        );

        if (!res) {
          new Error("いいねできませんでした");
        }
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log(error);
      }
    },
  },
  // ここまで
};
</script>

<style scoped>
.chat-window {
  background: white;
  padding: 30px 20px;
  border-bottom: 1px solid #eee;
}
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
  position: relative;
  margin-right: 6px;
  display: block;
  font-size: 13px;
}
.created-at {
  display: block;
  color: #999;
  font-size: 12px;
  margin-bottom: 20px;
  margin-left: 4px;
}
.messages {
  max-height: 400px;
  overflow: auto;
}
</style>
```

+ メッセージの部分をダブルクリックしてリロードすると `1` が付くようになる<br>

## アイコンを表示する

+ `root $ docker compose run --rm front npm i --save @fortawesome/vue-fontawesome@prerelease`を実行<br>

+ `root $ docker compose run --rm front npm i --save @fortawesome/fontawesome-svg-core@1.2.35`を実行<br>

+ `root $ docker compose run --rm front npm i --save @fortawesome/free-solid-svg-icons@5.15.3`を実行<br>

+ `front/src/main.js`を編集<br>

```js:main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import './assets/main.css'
// 追加
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome"
import { library } from "@fortawesome/fontawesome-svg-core"
import { faHeart } from "@fortawesome/free-solid-svg-icons"
// ここまで

// 編集
library.add(faHeart)
createApp(App).use(router).component("font-awesome-icon", FontAwesomeIcon).mount('#app') // 編集
```

+ `front/src/components/ChatWindow.vue`を編集<br>

```vue:ChatWindow.vue
<template>
  <div class="chat-window">
    <div v-if="messages" class="messages">
      <ul v-for="message in messages" :key="message.id">
        <li
          :class="{
            received: message.email !== uid,
            sent: message.email == uid,
          }"
        >
          <span class="name">{{ message.name }}</span>
          <div class="message" @dblclick="createLike(message.id)">
            {{ message.content }}
            // 編集
            <div v-if="message.likes.length" class="heart-container">
              <font-awesome-icon icon="heart" class="heart" />
              <span class="heart-count">{{ message.likes.length }}</span>
            </div>
            // ここまで
          </div>
          <span class="created-at">{{ message.created_at }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from "../api/index";

export default {
  props: ["messages"],
  data() {
    return {
      uid: localStorage.getItem("uid"),
    };
  },
  methods: {
    async createLike(messageId) {
      try {
        const res = await axios().post(
          `/messages/${messageId}/likes`,
          {},
          {
            headers: {
              uid: this.uid,
              "access-token": window.localStorage.getItem("access-token"),
              client: window.localStorage.getItem("client"),
            },
          }
        );

        if (!res) {
          new Error("いいねできませんでした");
        }
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log(error);
      }
    },
  },
};
</script>

<style scoped>
.chat-window {
  background: white;
  padding: 30px 20px;
  border-bottom: 1px solid #eee;
}
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
  position: relative;
  margin-right: 6px;
  display: block;
  font-size: 13px;
}
.created-at {
  display: block;
  color: #999;
  font-size: 12px;
  margin-bottom: 20px;
  margin-left: 4px;
}
.messages {
  max-height: 400px;
  overflow: auto;
}
</style>
```

+ ブラウザをリロードするといいねがついているメッセージにハートアイコンといいねの数が表示されている<br>


## CSSを適用する

+ `front/src/components/ChatWindow.vue`を編集<br>

```vue:ChatWindow.vue
<template>
  <div class="chat-window">
    <div v-if="messages" class="messages">
      <ul v-for="message in messages" :key="message.id">
        <li
          :class="{
            received: message.email !== uid,
            sent: message.email == uid,
          }"
        >
          <span class="name">{{ message.name }}</span>
          <div class="message" @dblclick="createLike(message.id)">
            {{ message.content }}
            <div v-if="message.likes.length" class="heart-container">
              <font-awesome-icon icon="heart" class="heart" />
              <span class="heart-count">{{ message.likes.length }}</span>
            </div>
          </div>
          <span class="created-at">{{ message.created_at }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from "../api/index";

export default {
  props: ["messages"],
  data() {
    return {
      uid: localStorage.getItem("uid"),
    };
  },
  methods: {
    async createLike(messageId) {
      try {
        const res = await axios().post(
          `/messages/${messageId}/likes`,
          {},
          {
            headers: {
              uid: this.uid,
              "access-token": window.localStorage.getItem("access-token"),
              client: window.localStorage.getItem("client"),
            },
          }
        );

        if (!res) {
          new Error("いいねできませんでした");
        }
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log(error);
      }
    },
  },
};
</script>

<style scoped>
.chat-window {
  background: white;
  padding: 30px 20px;
  border-bottom: 1px solid #eee;
}
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
  position: relative;
  margin-right: 6px;
  display: block;
  font-size: 13px;
}
.created-at {
  display: block;
  color: #999;
  font-size: 12px;
  margin-bottom: 20px;
  margin-left: 4px;
}
.messages {
  max-height: 400px;
  overflow: auto;
}
// 追加
.message {
  position: relative;
}

.heart-container {
  background: white;
  position: absolute;
  display: flex;
  justify-content: space-around;
  align-items: center;
  border-radius: 30px;
  min-width: 25px;
  border-style: solid;
  border-width: 1px;
  border-color: rgb(245, 245, 245);
  padding: 1px 2px;
  z-index: 2;
  bottom: -7px;
  right: 0px;
  font-size: 9px;
}
.heart {
  color: rgb(236, 29, 29);
}
.heart-count {
  color: rgb(20, 19, 19);
}
ここまで
</style>
```

+ リロードしてみる<br>

## いいねがすぐに反映するようにする

+ `front/components/ChatWindow.vue`を編集<br>

```vue:ChatWindow.vue
<template>
  <div class="chat-window">
    <div v-if="messages" class="messages">
      <ul v-for="message in messages" :key="message.id">
        <li
          :class="{
            received: message.email !== uid,
            sent: message.email == uid,
          }"
        >
          <span class="name">{{ message.name }}</span>
          <div class="message" @dblclick="createLike(message.id)">
            {{ message.content }}
            <div v-if="message.likes.length" class="heart-container">
              <font-awesome-icon icon="heart" class="heart" />
              <span class="heart-count">{{ message.likes.length }}</span>
            </div>
          </div>
          <span class="created-at">{{ message.created_at }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from "../api/index";

export default {
  emits: ["connectCable"], // 追加
  props: ["messages"],
  data() {
    return {
      uid: localStorage.getItem("uid"),
    };
  },
  methods: {
    async createLike(messageId) {
      try {
        const res = await axios().post(
          `/messages/${messageId}/likes`,
          {},
          {
            headers: {
              uid: this.uid,
              "access-token": window.localStorage.getItem("access-token"),
              client: window.localStorage.getItem("client"),
            },
          }
        );

        if (!res) {
          new Error("いいねできませんでした");
        }
        this.$emit("connectCable"); // 追加
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log(error);
      }
    },
  },
};
</script>

<style scoped>
.chat-window {
  background: white;
  padding: 30px 20px;
  border-bottom: 1px solid #eee;
}
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
  position: relative;
  margin-right: 6px;
  display: block;
  font-size: 13px;
}
.created-at {
  display: block;
  color: #999;
  font-size: 12px;
  margin-bottom: 20px;
  margin-left: 4px;
}
.messages {
  max-height: 400px;
  overflow: auto;
}

.message {
  position: relative;
}

.heart-container {
  background: white;
  position: absolute;
  display: flex;
  justify-content: space-around;
  align-items: center;
  border-radius: 30px;
  min-width: 25px;
  border-style: solid;
  border-width: 1px;
  border-color: rgb(245, 245, 245);
  padding: 1px 2px;
  z-index: 2;
  bottom: -7px;
  right: 0px;
  font-size: 9px;
}
.heart {
  color: rgb(236, 29, 29);
}
.heart-count {
  color: rgb(20, 19, 19);
}
</style>
```

+ `front/components/ChatroomPage.vue`を編集<br>

```vue:ChatroomPage.vue
<template>
  <div class="container">
    <navbar-menu />
    <chat-window @connectCable="connectCable" :messages="messages" /> // 編集
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
      const cable = ActionCable.createConsumer("ws://localhost:3000/cable");
      this.messageChannel = cable.subscriptions.create("RoomChannel", {
        connected: () => {
          this.getMessages();
        },
        received: () => {
          this.getMessages();
        },
      });
    } else if (
      process.env.VUE_APP_API_ORIGIN ==
      "https://rails-vue3cli-api.herokuapp.com"
    ) {
      const cable = ActionCable.createConsumer(
        "wss://rails-vue3cli-api.herokuapp.com/cable"
      );
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

+ いいね してみるとすぐに反映される<br>

## CSSで微調整を加える

+ `front/components/ChatWindow.vue`を編集<br>

```vue:ChatWindow.vue
<template>
  <div class="chat-window">
    <div v-if="messages" class="messages">
      <ul v-for="message in messages" :key="message.id">
        <li
          :class="{
            received: message.email !== uid,
            sent: message.email == uid,
          }"
        >
          <span class="name">{{ message.name }}</span>
          <div class="message" @dblclick="createLike(message.id)">
            {{ message.content }}
            <div v-if="message.likes.length" class="heart-container">
              <font-awesome-icon icon="heart" class="heart" />
              <span class="heart-count">{{ message.likes.length }}</span>
            </div>
          </div>
          <span class="created-at">{{ message.created_at }}</span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from "../api/index";

export default {
  emits: ["connectCable"],
  props: ["messages"],
  data() {
    return {
      uid: localStorage.getItem("uid"),
    };
  },
  methods: {
    async createLike(messageId) {
      try {
        const res = await axios().post(
          `/messages/${messageId}/likes`,
          {},
          {
            headers: {
              uid: this.uid,
              "access-token": window.localStorage.getItem("access-token"),
              client: window.localStorage.getItem("client"),
            },
          }
        );

        if (!res) {
          new Error("いいねできませんでした");
        }
        this.$emit("connectCable");
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log(error);
      }
    },
  },
};
</script>

<style scoped>
.chat-window {
  background: white;
  padding: 30px 20px;
  border-bottom: 1px solid #eee;
}
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
  position: relative;
  margin-right: 6px;
  display: block;
  font-size: 13px;
}
.created-at {
  display: block;
  color: #999;
  font-size: 12px;
  margin-bottom: 20px;
  margin-left: 4px;
}
.messages {
  max-height: 400px;
  overflow: auto;
}

.message {
  position: relative;
}

.heart-container {
  background: white;
  position: absolute;
  display: flex;
  justify-content: space-around;
  align-items: center;
  border-radius: 30px;
  min-width: 25px;
  border-style: solid;
  border-width: 1px;
  border-color: rgb(245, 245, 245);
  padding: 1px 2px;
  z-index: 2;
  bottom: -7px;
  right: 0px;
  font-size: 9px;
}
.heart {
  color: rgb(236, 29, 29);
}
.heart-count {
  color: rgb(20, 19, 19);
}
// 追加
.received .message::selection {
  background: #eee;
}

.sent .message::selection {
  background: #677bb4;
}
// ここまで
</style>
```

+ これでいいねの時にメッセージ部分をクリックしてもテキスト選択部分の色が変更されなくなる<br>
