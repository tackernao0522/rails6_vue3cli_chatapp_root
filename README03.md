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
