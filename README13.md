# 10-2 チャット画面を自動スクロールする

+ `font/src/components/ChatWindow.vue`を編集<br>

````vue:ChatWindow.vue
<template>
  <div class="chat-window">
    <div v-if="messages" class="messages" ref="messages"> // 編集
      <ul v-for="message in messages" :key="message.id">
        <li
          :class="{
            received: message.email !== uid,
            sent: message.email == uid,
          }"
        >
          <span class="name">{{ message.name }}</span>
          <div class="message" @dblclick="handleLike(message)">
            {{ message.content }}
            <div v-if="message.likes.length" class="heart-container">
              <font-awesome-icon icon="heart" class="heart" />
              <span class="heart-count">{{ message.likes.length }}</span>
            </div>
          </div>
          <span class="created-at">{{ message.created_at }}前</span>
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
    handleLike(message) {
      for (let i = 0; i < message.likes.length; i++) {
        const like = message.likes[i];
        if (like.email === this.uid) {
          this.deleteLike(like.id);
          return;
        }
      }
      this.createLike(message.id);
    },
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
    async deleteLike(likeId) {
      try {
        const res = await axios().delete(`/likes/${likeId}`, {
          headers: {
            uid: this.uid,
            "access-token": window.localStorage.getItem("access-token"),
            client: window.localStorage.getItem("client"),
          },
        });

        if (!res) {
          new Error("いいねを削除できませんでした");
        }
        this.$emit("connectCable");
      } catch (error) {
        // eslint-disable-next-line no-console
        console.log(error);
      }
    },
    // 追加
    scrollToBottom() {
      const element = this.$refs.messages;
      element.scrollTop = element.scrollHeight;
    },
    // ここまで
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
.received .message::selection {
  background: #eee;
}

.sent .message::selection {
  background: #677bb4;
}
</style>
```

## メソッドを実行させる

+ `front/views/ChatroomPage.vue`を編集<br>

```vue:ChatroomPage.vue
<template>
  <div class="container">
    <navbar-menu />
    <chat-window
      @connectCable="connectCable"
      :messages="formattedMessages"
      ref="chatWindow" // 追加
    />
    <new-chat-form @connectCable="connectCable" />
  </div>
</template>

<script>
import axios from "../api/index";
import ActionCable from "actioncable";
import ChatWindow from "../components/ChatWindow.vue";
import NewChatForm from "../components/NewChatForm.vue";
import NavbarMenu from "../components/NavbarMenu.vue";
import { formatDistanceToNow } from "date-fns";
import { ja } from "date-fns/locale";

export default {
  components: { NavbarMenu, ChatWindow, NewChatForm },
  data() {
    return {
      messages: [],
    };
  },
  computed: {
    formattedMessages() {
      if (!this.messages.length) {
        return [];
      }
      return this.messages.map((message) => {
        let time = formatDistanceToNow(new Date(message.created_at), {
          locale: ja,
        });
        return { ...message, created_at: time };
      });
    },
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
          // 編集
          this.getMessages().then(() => {
            this.$refs.chatWindow.scrollToBottom();
          });
        },
        received: () => {
          this.getMessages().then(() => {
            this.$refs.chatWindow.scrollToBottom();
          });
        },
        // ここまで
      });
    } else {
      const cable = ActionCable.createConsumer(
        "wss://rails-vue3cli-api.herokuapp.com/cable"
      );
      this.messageChannel = cable.subscriptions.create("RoomChannel", {
        connected: () => {
          // 編集
          this.getMessages().then(() => {
            this.$refs.chatWindow.scrollToBottom();
          });
        },
        received: () => {
          this.getMessages().then(() => {
            this.$refs.chatWindow.scrollToBottom();
          });
        },
        // ここまで
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

## 本番環境のリロードエラーの解消

+ `front/server.js`を編集<br>

```js:server.js
const express = require('express');
const path = require('path');
const history = require('connect-history-api-fallback');

const app = express();

const staticFileMiddleware = express.static(path.join(__dirname + '/dist'));

app.use(staticFileMiddleware);
app.use(history({
  disableDotRule: true,
  verbose: true
}));
app.use(staticFileMiddleware);

app.get('/', function (req, res) {
  res.render(path.join(__dirname + '/dist/index.html'));
});

var server = app.listen(process.env.PORT || 3000, function () {
  var port = server.address().port;
  console.log("App now running on port", port);
});
```
