# 2-3 Postmanを使って機能を確認する

+ `root $ docker compose up api`を実行<br>

## サインアップを試す

+ `POSTMAN(POST) localhost:3000/auth`を入力<br>

+ `Body`を選択<br>

+ `form-data`を選択<br>

+ `KEY`にそれぞれ `name`, `emai`, `password`, `password_confirmation`と入力<br>

+ `VALUE`にそれぞれを入力<br>

+ `Send`をクリック<br>

```:json
{
    "status": "success",
    "data": {
        "id": 1,
        "provider": "email",
        "uid": "takaki55730317@gmail.com",
        "name": "takaki",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T10:38:27.821Z",
        "updated_at": "2022-05-15T10:38:27.983Z"
    }
}
```

## ログインを試す

+ `POSTMAN(POST) localhost:3000/auth/sign_in`を入力<br>

+ `Body`タブを選択する<br>

+ `form-data`を選択する<br>

+ `KEY`にそれぞれ `email`, `password`を入力<br>

+ `VALUE`にそれぞれ入力して`Send`する<br>

```:json
{
    "data": {
        "id": 1,
        "email": "takaki55730317@gmail.com",
        "provider": "email",
        "uid": "takaki55730317@gmail.com",
        "name": "takaki"
    }
}
```

## ログアウトを試す

### ログアウト失敗パターン

+ `POSTMAN(DELETE) localhost:3000/auth/sign_out`を入力<br>

+ `form-data`タブを選択<br>

+ `KEY`に`accsess-token`と`client`と`uid`を入力<br>

+ `VALUE`のそれぞれのログインした時の値を入力して`uid`のみ空にしてみて`Send`<br>

```:json
{
    "success": false,
    "errors": [
        "User was not found or was not logged in."
    ]
}
```

### ログアウト成功パターン

`uid`にVALUEを入れて`Send`する<br>

```:json
{
    "success": true
}
```

# 3-1 テーブルとモデルを作成

+ `root $ docker compose run --rm api rails g model message`を実行<br>

+ `api/db/migrate/create_messages.rb`を編集<br>

```rb:create_messages.rb
class CreateMessages < ActiveRecord::Migration[6.0]
  def change
    create_table :messages do |t|
      t.references :user # アソシエーション user_idカラムが作成される
      t.string :content
      t.timestamps
    end
  end
end
```

+ `root $ docker compose run --rm api rails db:migrate`を実行<br>

+ `root $ docker compose run --rm api rails db:migrate:status`を実行して確認してみる<br>

```:terminal
database: app_development

 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20220515092932  Devise token auth create users
   up     20220515113850  Create messages
```

# 3-2 モデルの設定をする

+ `api/app/models/message.rb`を編集<br>

```rb:message.rb
class Message < ApplicationRecord
  belongs_to :user
end
```

+ `api/app/models/user.rb`を編集<br>

```rb:user.rb
# frozen_string_literal: true

class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
  include DeviseTokenAuth::Concerns::User

  has_many :messages # 追加
end
```

## バリデーションの設定

+ `api/app/models/message.rb`を編集<br>

```rb:message.rb
class Message < ApplicationRecord
  belongs_to :user

  validates :content, presence: true
end
```

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

  # 追加
  validates :name, presence: true
  validates :name, length: { maximum: 30 }
end
```

## Postmanでバリデーションのテスト

+ `POSTMAN(POST) http://localhost:3000/auth`を入力<br>

+ `Body`タブを選択<br>

+ `form-data`を選択<br>

+ `KEY`に `name`, `email`, `password`, `password_confirmation`を入力<br>

+ `VALUE`に 30文字以上のnameを入力、 emailを新規で入力、 passwordを入力、password_confirmationを入力<br>

```:json
{
    "status": "error",
    "data": {
        "id": null,
        "provider": "email",
        "uid": "",
        "name": "testtesttesttesttesttesttesttesttesttesttesttest",
        "email": "cheap_trick_magic@yahoo.co.jp",
        "created_at": null,
        "updated_at": null
    },
    "errors": {
        "name": [
            "is too long (maximum is 30 characters)"
        ],
        "full_messages": [
            "Name is too long (maximum is 30 characters)"
        ]
    }
}
```

+ `name`を30文字以内で登録すると<br>

```:json
{
    "status": "success",
    "data": {
        "id": 2,
        "provider": "email",
        "uid": "cheap_trick_magic@yahoo.co.jp",
        "name": "naomi",
        "email": "cheap_trick_magic@yahoo.co.jp",
        "created_at": "2022-05-15T12:06:24.722Z",
        "updated_at": "2022-05-15T12:06:24.883Z"
    }
}
```

## 3-3 コントローラの作成

+ `root $ docker compsose run --rm api rails g controller messages`を実行<br>

+ `api/app/controllers/messages_controller.rb`を編集<br>

```rb:messages_controller.rb
class MessagesController < ApplicationController
  def index
    messages = Message.all
    messages_array = messages.map do |message|
      id: message.id,
      user_id: message.user.name,
      content: message.content,
      email: message.user.email,
      created_at: message.created_at
    end

    render json: messages_array, status: 200
  end
end
```

下記のようなオブジェクトの配列が作成される<br>

```:json
[
  {
    "id": 1,
    "user_id": 1,
    "name": "Momoko",
    "content": "こんにちは!",
    "email": "momoko@test.com",
    "created_at": "2021-05-12 21:59:59"
  },
  {
    "id": 2,
    "user_id": 2,
    "name": "Hanako",
    "content": "ヤッホー!",
    "email": "hanako@test.com",
    "created_at": "2021-06-30 21:59:59"
  }
]
```

+ `api/config/routes.rb`<br>

```rb:routes.rb
class MessagesController < ApplicationController
  def index
    messages = Message.all
    messages_array = messages.map do |message|
      {
        id: message.id,
        user_id: message.user.id,
        name: message.user.name,
        content: message.content,
        email: message.user.email,
        created_at: message.created_at
      }
    end

    render json: messages_array, status: 200
  end
end
```

## Postmanで確認

+ `POSTMAN(GET) localhost:3000//messages`を入力して`Send`<br>

```:json
[]
```

## Seedで初期データを作る

+ `api/db/seeds.rb`を編集<br>

```rb:seeds.rb
3.times do |number|
  Message.create(content: "#{number}番目のメッセージです！", user_id: User.first.id)
  puts "#{number}番目のメッセージを作成しました"
end

puts "メッセージの作成が完了しました"
```

+ `root docker compose run --rm api rails db:seed`を実行<br>

+ `POSTMAN(GET) localhost:3000//messages`を入力して`Send`<br>

```:json
[
    {
        "id": 1,
        "user_id": 1,
        "name": "takaki",
        "content": "0番目のメッセージです！",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T13:12:13.623Z"
    },
    {
        "id": 2,
        "user_id": 1,
        "name": "takaki",
        "content": "1番目のメッセージです！",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T13:12:13.680Z"
    },
    {
        "id": 3,
        "user_id": 1,
        "name": "takaki",
        "content": "2番目のメッセージです！",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T13:12:13.710Z"
    }
]
```

# 3-4 認証を設定する

+ `api/app/controllers/messages_controller.rb`を編集<br>

```rb:messages_controller.rb
class MessagesController < ApplicationController
  # indexメソッドのみauthenticate_user!を実行する 認証失敗時やエラー時はindexメソッドは実行されない
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
        created_at: message.created_at
      }
    end

    render json: messages_array, status: 200
  end
end
```

## Postmanで確認

+ `POSTMAN(GET) localhost:3000/messages`を未ログイン状態で`Send`してみる<br>

```:json
{
    "errors": [
        "You need to sign in or sign up before continuing."
    ]
}
```

## 認証情報を取得するためにログインをする

+ `Postman(POST) localhost:3000/auth/sign_in`を入力<br>

+ `Body`を選択<br>

+ `form-data`に`email`と`password`を入れて`Send`<br>

```:json
{
    "data": {
        "id": 1,
        "email": "takaki55730317@gmail.com",
        "provider": "email",
        "name": "takaki",
        "uid": "takaki55730317@gmail.com"
    }
}
```

+ 下部の`access-token`, `client`, `uid`の値を使う<br>

+ `Postman(GET) localhost:3000/messages`を入力<br>

+ `Body`タブを選択し、`form-data`にそれぞれの`Key`と`Value`を入力して`Send`してみる<br>

```:json
[
    {
        "id": 1,
        "user_id": 1,
        "name": "takaki",
        "content": "0番目のメッセージです！",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T13:12:13.623Z"
    },
    {
        "id": 2,
        "user_id": 1,
        "name": "takaki",
        "content": "1番目のメッセージです！",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T13:12:13.680Z"
    },
    {
        "id": 3,
        "user_id": 1,
        "name": "takaki",
        "content": "2番目のメッセージです！",
        "email": "takaki55730317@gmail.com",
        "created_at": "2022-05-15T13:12:13.710Z"
    }
]
```
