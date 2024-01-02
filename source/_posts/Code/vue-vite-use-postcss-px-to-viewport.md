---
title: 如何在Vue、Vite中使用postcss-px-to-viewport？
date: '2022-03-05'
categories: Vue3
keywords: vue-vite-use-postcss-px-to-viewport
description: 如何在Vue、Vite项目中使用postcss-px-to-viewport来实现项目的适配。
cover: /images/js.webp
tags:
  - JavaScript
  - Vue3
  - Vite
abbrlink: 6ef67a7a
---

vue3 可使用以下配置:

```js
// vue.config.js
module.exports = defineConfig({
  ..., // 其他配置
  css: {
    // css预设器配置项
    loaderOptions: {
      postcss: {
        postcssOptions: {
          plugins: [
            require('postcss-px-to-viewport-8-plugin')({
              unitToConvert: 'px', // 要转化的单位
              viewportWidth: 1920, // 设计稿的视口宽度
              // viewportHeight: 1080, // 设计稿的视口宽度
              unitPrecision: 6, // 转换后的精度，即小数点位数
              propList: ['*'], // 指定转换的css属性的单位，*代表全部css属性的单位都进行转换
              viewportUnit: 'vw', // 指定需要转换成的视窗单位，默认vw
              fontViewportUnit: 'vw', // 指定字体需要转换成的视窗单位，默认vw
              selectorBlackList: ['ignore-'], // 指定不转换为视窗单位的类名，
              minPixelValue: 1, // 默认值1，小于或等于1px则不进行转换
              mediaQuery: true, // 是否在媒体查询的css代码中也进行转换，默认false
              replace: true, // 是否转换后直接更换属性值
              // include: [/\/src\/views\/control\//],
              // exclude: [
              //   //
              //   /\/src\/views\//
              // ], // 设置忽略文件，用正则做目录名匹配
              landscape: false, // 是否处理横屏情况
              // landscapeUnit: 'vw', // 横屏时使用的单位
              // landscapeWidth: 2048 // 横屏时使用的视口宽度
            }),
          ],
        },
      }
    }
  }
})
```

vite 可使用以下配置:

```js
// vite.config.js
import postcsspxtoviewport8plugin from 'postcss-px-to-viewport-8-plugin'
export default defineConfig({
  base: './',
  outputDir: 'dist',
  css: {
    postcss: {
      plugins: [
        postcsspxtoviewport8plugin({
          unitToConvert: 'px', // 要转化的单位
          viewportWidth: 1920, // 设计稿的视口宽度
          // viewportHeight: 1080, // 设计稿的视口宽度
          unitPrecision: 6, // 转换后的精度，即小数点位数
          propList: ['*'], // 指定转换的css属性的单位，*代表全部css属性的单位都进行转换
          viewportUnit: 'vw', // 指定需要转换成的视窗单位，默认vw
          fontViewportUnit: 'vw', // 指定字体需要转换成的视窗单位，默认vw
          selectorBlackList: ['ignore-'], // 指定不转换为视窗单位的类名，
          minPixelValue: 1, // 默认值1，小于或等于1px则不进行转换
          mediaQuery: true, // 是否在媒体查询的css代码中也进行转换，默认false
          replace: true, // 是否转换后直接更换属性值
          // include: [/\/src\/views\/control\//],
          // exclude: [
          //   //
          //   /\/src\/views\//
          // ], // 设置忽略文件，用正则做目录名匹配
          landscape: false, // 是否处理横屏情况
          // landscapeUnit: 'vw', // 横屏时使用的单位
          // landscapeWidth: 2048 // 横屏时使用的视口宽度
        }),
      ],
    },
  },
})
```

如果需要对单独的文件夹适配不同的分辨率，可以通过 `exclude` 来排除其他文件夹，然后再指定一组配置，vite、vue 同理，修改 `plugins` 内的数组即可，如下:

```js
// vite.config.js
import postcsspxtoviewport8plugin from 'postcss-px-to-viewport-8-plugin'
export default defineConfig({
  css: {
    postcss: {
      plugins: [
        postcsspxtoviewport8plugin({
          unitToConvert: 'px', // 要转化的单位
          viewportWidth: 2048, // 设计稿的视口宽度
          // viewportHeight: 1536, // 设计稿的视口宽度
          unitPrecision: 6, // 转换后的精度，即小数点位数
          propList: ['*'], // 指定转换的css属性的单位，*代表全部css属性的单位都进行转换
          viewportUnit: 'vw', // 指定需要转换成的视窗单位，默认vw
          fontViewportUnit: 'vw', // 指定字体需要转换成的视窗单位，默认vw
          selectorBlackList: ['ignore-'], // 指定不转换为视窗单位的类名，
          minPixelValue: 1, // 默认值1，小于或等于1px则不进行转换
          mediaQuery: true, // 是否在媒体查询的css代码中也进行转换，默认false
          replace: true, // 是否转换后直接更换属性值
          // include: [/\/src\/views\/control\//],
          exclude: [
            //
            /\/src\/views\//,
            /\/src\/views-3840\//,
          ], // 设置忽略文件，用正则做目录名匹配
          landscape: false, // 是否处理横屏情况
          // landscapeUnit: 'vw', // 横屏时使用的单位
          // landscapeWidth: 2048 // 横屏时使用的视口宽度
        }),
        postcsspxtoviewport8plugin({
          unitToConvert: 'px', // 要转化的单位
          viewportWidth: 3840, // 设计稿的视口宽度
          // viewportHeight: 1536, // 设计稿的视口宽度
          unitPrecision: 6, // 转换后的精度，即小数点位数
          propList: ['*'], // 指定转换的css属性的单位，*代表全部css属性的单位都进行转换
          viewportUnit: 'vw', // 指定需要转换成的视窗单位，默认vw
          fontViewportUnit: 'vw', // 指定字体需要转换成的视窗单位，默认vw
          selectorBlackList: ['ignore-'], // 指定不转换为视窗单位的类名，
          minPixelValue: 1, // 默认值1，小于或等于1px则不进行转换
          mediaQuery: true, // 是否在媒体查询的css代码中也进行转换，默认false
          replace: true, // 是否转换后直接更换属性值
          // include: [/\/src\/views\/control\//],
          exclude: [
            //
            /\/src\/views\//,
            /\/src\/views-control\//,
          ], // 设置忽略文件，用正则做目录名匹配
          landscape: false, // 是否处理横屏情况
          // landscapeUnit: 'vw', // 横屏时使用的单位
          // landscapeWidth: 2048 // 横屏时使用的视口宽度
        }),
      ],
    },
  },
})
```
