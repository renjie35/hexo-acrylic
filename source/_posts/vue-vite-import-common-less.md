---
title: '导入公用less文件 | Vue、Vite如何导入公用Less文件？'
date: '2021-09-23'
categories: '技术分享'
keywords: 'vue-vite-import-common-less'
description: '学习如何在Vue和Vite项目中导入公用Less文件，避免重复代码和样式，提高开发效率。逐步指导如何配置和使用，让您的开发过程更顺畅。导入公用less文件的步骤和技巧。'
cover: '/images/js.webp'
tags: ['javascript', 'vue3', 'vite']
---

vue3 可使用以下配置:

```js
// vue.config.js
module.exports = defineConfig({
  ..., // 其他配置
  css: {
    // css预设器配置项
    loaderOptions: {
      less: {
        additionalData: `@import '@/assets/common.less';`,
      },
    }
  }
})
```

vite 可使用以下配置:

```js
// vite.config.js
export default defineConfig({
  base: './',
  outputDir: 'dist',
  css: {
    // css预处理器
    preprocessorOptions: {
      less: {
        charset: false,
        additionalData: '@import "./src/assets/common.less";',
      },
    },
  },
})
```
