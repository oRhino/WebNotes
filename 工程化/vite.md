## Vite

### vite 配置别名

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
