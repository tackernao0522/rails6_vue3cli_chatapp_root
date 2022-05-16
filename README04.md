# 6-1 Vue.jsプロジェクトの作成

+ `下記の指定するファイルがあれば削除`<br>

1. src/components/HelloWorld.vue <br>
2. src/views/About.vue <br>
3. src/views/Home.vue <br>
4. src/assets/logo.png <br>

+ `front $ mkdir src/router && touch $_/index.js`を実行<br>

+ `front/src/App.vue`を編集<br>

```vue:App.vue
<template>
  <router-view />
</template>

<script>
export default {};
</script>

<style scoped>
</style>
```

+ `front/src/router/index.js`を編集<br>

```js:index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = []

const router = createRouter({
  history: createWebHistory(process.env.Base_URL),
  routes
})

export default router
```

+ エラーが出た場合は `root $ docker compose run --rm font npm install --save vue-router`を実行<br>

# 6-3 Vue.jsの基本的な使い方を学ぶ

## ウェルカムページを作成する

+ `front $ mkdir src/views && touch $_/WelcomePage.vue`を実行<br>

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <div>こんにちは</div>
</template>

<script>
export default {

}
</script>

<style>

</style>
```

+ `front/src/router/index.js`を編集<br>

```js:index.js
import { createRouter, createWebHistory } from 'vue-router'
import WelcomePage from '../views/WelcomePage'

const routes = [
  {
    path: '/',
    name: 'Welcome',
    component: WelcomePage
  }
]

const router = createRouter({
  history: createWebHistory(process.env.Base_URL),
  routes
})

export default router
```

+ `root $ docker compose restart front`を実行<br>

+ localhost:8080 にアクセスして表示されればOK<br>

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
</template>

<script>
export default {
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
    };
  },
};
</script>

<style>
</style>
```

+ `front/src/WelcomePage.vue`を再編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  <p>{{ subtitle }}</p>
</template>

<script>
export default {
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
    };
  },
};
</script>

<style>
</style>
```

## v-if ディレクティブを使ってみる

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  <p v-if="false">{{ subtitle }}</p>
</template>

<script>
export default {
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
    };
  },
};
</script>

<style>
</style>
```

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  <p v-if="isEnablesd">{{ subtitle }}</p>
</template>

<script>
export default {
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
      isEnablesd: true
    };
  },
};
</script>

<style>
</style>
```

## クリックイベントとメソッドオプションを使ってみる

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  <p v-if="isEnablesd">{{ subtitle }}</p>
  <button v-on:click="toggle">トグルする</button>
</template>

<script>
export default {
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
      isEnablesd: true,
    };
  },
  methods: {
    toggle() {
      this.isEnablesd = !this.isEnablesd
    }
  }
};
</script>

<style>
</style>
```

## dblclick イベントを使ってみる(ダブルクリックで変わる)

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  <p v-if="isEnablesd">{{ subtitle }}</p>
  <button @dblclick="toggle">トグルする</button>
</template>

<script>
export default {
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
      isEnablesd: true,
    };
  },
  methods: {
    toggle() {
      this.isEnablesd = !this.isEnablesd
    }
  }
};
</script>

<style>
</style>
```

## computed プロパティを使ってみる

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  <p v-if="isEnabled">{{ subtitle }}</p>
  <button @dblclick="toggle">トグルする</button>
  <p>{{ text }}</p>
</template>

<script>
export default {
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
      isEnabled: true,
    };
  },
  methods: {
    toggle() {
      this.isEnabled = !this.isEnabled;
    },
  },
  computed: {
    text() {
      if (this.isEnabled) {
        return "こんにちは!";
      } else {
        return "さよなら!";
      }
    },
  },
};
</script>

<style>
</style>
```

# 6-4 Vue.jsの基本的な使い方を学ぼう (後編)

+ `front $ mkdir src/compontns && touch $_/TestComponent.vue`を実行<br>

+ `src/components/TestComponent.vue`を編集<br>

```vue:TestComponent.vue
<template>
  <div>
    これはコンポーネントから表示されています。
  </div>
</template>

<script>
export default {

}
</script>

<style>

</style>
```

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  <test-component />
</template>

<script>
import TestComponent from '../compontns/TestComponent.vue';
export default {
  components: { TestComponent },
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
      isEnabled: true,
    };
  },
  methods: {
    toggle() {
      this.isEnabled = !this.isEnabled;
    },
  },
  computed: {
    text() {
      if (this.isEnabled) {
        return "こんにちは!";
      } else {
        return "さよなら!";
      }
    },
  },
};
</script>

<style>
</style>
```

## コンポーネントにデータを渡す

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  <test-component message="コンポーネントにデータが渡されています" />
</template>

<script>
import TestComponent from '../compontns/TestComponent.vue';
export default {
  components: { TestComponent },
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
      isEnabled: true,
    };
  },
  methods: {
    toggle() {
      this.isEnabled = !this.isEnabled;
    },
  },
  computed: {
    text() {
      if (this.isEnabled) {
        return "こんにちは!";
      } else {
        return "さよなら!";
      }
    },
  },
};
</script>

<style>
</style>
```

+ `front/src/components/TestComponents/TestComponent.vue`を編集<br>

```vue:TestComponent.vue
<template>
  <div>
    これはコンポーネントから表示されています。
    # 追記
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  props: ["message"],
};
</script>

<style>
</style>
```

## 値が変わるデータを子コンポーネントに渡す

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  # 編集
  <test-component :isEnabled="isEnabled" message="コンポーネントにデータが渡されています。" />
</template>

<script>
import TestComponent from '../compontns/TestComponent.vue';
export default {
  components: { TestComponent },
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
      isEnabled: true,
    };
  },
  methods: {
    toggle() {
      this.isEnabled = !this.isEnabled;
    },
  },
  computed: {
    text() {
      if (this.isEnabled) {
        return "こんにちは!";
      } else {
        return "さよなら!";
      }
    },
  },
};
</script>

<style>
</style>
```

+ `front/src/components/TestComponent.vue`を編集<br>

```vue:TestComponent.vue
<template>
  <div>
    これはコンポーネントから表示されています。
    <p>{{ message }}</p>
    # 追加
    <p>{{ isEnabled }}</p>
  </div>
</template>

<script>
export default {
  # 編集
  props: ["message", "isEnabled"],
};
</script>

<style>
</style>
```

## 子コンポーネントから親コンポーネントのメソッドを実行する

+ `front/src/components/TestComponent.vue`を編集<br>

```vue:TestComponent.vue
<template>
  <div>
    これはコンポーネントから表示されています。
    <p>{{ message }}</p>
    # 編集
    <button @click="onClick">親メソッドを実行するボタン</button>
  </div>
</template>

<script>
export default {
  props: ["message"],
  methods: {
    # 追加
    onClick() {
      this.$emit("toggle");
    },
  },
};
</script>

<style>
</style>
```

+ `front/src/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  # 編集
  <test-component @toggle="toggle" message="コンポーネントにデータが渡されています。" />
  # 追加
  <p v-if="isEnabled">こんにちは</p>
</template>

<script>
import TestComponent from '../compontns/TestComponent.vue';
export default {
  components: { TestComponent },
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
      isEnabled: true,
    };
  },
  methods: {
    toggle() {
      this.isEnabled = !this.isEnabled;
    },
  },
  computed: {
    text() {
      if (this.isEnabled) {
        return "こんにちは!";
      } else {
        return "さよなら!";
      }
    },
  },
};
</script>

<style>
</style>
```

## スタイルを適用する

+ `front/views/WelcomePage.vue`を編集<br>

```vue:WelcomePage.vue
<template>
  <h1>{{ title }}</h1>
  <test-component @toggle="toggle" message="コンポーネントにデータが渡されています。" />
  <p v-if="isEnabled">こんにちは</p>
</template>

<script>
import TestComponent from '../compontns/TestComponent.vue';
export default {
  components: { TestComponent },
  data() {
    return {
      title: "初めてのVue.jsアプリです!",
      subtitle: "ようこそ",
      isEnabled: true,
    };
  },
  methods: {
    toggle() {
      this.isEnabled = !this.isEnabled;
    },
  },
  computed: {
    text() {
      if (this.isEnabled) {
        return "こんにちは!";
      } else {
        return "さよなら!";
      }
    },
  },
};
</script>

# 編集
<style scoped>
  p {
    font-size: 20px;
    color: red;
  }
</style>
```