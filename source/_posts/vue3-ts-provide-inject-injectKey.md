---
title: 'Vue3 provide inject如何通过injectKey保存类型'
date: '2023-01-17'
categories: '技术分享'
keywords: 'vue3-ts-provide-inject-injectKey'
description: '学习如何在Vue3中使用provide-inject来共享数据，并通过使用injectKey来保存类型，避免上下文丢失问题。'
cover: '/images/ts.webp'
tags: ['typescript', 'vue3']
---

# Vue3 provide inject 正常使用方式

父页面通过 `provide` 注入对象信息，子页面通过 `inject` 获取对象信息。

可以通过 `injectKey` 来保存类型，避免上下文丢失问题。

```ts
// userInfoKey.ts
import { InjectionKey } from 'vue'

export interface UserInfo {
  id: Number
  name: String
}

export const userInfoKey: InjectionKey<UserInfo> = Symbol()
```

```ts
// Parent.vue
import { userInfoKey, UserInfo } from '@/composables/userInfoKey'

provide(userInfoKey, {
  id: 1,
  name: 'test',
})

const userInfo: UserInfo = {
  id: 1,
  name: 'test',
}
provide('userInfo', userInfo)
```

```ts
// Child.vue
import { userInfoKey, UserInfo } from '@/composables/userInfoKey'
// 丢失上下文的对象
const userInfo1 = inject('userInfo')

// 保留了对象的类型
const userInfo2 = inject(userInfoKey)
```
