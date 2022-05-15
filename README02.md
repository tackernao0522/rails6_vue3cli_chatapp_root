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
