# 11-1 リファクタリングをする

## N+1問題を解決する(Ruby on Rails includesを使用)

+ `api/app/controllers/messages_controller.rb`を編集<br>

```rb:messages_controller.rb
class MessagesController < ApplicationController
  before_action :authenticate_user!, only: ["index"]

  def index
    messages = Message.includes(:user, [likes: :user]) # 編集
    messages_array = messages.map do |message|
      {
        id: message.id,
        user_id: message.user.id,
        name: message.user.name,
        content: message.content,
        email: message.user.email,
        created_at: message.created_at,
        likes: message.likes.map { |like| { id: like.id, email: like.user.email } }
      }
    end

    render json: messages_array, status: 200
  end
end
```

## ステータスコードを使用する(Ruby on Rails)

+ `api/controllers/messages_controller.rb`を編集<br>

```rb:messages_controller.rb
class MessagesController < ApplicationController
  before_action :authenticate_user!, only: ["index"]

  def index
    messages = Message.includes(:user, [likes: :user])
    messages_array = messages.map do |message|
      {
        id: message.id,
        user_id: message.user.id,
        name: message.user.name,
        content: message.content,
        email: message.user.email,
        created_at: message.created_at,
        likes: message.likes.map { |like| { id: like.id, email: like.user.email } }
      }
    end

    render json: messages_array, status: :ok # 編集
  end
end
```

+ `api/app/controllers/likes_controller.rb`を編集<br>

```rb:likes_controller.rb
class LikesController < ApplicationController
  before_action :authenticate_user!, only: ['create', 'destroy']

  def create
    like = Like.new(message_id: params[:id], user_id: current_user.id)

    if like.save
      # 編集
      render json: { id: like.id, email: current_user.uid, message: '成功しました' }, status: :ok
    else
      render json: { message: '保存できませんでした', errors: like.errors.messages }, status: :bad_request
    end
  end

  def destroy
    like = Like.find(params[:id])

    if like.destroy
      # 編集
      render json: { id: like.id, email: like.user.uid, message: '削除に成功しました' }, status: :ok
    else
      # 編集
      render json: { message: '削除できませんでした', errors: like.errors.messages }, status: :bad_request
    end
  end
end
```

## 共通メソッド化する(Vue.js)

+ `front touch src/auth/removeItem.js`を実行<br>

+ `front/src/auth/removeItem.js`を編集<br>

```js:removeItem.js
const removeItem = () => {
  window.localStorage.removeItem('uid')
  window.localStorage.removeItem('access-token')
  window.localStorage.removeItem('client')
  window.localStorage.removeItem('name')
}

export default removeItem
```

+ `front/auth/validate.js`を編集<br>

```js:validate.js
import { ref } from 'vue'
import axios from '../api/index'
import removeItem from './removeItem' // 追加

const error = ref(null)

const validate = async () => {
  error.value = null

  const uid = window.localStorage.getItem('uid')
  const client = window.localStorage.getItem('client')
  const accessToken = window.localStorage.getItem('access-token')

  try {
    const res = await axios().get('/auth/validate_token', {
      headers: {
        uid: uid,
        "access-token": accessToken,
        client: client,
      }
    })

    if (!res) {
      throw new Error('認証に失敗しました')
    }

    error.value = null

    return res
  } catch (err) {
    error.value = '認証に失敗しました'
    // 編集
    removeItem()
  }
}

const useValidate = () => {
  return { error, validate }
}

export default useValidate
```

+ `front/src/components/NavbarMenu.vue`を編集<br>

```vue:NavbarMenu.vue
<template>
  <nav>
    <div>
      <p>
        こんにちは、<span class="name">{{ name }}</span
        >さん
      </p>
      <p class="email">現在、 {{ email }} でログイン中です</p>
      <div class="error">{{ error }}</div>
    </div>
    <button @click="logout">ログアウト</button>
  </nav>
</template>

<script>
import axios from "../api/index";
import removeItem from "../auth/removeItem"; // 追加

export default {
  data() {
    return {
      name: window.localStorage.getItem("name"),
      email: window.localStorage.getItem("uid"),
      error: null,
    };
  },
  methods: {
    async logout() {
      this.error = null;

      try {
        const res = await axios().delete("/auth/sign_out", {
          headers: {
            uid: this.email,
            "access-token": window.localStorage.getItem("access-token"),
            client: window.localStorage.getItem("client"),
          },
        });

        if (!res) {
          new Error("ログアウトできませんでした");
        }

        if (!this.error) {
          console.log("ログアウトしました");
          removeItem(); // 編集
          this.$router.push({ name: "WelcomePage" });
        }

        this.error = null;

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
