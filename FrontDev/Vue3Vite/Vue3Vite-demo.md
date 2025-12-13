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
npm install @element-plus/icons-vue
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