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

## Heroku production Actioncableの設定

※ 参考: https://tech-univ.com/2021/11/12/%E3%82%B3%E3%83%A1%E3%83%B3%E3%83%88%E6%A9%9F%E8%83%BD%E3%82%92%E5%AE%9F%E8%A3%85%E3%81%97%E3%82%88%E3%81%86%E3%80%9Caction-cable%E3%80%9C/ <br>

+ `api/config/cable.yml`を編集<br>

```yml:cable.yml
development:
  adapter: async

test:
  adapter: test

production:
  adapter: async # 編集
  url: <%= ENV.fetch("REDIS_URL") { "redis://localhost:6379/1" } %>
  channel_prefix: app_production
```

+ `api/config/production.rb`を編集<br>

```rb:production.rb
Rails.application.configure do
  # Settings specified here will take precedence over those in config/application.rb.

  # Code is not reloaded between requests.
  config.cache_classes = true

  # Eager load code on boot. This eager loads most of Rails and
  # your application in memory, allowing both threaded web servers
  # and those relying on copy on write to perform better.
  # Rake tasks automatically ignore this option for performance.
  config.eager_load = true

  # Full error reports are disabled and caching is turned on.
  config.consider_all_requests_local       = false

  # Ensures that a master key has been made available in either ENV["RAILS_MASTER_KEY"]
  # or in config/master.key. This key is used to decrypt credentials (and other encrypted files).
  # config.require_master_key = true

  # Disable serving static files from the `/public` folder by default since
  # Apache or NGINX already handles this.
  config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present?

  # Enable serving of images, stylesheets, and JavaScripts from an asset server.
  # config.action_controller.asset_host = 'http://assets.example.com'

  # Specifies the header that your server uses for sending files.
  # config.action_dispatch.x_sendfile_header = 'X-Sendfile' # for Apache
  # config.action_dispatch.x_sendfile_header = 'X-Accel-Redirect' # for NGINX

  # Store uploaded files on the local file system (see config/storage.yml for options).
  config.active_storage.service = :local

  # Mount Action Cable outside main process or domain.
  # config.action_cable.mount_path = nil
  # config.action_cable.url = 'wss://example.com/cable'
  # config.action_cable.allowed_request_origins = [ 'http://example.com', /http:\/\/example.*/ ]

  ActionCable.server.config.disable_request_forgery_protection = true
  config.action_cable.url = "wss://rails-vue3cli-api.herokuapp.com/cable"
  config.action_cable.allowed_request_origins = ['https://rails-vue3cli-api.herokuapp.com', 'http://rails-vue3cli-api.herokuapp.com']

  # Force all access to the app over SSL, use Strict-Transport-Security, and use secure cookies.
  # config.force_ssl = true

  # Use the lowest log level to ensure availability of diagnostic information
  # when problems arise.
  config.log_level = :debug

  # Prepend all log lines with the following tags.
  config.log_tags = [ :request_id ]

  # Use a different cache store in production.
  # config.cache_store = :mem_cache_store

  # Use a real queuing backend for Active Job (and separate queues per environment).
  # config.active_job.queue_adapter     = :resque
  # config.active_job.queue_name_prefix = "app_production"

  config.action_mailer.perform_caching = false

  # Ignore bad email addresses and do not raise email delivery errors.
  # Set this to true and configure the email server for immediate delivery to raise delivery errors.
  # config.action_mailer.raise_delivery_errors = false

  # Enable locale fallbacks for I18n (makes lookups for any locale fall back to
  # the I18n.default_locale when a translation cannot be found).
  config.i18n.fallbacks = true

  # Send deprecation notices to registered listeners.
  config.active_support.deprecation = :notify

  # Use default logging formatter so that PID and timestamp are not suppressed.
  config.log_formatter = ::Logger::Formatter.new

  # Use a different logger for distributed setups.
  # require 'syslog/logger'
  # config.logger = ActiveSupport::TaggedLogging.new(Syslog::Logger.new 'app-name')

  # 追加
  if ENV["RAILS_LOG_TO_STDOUT"].present?
    logger           = ActiveSupport::Logger.new(STDOUT)
    logger.formatter = config.log_formatter
    config.logger    = ActiveSupport::TaggedLogging.new(logger)
  end
  # ここまで

  # Do not dump schema after migrations.
  config.active_record.dump_schema_after_migration = false

  # Inserts middleware to perform automatic connection switching.
  # The `database_selector` hash is used to pass options to the DatabaseSelector
  # middleware. The `delay` is used to determine how long to wait after a write
  # to send a subsequent read to the primary.
  #
  # The `database_resolver` class is used by the middleware to determine which
  # database is appropriate to use based on the time delay.
  #
  # The `database_resolver_context` class is used by the middleware to set
  # timestamps for the last write to the primary. The resolver uses the context
  # class timestamps to determine how long to wait before reading from the
  # replica.
  #
  # By default Rails will store a last write timestamp in the session. The
  # DatabaseSelector middleware is designed as such you can define your own
  # strategy for connection switching and pass that into the middleware through
  # these configuration options.
  # config.active_record.database_selector = { delay: 2.seconds }
  # config.active_record.database_resolver = ActiveRecord::Middleware::DatabaseSelector::Resolver
  # config.active_record.database_resolver_context = ActiveRecord::Middleware::DatabaseSelector::Resolver::Session
end
```

+ `api git push heroku`でデプロイしてテストしてみる<br>
