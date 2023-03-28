## Vite

### vite 配置别名

**https://cn.vitejs.dev/config/shared-options.html#resolve-alias**

```js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import path, { join } from 'path';

// https://vitejs.dev/config/
export default defineConfig({
  //....
  // 软链接
  resolve: {
    alias: {
      '@': join(__dirname, '/src'),
    },
  },
});
```

### vite 代理服务器

**https://cn.vitejs.dev/config/server-options.html#server-proxy**

```js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

// https://vitejs.dev/config/
export default defineConfig({
  //...
  // 代理
  server: {
    proxy: {
      // 代理所有 /api 的请求，该求情将被代理到 target 中
      '/api': {
        // 代理请求之后的请求地址
        target: 'https://api.test.com/',
        // 跨域
        changeOrigin: true,
      },
    },
  },
});
```

### vite 环境变量

Vite 使用 dotenv 从你的 环境目录 中的下列文件加载额外的环境变量

```js
.env                # 所有情况下都会加载
.env.local          # 所有情况下都会加载，但会被 git 忽略
.env.[mode]         # 只在指定模式下加载
.env.[mode].local   # 只在指定模式下加载，但会被 git 忽略

```

- 只有以 VITE\_ 为前缀的变量才会暴露给经过 vite 处理的代码

```js
VITE_SOME_KEY = 123;
console.log(import.meta.env.VITE_SOME_KEY); // 123
```

### vite 中使用 svg

插件: vite-plugin-svg-icons

1. 安装

```shell
yarn add vite-plugin-svg-icons -D
##  or
npm i vite-plugin-svg-icons -D
```

2. 配置
   在 vite.config.js 中配置插件

```js
import viteSvgIcons from 'vite-plugin-svg-icons';
import path from 'path';

export default () => {
  return {
    plugins: [
      //....
      viteSvgIcons({
        // 指定需要缓存的图标文件夹
        iconDirs: [path.resolve(process.cwd(), 'src/assets/icons')],
        // 指定symbolId格式
        symbolId: 'icon-[name]',
      }),
    ],
  };
};
```

在 src/main.ts 内引入注册脚本

```js
import 'virtual:svg-icons-register';
```

3. 使用

```js
<template>
  <svg aria-hidden="true">
    <use :class="fillClass" :xlink:href="symbolId" :fill="color" />
  </svg>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  // 显示的 svg 图标名称（剔除 icon-）
  name: {
    type: String,
    required: true,
  },
  // 直接指定 svg 图标的颜色
  color: {
    type: String,
  },
  // 通过 tailwind 指定 svg 颜色的类名
  fillClass: {
    type: String,
  },
});
// 真实显示的 svg 图标名（拼接 #icon-）
const symbolId = computed(() => `#icon-${props.name}`);
</script>
```

### vite 自动注册组件

- vite 的 Glob 导入功能：可帮助我们在文件系统中导入多个模块

- vue 的 defineAsyncComponent 方法：可以创建一个按需加载的异步组件

```js
import { defineAsyncComponent } from 'vue';

export default {
  install(app) {
    // 获取当前路径任意文件夹下的 index.vue 文件
    const components = import.meta.glob('./*/index.vue');
    // 遍历获取到的组件模块
    for (const [key, value] of Object.entries(components)) {
      // 拼接组件注册的 name
      const componentName = 'm-' + key.replace('./', '').split('/')[0];
      // 通过 defineAsyncComponent 异步导入指定路径下的组件
      app.component(componentName, defineAsyncComponent(value));
    }
  },
};
```
