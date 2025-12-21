---
layout: default
title: Vite Demo
parent: VUE 3
grand_parent: Front-end Dev
nav_order: 900
---


This is the steps for Vite to create or init template vue  。
{: .fs-6 .fw-300 }


## Table of contents
{: .no_toc .text-delta }


1. TOC
{:toc}


## 使用 Vite 创建 vue 3 的项目


### 1、环境检查


### 基础环境

 - windows 10 22H2
 - node.js   v22.20.0
 - npm   10.9.3

> 可以使用 nvm 来管理node.js版本，具体可以参考Node章节文章。

### vs code 插件

- Volar  vue 3 语法支持
- vue3-snippets 语法高亮，代码格式化
- WindiCSS IntelliSense

### 2、检查镜像

检查当前npm镜像源

```bash
npm config get registry
```
只要是当下可用镜像即可，国内镜像环境可能会变化。
> https://mirrors.huaweicloud.com/repository/npm/

如果不是就去搜索里找找当下可用的国内镜像源：

```bash
# 华为镜像
npm config set registry=https://mirrors.huaweicloud.com/repository/npm/
# 或者淘宝镜像
npm config set registry https://registry.npmmirror.com
```

## create vue by vite



 [Vite 文档]https://cn.vite.dev/guide/

> Vite 需要 Node.js 版本 20.19+, 22.12+。然而，有些模板需要依赖更高的 Node 版本才能正常运行，当你的包管理器发出警告时，请注意升级你的 Node 版本。

给项目取个名字，我这里是参考B站视频学习的,所以取名为 vue3-vite-element-plus-demo

进去自己工作目录，初始化vue项目：

```bash
# npm v6 之前使用 init, v6 之后推荐使用 create  实际是一样的
npm init vite@<vite version> <project_name> --template vue
npm create vite@<vite version> <project_name> --template vue

# npm 7+ 多 --
npm create vite@<vite version> <project_name> -- --template vue
```

可以先看看有哪些vite可用版本

```text
D:\dev-tools-JetBrains\front-web>npm view create-vite versions
[
  '0.0.0-alpha.0', '0.0.0',        '2.5.0',        '2.5.1',
  '2.5.2',         '2.5.3',        '2.5.4',        '2.6.0',
  '2.6.1',         '2.6.2',        '2.6.3',        '2.6.4',
  '2.6.5',         '2.6.6',        '2.7.0',        '2.7.1',
  '2.7.2',         '2.8.0',        '2.9.0',        '2.9.1',
  '2.9.2',         '2.9.3',        '2.9.4',        '2.9.5',
  '3.0.0',         '3.0.1',        '3.0.2',        '3.1.0',
  '3.2.0',         '3.2.1',        '4.0.0-beta.0', '4.0.0',
  '4.1.0-beta.0',  '4.1.0',        '4.2.0-beta.0', '4.2.0-beta.1',
  '4.2.0',         '4.3.0-beta.0', '4.3.0',        '4.3.1',
  '4.3.2',         '4.4.0',        '4.4.1',        '5.0.0-beta.0',
  '5.0.0-beta.1',  '5.0.0',        '5.1.0',        '5.2.0',
  '5.2.1',         '5.2.2',        '5.2.3',        '5.3.0',
  '5.4.0',         '5.5.0',        '5.5.1',        '5.5.2',
  '5.5.3',         '5.5.4',        '5.5.5',        '6.0.0',
  '6.0.1',         '6.1.0',        '6.1.1',        '6.2.0',
  '6.2.1',         '6.3.0',        '6.3.1',        '6.4.0',
  '6.4.1',         '6.5.0',        '7.0.0',        '7.0.1',
  '7.0.2',         '7.0.3',        '7.1.0',        '7.1.1',
  '7.1.2',         '7.1.3',        '8.0.0-beta.0', '8.0.0',
  '8.0.1',         '8.0.2',        '8.0.3',        '8.1.0',
  '8.2.0'
]
```

### 1、创建项目

```bash
npm create vite@latest vue3-vite-element-plus-demo --template vue

# 指定版本
npm create vite@7.1.3 vue3-vite-element-plus-demo --template vue
```

npm 版本 7+ 需要添加额外的 --：

```bash
# npm 7+，需要添加额外的 --：
npm create vite@latest vue3-vite-element-plus-demo -- --template vue

# 指定版本
npm create vite@7.1.3 vue3-vite-element-plus-demo -- --template vue
```

执行效果
```text
D:\dev-tools-JetBrains\front-web>npm create vite@7.1.3 vue3-vite-element-plus-demo -- --template vue
Need to install the following packages:
create-vite@7.1.3
Ok to proceed? (y) y


> npx
> cva vue3-vite-element-plus-demo --template vue

|
o  Scaffolding project in D:\dev-tools-JetBrains\front-web\vue3-vite-element-plus-demo...
|
—  Done. Now run:

  cd vue3-vite-element-plus-demo
  npm install   
  npm run dev
```


### 2、安装依赖启动

```bash
cd  vue3-vite-element-plus-demo
npm install
npm run dev
```

启动之后的效果

```text
  VITE v7.2.7  ready in 3187 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

> 学习视频的是 Vite v2.9.7 ,实际页面有变化。


目录结构：

```text
vue3-vite-element-plus-demo
|
├─.vscode
│      extensions.json
├─node_modules
├─public
│      vite.svg
└─src
│   │  App.vue
│   │  main.js
│   │  style.css
│   ├─assets
│   │      vue.svg
│   └─components
|         HelloWorld.vue
│  .gitignore
└─index.html
└── package-lock.json
└─ package.json
└─ README.md
└─ tree.txt
└─ vite.config.js
```


### 3、安装 element-plus

[element-plus 官网](https://element-plus.org/zh-CN/guide/design)

npm 安装 element-plus 组件

```bash
npm install element-plus --save
``` 
引入配置到 main.js 中：

```js
// main.js
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'

const app = createApp(App)

app.use(ElementPlus)
app.mount('#app')
```

{ .tip}
> 此处是完整引入,也可以按需引入。
> 
> 按需安装 npm install -D unplugin-vue-components unplugin-auto-import

按需引入时则在  vite.config.js 中配置：
```js
import { defineConfig } from 'vite'
import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

export default defineConfig({
  // ...
  plugins: [
    // ...
    AutoImport({
      resolvers: [ElementPlusResolver()],
    }),
    Components({
      resolvers: [ElementPlusResolver()],
    }),
  ],
})
```


在 app.vue 添加 element plus 的组件元素看看是否生效

```vue
<script setup>

</script>

<template>
  <div>
    <h1>Hello Word</h1>
  </div>
  <div class="button-row">
      <el-button>Default</el-button>
      <el-button type="primary">Primary</el-button>
      <el-button type="success">Success</el-button>
      <el-button type="info">Info</el-button>
      <el-button type="warning">Warning</el-button>
      <el-button type="danger">Danger</el-button>
    </div>
</template>

<style scoped>

</style>
```


### 4、安装 windi css

[官方网站](https://cn.windicss.org/)

[官方文档](https://cn.windicss.org/integrations/vite.html)


安装相关包：

```bash
npm i -D vite-plugin-windicss windicss
```

然后，在你的 Vite 配置 `vite.config.js` 中添加插件：

```js
import WindiCSS from 'vite-plugin-windicss'

export default {
  plugins: [
    WindiCSS(),
  ],
}

```

最后，在你的 Vite 入口文件`main.js`中导入 `virtual:windi.css`：

```js
import 'virtual:windi.css'
```

再安装一下 windi css 的插件 ： WindiCSS IntelliSense


### 5、安装 vue Router

Vue.js 的官方路由。

[使用文档](https://router.vuejs.org/zh/introduction.html)

```bash
npm install vue-router@4
```
在 src 目录下创建 router 文件夹，在 router 文件夹下创建 index.js 文件

```js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [ ]

const router = createRouter({
   history: createMemoryHistory(),
   routes, //  routes: routes 缩写
})

export default router

```

然后挂载到入口文件

```js
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import './style.css'
import App from './App.vue'

// 增加导入
import Router from './router'


const app = createApp(App)

// 增加挂载
app.use(Router)
app.use(ElementPlus)

import 'virtual:windi.css'

app.mount('#app')

```

### 6、 给 src 添加别名

导入 path, 使用 path.resolve 配置别名

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import WindiCSS from 'vite-plugin-windicss'

import path from "path"


// https://vite.dev/config/
export default defineConfig({
  resolve:{
    alias:{
      // 将 ~ 给当前目录的src 取个别名
      "~": path.resolve(__dirname, "src")
    }
  },
  plugins: [vue(), WindiCSS()],
})

```

### 7、 添加后台首页

在 src 下面创建pages/index.vue

```vue
<template>
    <div>
        <h1>后台首页</h1>
    </div>
</template>
```


在路径里增加 首页
```js

import { createRouter, createWebHistory } from 'vue-router'

// 引入后台主页
import Index from "~/pages/index.vue"

// 添加后台主页路由, 这个路由名字随意，但是下方写法就不一样
const  routers = [
   { path: "/", component: Index},
   { path: "/about", component: About},

]

const router = createRouter({
   history: createWebHistory(),
   routes: routers
   //routes  // routers: routers 的缩写,新手这里要注意，这个key一定是 routes
});

export default router
```

在 app.vue 里面引入路由

```vue
<template>
    <router-view></router-view>
</template>
```

可参考类似步骤添加其他页面。

### 8、 添加图标库

```bash
npm install @element-plus/icons-vue --save
```

```js
import { createApp } from 'vue'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import './style.css'
import App from './App.vue'
import Router from './router/router'

// 引入图标库 step (1/2)
import * as ElementPlusIconsVue from '@element-plus/icons-vue'

const app = createApp(App)


// 引入图标库 step (2/2)
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
   app.component(key, component)
}

app.use(Router)
app.use(ElementPlus)

import 'virtual:windi.css'

app.mount('#app')

```
在页面中使用图标

借助 <el-icon> 标签使用

```vue
<template>
  <p>
    with extra class <b>is-loading</b>, your icon is able to rotate 360 deg in 2
    seconds, you can also override this
  </p>
  <el-icon :size="20">
    <Edit />
  </el-icon>
  <el-icon color="#409efc" class="no-inherit">
    <Share />
  </el-icon>
  <el-icon>
    <Delete />
  </el-icon>
  <el-icon class="is-loading">
    <Loading />
  </el-icon>
  <el-button type="primary">
    <el-icon style="vertical-align: middle">
      <Search />
    </el-icon>
    <span style="vertical-align: middle"> Search </span>
  </el-button>
</template>
```


使用SVG图标

```vue
<template>
  <div style="font-size: 20px">
    <!-- 由于SVG图标默认不携带任何属性 -->
    <!-- 你需要直接提供它们 -->
    <Edit style="width: 1em; height: 1em; margin-right: 8px" />
    <Share style="width: 1em; height: 1em; margin-right: 8px" />
    <Delete style="width: 1em; height: 1em; margin-right: 8px" />
    <Search style="width: 1em; height: 1em; margin-right: 8px" />
  </div>
</template>
```

> 只要你安装了 @element-plus/icons-vue，就可以在任意版本里使用 SVG 图标。


文本框添加图标

要在输入框中添加图标，你可以简单地使用 `prefix-icon` 和 `suffix-icon` 属性。 另外， `prefix` 和 `suffix` 命名的插槽也能正常工作。

（1）属性模式
```vue
<template>
   <div class="input-group">
      <span class="label">Using attributes</span>
      <div class="input-container">
         <el-input
                 v-model="input1"
                 class="responsive-input"
                 placeholder="Pick a date"
                 :suffix-icon="Calendar"
         />
         <el-input
                 v-model="input2"
                 class="responsive-input"
                 placeholder="Type something"
                 :prefix-icon="Search"
         />
      </div>
   </div>
</template>
```

（2）插槽模式

```vue
<template>
   <div class="input-group">
      <span class="label">Using slots</span>
      <div class="input-container">
         <el-input
                 v-model="input3"
                 class="responsive-input"
                 placeholder="Pick a date"
         >
            <template #suffix>
               <el-icon class="el-input__icon"><calendar /></el-icon>
            </template>
         </el-input>
         <el-input
                 v-model="input4"
                 class="responsive-input"
                 placeholder="Type something"
         >
            <template #prefix>
               <el-icon class="el-input__icon"><search /></el-icon>
            </template>
         </el-input>
      </div>
   </div>
</template>
```


### 9、安装 axios 进行请求交互

axios 是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js 中。

[官网文档](http://axios-js.com/)

```bash
npm install axios --save
```

vue-axios

```bash
npm install --save axios vue-axios
```

在 src 新建 axios.js 文件

```js
import axios from 'axios'

const instance = axios.create({
    // http://localhost:3000  使用 /api 代理后端接口
    baseURL: '/api',
    timeout: 5000
})

export default instance
```

修改 vite.config.js 增加 server下的 proxy 节点配置代理

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'


import AutoImport from 'unplugin-auto-import/vite'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

import WindiCSS from 'vite-plugin-windicss'
import { viteMockServe } from 'vite-plugin-mock';


import path from "path"


// https://vite.dev/config/
export default defineConfig({
  resolve:{
    alias:{
      // 将 ~ 给当前目录的src 取个别名
      "~": path.resolve(__dirname, "src")
    }
  },
  server:{
    // 允许通过局域网访问
    cors: true,
    // 配置代理解决跨域
    proxy: {
      '/api':{
        target: "http://localhost:8080/",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/,'')
      }
    }
  },
  plugins: [
    vue(), 
    WindiCSS(),
    AutoImport({
      resolvers: [ElementPlusResolver()],
      imports: ['vue','@vueuse/core']
    }),
    Components({
      resolvers: [ElementPlusResolver()],
    }), 
    viteMockServe({
      mockPath: './src/mock', // Mock文件存放目录
      localEnabled: true, // 开发环境启用
      prodEnabled: false, // 生产环境禁用
      watchFiles: true, // 监视文件更改
      logger: true, // 控制台显示请求日志
      supportTs: false // 重要：禁用TS支持
    }),
  ],
})

```

新建一个 src/api/manager.js 文件



```js
import axios from '~/axios'

export function login(username, password) {
   
    return axios.post('/admin/login', {
        username,
        password
    })
}
```

在登录页面 import 函数

```vue
<script setup>

   import { ref, reactive } from 'vue';
   import { login } from '~/api/manager'
   import { ElNotification} from 'element-plus';

   const form = reactive({
      username:"",
      password:""
   })

   const ruleFs = reactive({
      username: [
         {required: true, message:"用户名不能为空", trigger:'blur'},
         {min: 1, max: 10, message:"用户名长度1到10个字符", trigger:'blur'},
      ],
      password: [
         {required: true, message:"密码不能为空", trigger:'blur'},
         {min: 1, message:"密码不能少于6位", trigger:'blur'}
      ]
   });

   const formRef = ref(null);

   const obSubmit = ()=>{

      formRef.value.validate(valid=>{

         if(!valid){
            console.log(' valied failed !')
         }

         console.log('submit!')
         login(form.username, form.password)
                 .then(res=>{
                    console.log('res', res)
                    // 提示成功
                    ElNotification({
                       message: "登录成功",
                       type: 'success',
                       duration: 3000,
                    })
                    // 存储用户信息
                 }).catch(err=>{
                     console.log('err!', err)
                     ElNotification({
                        message: err.message || "请求失败",
                        type: 'error',
                        duration: 3000,
                     })
         })
      });
   }
</script>

```

### 10 mockjs 模拟数据

因为使用 vite-plugin-mock 插件，所以需要 vite-plugin-mock

> 注意 mockjs 和 vite-plugin-mock 是不同的组件

```bash
npm install vite-plugin-mock 
```
>--save-dev代表开发依赖，可简写为 -D

在 src 目录下新建 mock 文件夹，并在其中新建 index.js 文件

```js
// --- 1. 内部通用的帮助函数 (Utility Function) ---

/**
 * 创建一个简化的 Mock 方法配置
 * @param {string} url API 请求路径
 * @param {string} method HTTP 方法 ('get', 'post', 'put', 'delete')
 * @param {any|Function} responseData 模拟返回的数据
 */
function createMockMethod(url, method, responseData) {
  return {
    url,
    method,
    // 如果 responseData 是函数，则直接使用它；否则包装成函数返回数据
    response: typeof responseData === 'function' 
      ? responseData 
      : () => responseData,
  };
}


// --- 2. 模块 A：用户认证模块 (Login Module) ---

// 我们可以定义一个常量数组，专门存放用户相关的 mock
const loginMocks = [
  createMockMethod('/api/admin/login', 'post', ({ body }) => {
    const { username } = body;
    if (username === 'admin') {
      return { code: 200, message: '登录成功', data: { token: 'admin-token' } };
    } else {
      return { code: 500, message: '用户名或密码错误', data: null };
    }
  }),
  
  createMockMethod('/api/user/info', 'get', {
    code: 200, 
    data: { name: 'Admin', avatar: 'https://example.com/avatar.png' }
  }),
];


// --- 3. 模块 B：商品管理模块 (Products Module) ---

// 我们可以定义另一个常量数组，专门存放商品相关的 mock
const productMocks = [
  createMockMethod('/api/products/list', 'get', {
    code: 200,
    data: [
      { id: 1, name: 'Apple Watch', price: 2999 },
      { id: 2, name: 'MacBook Pro', price: 12999 },
    ],
  }),

  createMockMethod('/api/products/add', 'post', {
    code: 0,
    message: 'Product added successfully',
  }),
];


// --- 4. 最终导出：将所有模块的 Mock 数组合并导出一个大数组 ---

// 使用扩展运算符 (...) 合并所有内部定义的 mock 数组
export default [
  loginMocks,
  productMocks,
];
```



### 11、安装 vueuse 管理登录信息

vueuse 是一个为 Vue.js 3 提供的一组基于 Composition API 实用函数集合。 

[vueuse 官网 https://vueuse.org](https://vueuse.org/guide/#installation))

{ .tips}
> From v12.0, VueUse no longer supports Vue 2. Please use v11.x for Vue 2 support.

核心软件包的目标是轻量级且无依赖项。而附加组件则将流行的软件包封装到统一的 API 风格中。

- @vueuse/head  头部,vue3的文档管理器，支持服务器渲染
- @vueuse/core  核心包，包含常用的工具函数
- @vueuse/integrations  集成包，包含常用的第三方库的封装
- @vueuse/motion  动画库
- @vueuse/router  路由库
- @vueuse/sound  声音库
- @vueuse/universal  通用库
- @vueuse/web  网络库
- @vueuse/compat  兼容库
- @vueuse/shared  共享库
- @vueuse/gesture  手势库
- @vueuse/rxjs RxJS 库
- @vueuse/firebase Firebase 实时绑定库

安装cookie 状态管理插件 universal-cookie ：

```bash
npm i  @vueuse/core  --save
npm install @vueuse/integrations  --save

# 管理cookie 的
npm i universal-cookie@^7 --save

# @vueuse/integrations 需要用到的依赖，如果没有需要手动安装一下
npm i change-case@^5  --save
npm i drauu@^0  --save
npm i focus-trap@^7  --save  
npm install fuse.js@^7  --save
npm install idb-keyval@^6  --save
npm install jwt-decode@^4  --save
npm install --save qrcode
npm install --save sortable
```


### 11、安装 vuex 状态管理模式

Vuex 是一个用于 Vue.js 应用程序的状态管理模式和库 。它为应用程序中的所有组件提供一个集中式的状态存储，并通过规则确保状态只能以可预测的方式进行修改。

[官网 https://vuex.vuejs.org](https://vuex.vuejs.org/installation.html)

```bash
npm install vuex@next --save
```

官方demo ：

```js
import { createApp } from 'vue'
import { createStore } from 'vuex'

// Create a new store instance.
const store = createStore({
  state () {
    return {
      count: 0
    }
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

const app = createApp({ /* your root component */ })

// Install the store instance as a plugin
app.use(store)

```
新建 src/store/index.js

```js
import { createStore } from 'vuex'

const store = createStore({
   state () {
      return {
         user: {}
      }
   },
   mutations: {
      setuserinfo (state, user) {
         state.user = user;
      }
   }
})

export default store ;
```

在 main.js 中使用

```js

import store from './store';

// 其他省略

app.use(store);

```

整合登录、退出、获取用户信息后的

```js
import { createStore } from 'vuex';
import { login, getinfo, logout } from '../api/manager';
import { setToken, removeToken } from '../utils/auth';

const store = createStore({
   state () {
      return {
         user: {}
      }
   },
   mutations: {
      SET_USERINFO(state, user) {
         state.user = user;
      }
   },
   actions: {
        // 登录actions
        login({commit},{username, password}){
            return new Promise((resolve, reject)=>{
            login(username, password).then(res=>{
                setToken(res.token);
                resolve(res)
            }).catch(err=>reject(err));
        })
        },
        //登录成功后获取当前用户的登录信息
        getinfo({commit}){
            return new Promise((resolve, reject)=>{
                getinfo().then(res=>{
                    commit("SET_USERINFO",res);
                    resolve(res);
                }).catch(err=>reject(err));
            });
        },
        // 登出 actions
        logout({commit}){
            // 移除token
            removeToken();
            // 清除当前用户状态
            commit("SET_USERINFO",{});
        }
        
    },
});

export default store ;
```

### 12、全局路由守卫

在 src/permission.js 文件

```js
import router from "./router";

// 全局守卫
router.beforeEach((to, from, next)=>{
    
    next();
});
```

整合了 token 获取和用户信心管理之后的最终效果

```js
import router from "./router"; 
import {getToken} from "~/utils/auth";
import { notice } from '~/utils/notice';
import store from "./store";

// 全局路由守卫
router.beforeEach(async (to, from, next)=>{

    const token = getToken();

    // 找不到 token 且不是去登录
    if(!token && to.path !='/login'){
        notice("请先登录", "error");
        // 没有登录强制回到登录页面
        return next({ path: "/login"});
    }

    if(token && to.path == "/login"){
        notice("请勿重复登录", "error");
        // 没有登录强制回到登录页面
        return next({ path: "/"});
    }

    // 如果用户登录了，自动获取用户信息，存储到 vuex 中
    if(token){
        await store.dispatch("getinfo");
    }
    next();
});
```

在 src/main.js 导入引用

```js
import  '~/permission';
```


### 13 进度条 nprogress

```bash
mpn install --save nprogress
```

```js
import 'nprogress/nprogress.css'
```

在 通知组件中增加管理进度的方法

```js
import nProgress from "nprogress";

// 开启 loading
export function showLoading(){
    nProgress.start();
}

// 关闭 loading
export function hideLoading(){
    nProgress.done();
}
```



### 14 自定义组件

新建组件 src/component/FormDrawer.vue

```vue
<template>
<el-drawer v-model="showDrawer"  :title="title" :size="size" :close-on-click-modal="destroyOnClose">
    <div class="formDrawer">
        <div class="body">
            <slot></slot>
        </div>
        <div class="actions">
            <el-button  color="#626aef" class="w-[50px]" type="primary"
                @click="doSubmit" :loading="loading">{{ confirmText }}</el-button>
            <el-button class="w-[50px]" type="info"
                @click="close" >取消</el-button>
        </div>
    </div>
     
</el-drawer>
</template>  
<script setup>
    import {ref} from 'vue';
    
    // 定义抽屉的打开 关闭
    const showDrawer = ref(false);

    // 对外暴露属性
    const props = defineProps({
        title: String,
        size: {
            type:String,
            default:"40%"
        },
        destroyOnClose:{
            type: Boolean,
            default: false,
        },
        confirmText:{
            type:String,
            default:'确认'
        }
    });

    // open  打开抽屉
    const open = () => showDrawer.value = true ;
    // close 关闭抽屉
    const close = ()=> showDrawer.value = false ;

    // 按钮点击后的loading状态
    const loading = ref(false);
    const loadingShow = ()=> loading.value = true ;
    const loadingHide = ()=> loading.value = false ;

    // 使用编译器宏，暴露自己的属性给父级组件
    defineExpose({
        open,
        close,
        loadingShow,
        loadingHide
    });

    
    // 使用编译器宏, 传递按钮事件
    const emit = defineEmits(["submit"]);
    const doSubmit = ()=> emit("submit");

</script> 
<style>
    .formDrawer{
        height: 100%;
        width: 100%;
        position: relative;
        @apply flex flex-col;
    } 
    .formDrawer .body{
        flex: 1;
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 50px;
        overflow-y: auto;
    }
    .formDrawer .actions{
        height: 50px;
        @apply mt-auto flex ;
    }
</style>
```

在修改密码的页面使用组件

```vue
<script setup>
import { Aim, FullScreen, Unlock } from '@element-plus/icons-vue';
import { logout } from '~/api/manager';
import { notice, showConfirm} from '~/utils/notice';
import { useRouter } from 'vue-router';
import { useStore } from 'vuex';
import {useFullscreen } from '@vueuse/core';

import { ref, reactive } from 'vue';
import { updatePassword } from '~/api/manager';
import FormDrawer from '~/components/FormDrawer.vue';

const showDrawer = ref(false);
 const formDrawerRef = ref(null);

const {isFullscreen, // 全屏状态
     toggle // 切换全屏
      } = useFullscreen();
const store = useStore();
const router = useRouter();

const handleCommand = (c)=>{
    switch (c){
        case "logout":
            handleLogout();
            break;
        case "rePassword":
            //showDrawer.value = true ;
            // 调用 组件节点的 open方法
            formDrawerRef.value.open();
            break;
        case "profile":
            console.log('查看信息');
            break;
    }
}

// 刷新
const handleRefresh = ()=>{
    location.reload();
}
// 修改密码  

    const form = reactive({
        oldpassword:"",
        password:"",
        repassword:""
    })

    const ruleFs = reactive({
        oldpassword: [
            {required: true, message:"旧密码不能为空", trigger:'blur'}
        ],
        password: [
            {required: true, message:"新密码不能为空", trigger:'blur'},
            {min: 6, message:"新密码不能少于6位", trigger:'blur'}
        ],
        repassword: [
            {required: true, message:"密码不能为空", trigger:'blur'},
            {min: 6, message:"新密码二次验证失败", trigger:'blur'}
        ]
    });

   
    const formRef = ref(null);
    // loading 等待
    const loading = ref(false);
    const onSubmit = ()=>{

        formRef.value.validate(valid=>{

            if(!valid){
                console.log(' valied failed !')
            }
            formDrawerRef.value.loadingShow()
            updatePassword(form).then(res=>{
                console.log(res)
                if(res.code ==200){    
                    notice("修改密码成功，请重新登录");
                    store.dispatch("/logout");
                    router.push("/login");
                }else{
                    notice( res.message||"修改密码失败", 'error');
                }
            }).finally(()=>{
                formDrawerRef.value.loadingHide();
            })
        });
    }

    const obCancel = ()=>{
        showDrawer.value = false ;
    }
</script>

<template>
<div class="s-header">
    <span class="s-logo">
        <el-icon class="mf-1 mr-1"><eleme-filled/></el-icon>
        个人学习网站
    </span>
    <el-tooltip content="收起" effect="dark">
        <el-icon class="icon-btn"><fold/></el-icon>
    </el-tooltip>
    <el-tooltip content="刷新" placement="bottom" effect="dark">
        <el-icon class="icon-btn" @click="handleRefresh"><refresh/></el-icon>
    </el-tooltip>
    <div class="s-header-right">
        <el-tooltip content="全屏" placement="bottom" effect="dark">
            <el-icon class="icon-btn" @click="toggle">
                <FullScreen v-if="!isFullscreen"/><Aim v-else/>
            </el-icon>
        </el-tooltip>
        <el-dropdown @command="handleCommand">
            <span class="flex items-center text-light-50">
            <el-avatar class="mr-2" :size="25" :src="$store.state.user.avatar"></el-avatar>
            {{ $store.state.user.username }}
            <!-- <el-avatar :size="25" :src="$store.user.avatar"></el-avatar> -->
            <el-icon class="el-icon--right">
                <arrow-down />
            </el-icon>
            </span>
            <template #dropdown>
            <el-dropdown-menu>
                <el-dropdown-item command="rePassword" >修改密码</el-dropdown-item>
                <el-dropdown-item command="logout">退出登录</el-dropdown-item>
                <el-dropdown-item command="profile">个人信息</el-dropdown-item>
            </el-dropdown-menu>
            </template>
        </el-dropdown>
    </div>
</div>
   
<!-- 原始 抽屉使用 -->   
<!--
<el-drawer v-model="showDrawer" header-class="#626aef" title="修改密码" size="35%" :close-on-click-modal="false">
    <el-form ref="formRef" :rules="ruleFs" :model="form"  class="w-[300px]" size="small">
            <el-form-item prop="oldpassword" label="当前密码">
                <el-input v-model="form.oldpassword" placeholder="请输入旧密码" show-password>
                     <template #prefix>
                        <el-icon><Unlock/></el-icon>
                    </template>
                </el-input>
            </el-form-item>
            <el-form-item prop="password" label="新的密码" >
                <el-input v-model="form.password" placeholder="请输入新密码" show-password>
                    <template #prefix>
                        <el-icon><Lock /></el-icon>
                    </template>
                </el-input>
            </el-form-item>
            <el-form-item prop="repassword" label="确认密码" >
                <el-input v-model="form.repassword" placeholder="请再次输入新密码" show-password>
                    <template #prefix>
                        <el-icon><Lock /></el-icon>
                    </template>
                </el-input>
            </el-form-item>
            <el-form-item>
                <el-button  color="#626aef" class="w-[50px]" type="primary"
                @click="obSubmit" :loading="loading">确认</el-button>

                 <el-button color="#626aef" class="w-[50px]" type="success"
                @click="obCancel" >取消</el-button>
            </el-form-item>
        </el-form>  
</el-drawer>
-->

<!-- 使用自定义组件 打开抽屉-->
<form-drawer ref="formDrawerRef" title="修改密码" @submit="onSubmit">
    <el-form ref="formRef" :rules="ruleFs" :model="form"  class="w-[300px]" style="height: 1000px;" size="small">
            <el-form-item prop="oldpassword" label="当前密码">
                <el-input v-model="form.oldpassword" placeholder="请输入旧密码" show-password>
                     <template #prefix>
                        <el-icon><Unlock/></el-icon>
                    </template>
                </el-input>
            </el-form-item>
            <el-form-item prop="password" label="新的密码" >
                <el-input v-model="form.password" placeholder="请输入新密码" show-password>
                    <template #prefix>
                        <el-icon><Lock /></el-icon>
                    </template>
                </el-input>
            </el-form-item>
            <el-form-item prop="repassword" label="确认密码" >
                <el-input v-model="form.repassword" placeholder="请再次输入新密码" show-password>
                    <template #prefix>
                        <el-icon><Lock /></el-icon>
                    </template>
                </el-input>
            </el-form-item>
        </el-form>  
</form-drawer>

</template>
<style scoped>
    .s-header{
        @apply flex items-center bg-indigo-500 text-light-100 fixed top-0 left-0 right-0;
        height: 64px;
    }
    .s-logo{
       width: 250px;   
       @apply flex justify-center items-center font-thin;  
    }
    .icon-btn{
        @apply flex justify-center items-center ;
        width: 42px;
        height: 42px;
        cursor: pointer;
    }
    .s-header .s-header-right{
       @apply ml-auto flex items-center ;
    }
    .s-header .dropdown{
        height: 42px;
    }
</style>
```

动画

```
npm i gsap --save
```

echarts

```bash
npm install -s echarts
```

### 100、 sm-crypto 加密

```bash
npm install --save sm-crypto
```

```js
//使用
import {sm2,sm3,sm4} from 'sm-crypto'

// ----------- sm2 --------------------
//获取密钥对
let keypair = sm2.generateKeyPairHex()

publicKey = keypair.publicKey // 公钥
privateKey = keypair.privateKey // 私钥

// 默认生成公钥 130 位太长，可以压缩公钥到 66 位
const compressedPublicKey = sm2.compressPublicKeyHex(publicKey) // compressedPublicKey 和 publicKey 等价
sm2.comparePublicKeyHex(publicKey, compressedPublicKey) // 判断公钥是否等价

//加解密
const cipherMode = 1 // 1 - C1C3C2，0 - C1C2C3，默认为1

let encryptData = sm2.doEncrypt(msgString, publicKey, cipherMode) // 加密结果
let decryptData = sm2.doDecrypt(encryptData, privateKey, cipherMode) // 解密结果

encryptData = sm2.doEncrypt(msgArray, publicKey, cipherMode) // 加密结果，输入数组
decryptData = sm2.doDecrypt(encryptData, privateKey, cipherMode, {output: 'array'}) // 解密结果，输出数组


// ----------- sm3 --------------------
let hashData = sm3('abc') // 杂凑

// hmac
hashData = sm3('abc', {
    key: 'daac25c1512fe50f79b0e4526b93f5c0e1460cef40b6dd44af13caec62e8c60e0d885f3c6d6fb51e530889e6fd4ac743a6d332e68a0f2a3923f42585dceb93e9', // 要求为 16 进制串或字节数组
})

// ----------- sm4 --------------------
const msg = 'hello world! 我是 juneandgreen.' // 可以为 utf8 串或字节数组
const key = '0123456789abcdeffedcba9876543210' // 可以为 16 进制串或字节数组，要求为 128 比特
 // 加密，默认输出 16 进制字符串，默认使用 pkcs#7 填充（传 pkcs#5 也会走 pkcs#7 填充）
let encryptData = sm4.encrypt(msg, key)

// 解密
let decryptData = sm4.decrypt(encryptData, key) // 默认使用 pkcs#7 填充（传 pkcs#5 也会走 pkcs#7 填充）
```