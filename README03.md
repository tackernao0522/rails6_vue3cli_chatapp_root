# 4-1 Likesテーブルとモデルを作成

|likes|
|:---:|
|id|
|message_id|
|user_id|
|created_at|
|updated_at|

|いいね|
|:---:|
|id|
|いいねしたメッセージのID|
|いいねしたユーザのID|

+ `root $ docker compose run --rm api rails g model like`を実行<br>

+ `api/db/migrate/create_likes.rb`を編集<br>

```rb:create_likes.rb
class CreateLikes < ActiveRecord::Migration[6.0]
  def change
    create_table :likes do |t|
      t.references :user
      t.references :message
      t.timestamps
    end
  end
end
```

+ `root $ docker compose run --rm api rails db:migrate`を実行<br>

+ `root $ docker compose run --rm api rails db:migrate:statue`を実行して確認<br>

```:terminal
database: app_development

 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20220515092932  Devise token auth create users
   up     20220515113850  Create messages
   up     20220516020513  Create likes
```

## アソシエーションの追加

1. user has many likes （ユーザはたくさんのいいねを持っている）<br>
2. like belongs to user　（いいねはユーザに所属している）<br>
3. message has many likes（メッセージはたくさんのいいねを持っている）<br>
4. like belongs to message（いいねはメッセージに所属している）<br>

※ likes tableは中間テーブルになる

+ `api/app/models/user.rb`を編集<br>

```rb:user.rb
# frozen_string_literal: true

class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
  include DeviseTokenAuth::Concerns::User

  has_many :messages
  has_many :likes # 追加

  validates :name, presence: true
  validates :name, length: { maximum: 30 }
end
```

+ `api/app/models/like.rb`を編集<br>

```rb:like.rb
class Like < ApplicationRecord
  belongs_to :user
  belongs_to :message
end
```

+ `api/app/models/message.rb`を編集<br>

```rb:message.rb
class Message < ApplicationRecord
  belongs_to :user
  has_many :likes # 追加

  validates :content, presence: true
end
```

# 4-2 Likesコントローラを作成する(全編)

+ `root $ docker compose run --rm api rails g controller likes`を実行<br>

+ `api/app/controllers/likes_controller.rb`を編集<br>

```rb:likes_controller.rb
class LikesController < ApplicationController
  before_action :authenticate_user!, only: ['create']

  def create
    like = Like.new(message_id: params[:id], user_id: current_user.id)

    if like.save
      render json: { id: like.id, email: current_user.email, message: '成功しました' }, status: 200
    else
      render json: { message: '保存できませんでした', errors: like.errors.messages }, status: 400
    end
  end
end
```

## ルートを設定する

+ `config/routes.rb`を編集<br>

```rb:routes.rb
Rails.application.routes.draw do
  mount_devise_token_auth_for 'User', at: 'auth', controllers: {
    registrations: 'auth/registrations'
  }

  resources :messages, only: ['index'] do
    member do
      resources :likes, only: ['create']
    end
  end
end
```

`likes POST   /messages/:id/likes(.:format) likes#create`<br>

## Postmanでテスト

+ まずログインしている状態にする<br>

+ `POSTMAN(POST) localhost:3000/messages/1/likes`を入力<br>

+ `Body`タブを選択して`form-data`を選択して `access-token`, `client`, `uid`を入れて`Send`する<br>

```:json
{
    "id": 1,
    "email": "takaki55730317@gmail.com",
    "message": "成功しました"
}
```

+ `POSTMAN(POST) localhost:3000/messages/99999/likes`を入力<br>

+ `Body`タブを選択して`form-data`を選択して `access-token`, `client`, `uid`を入れて`Send`する<br>

```:json
{
    "message": "保存できませんでした",
    "errors": {
        "message": [
            "must exist"
        ]
    }
}
```

+ `POSTMAN(POST) localhost:3000/messages/2/likes`を入力<br>

+ `Body`タブを選択して`form-data`を選択して `access-token`, `client`, `uid`を入れて`Send`する<br>

```:json
{
    "id": 2,
    "email": "takaki55730317@gmail.com",
    "message": "成功しました"
}
```

## 4-3 likesコントローラを作成する(中編)

### Destoryメソッドを実装する

`api/app/controllers/likes_controller.rb`を編集<br>

```rb:likes_controller.rb
class LikesController < ApplicationController
  before_action :authenticate_user!, only: ['create', 'destory']

  def create
    like = Like.new(message_id: params[:id], user_id: current_user.id)

    if like.save
      render json: { id: like.id, email: current_user.email, message: '成功しました' }, status: 200
    else
      render json: { message: '保存できませんでした', errors: like.errors.messages }, status: 400
    end
  end

  def destroy
    like = Like.find(params[:id])

    if like.destroy
      render json: { id: like.id, email: like.user.email, message: '削除に成功しました' }, status: 200
    else
      render json: { message: '削除できませんでした', errors: like.errors.messages }, status: 400
    end
  end
end
```

## ルートを設定する

+ `api/config/routes.rb`を編集<br>

```rb:routes.rb

```

```
likes POST   /messages/:id/likes(.:format)  likes#create
messages GET /messages(.:format)            messages#index
like DELETE  /likes/:id(.:format)           likes#destroy
````

## Postmanでテスト
+ まずログイン状態にする<br>

+ `POSTMAN(DELETE) localhost:3000/likes/1`入力する<br>

+ `Body`タブを選択して`form-data`に`access-token`, `client`, `uid`を設定して`Send`する<br>

```:json
{
    "id": 1,
    "email": "takaki55730317@gmail.com",
    "message": "削除に成功しました"
}
```

## 4-4 コントローラを作成する(後編)

### messagesコントローラにいいねの情報を追加する

+ `api/app/controllers/messages_controller.rb`を編集<br>

```rb:messages_controller.rb
class MessagesController < ApplicationController
  before_action :authenticate_user!, only: ["index"]

  def index
    messages = Message.all
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

※ likes: message.likes.map { ... }は、messageに紐づくlikesレコードを取得し、idとemailの名前と値をもつオブジェクトを作成している<br>

## Postmanでテストする

+ ログイン状態にする<br>

+ `POSTMAN(GET) localhost:3000/messages`を入力する<br>

+ `Body`タブを選択して`form-data`に`access-token`と`client`と`uid`を設定して`Send`する<br>

```:json
[
    {
        "id": 1,
        "user_id": 1,
        "name": "takaki",
        "content": "0番目のメッセージです！",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T13:12:13.623Z",
        "likes": [
            {
                "id": 4,
                "email": "takaki55730317@gmail.com"
            },
            {
                "id": 5,
                "email": "takaki55730317@gmail.com"
            }
        ]
    },
    {
        "id": 2,
        "user_id": 1,
        "name": "takaki",
        "content": "1番目のメッセージです！",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T13:12:13.680Z",
        "likes": [
            {
                "id": 2,
                "email": "takaki55730317@gmail.com"
            },
            {
                "id": 3,
                "email": "takaki55730317@gmail.com"
            }
        ]
    },
    {
        "id": 3,
        "user_id": 1,
        "name": "takaki",
        "content": "2番目のメッセージです！",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T13:12:13.710Z",
        "likes": []
    }
]
```

# 5-1 リアルタイムチャット機能を実装する

## Action Cableを設定する

+ `root $ docker compose run --rm api rails g channel room`を実行<br>

+ `api/app/channels/room_channel.rb`を編集<br>

```rb:room_channel.rb
class RoomChannel < ApplicationCable::Channel
  def subscribed
    stream_from 'room_channel'
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end

  def receive(data)
    user = User.find_by(email: data['email'])

    if message = Message.create(content: data['message'], user_id: user.id)
      ActionCable.server.broadcast 'room_channel', { message: data['message'], name: user.name, created_at: message.created_at }
    end
  end
end
```
