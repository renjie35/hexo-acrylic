---
title: 'Vue3中子组件中的v-model调用时穿透|如何不破坏单向数据流'
date: '2022-09-08'
categories: '技术分享'
keywords: 'vue3-v-model-one-way-data-flow'
description: '在Vue3中，当v-model传递给子组件时，如何在子组件中使用v-model属性而不破坏单向数据流？'
cover: '/images/js.webp'
tags: ['javascript', 'vue3']
---

目前 Vue3 中，v-model 的实现是通过 `v-bind="$attrs"` 和 `v-on="$listeners"` 来实现的，这样就会导致子组件中的 v-model 属性会被传递到子组件中，如果子组件中也有 v-model 属性，那么就会导致子组件中的 v-model 属性会被覆盖，从而破坏了单向数据流。

那么如何在子组件中使用 v-model 属性而不破坏单向数据流呢?

可以利用 `computed` 属性配合 `Proxy` 来实现，再写成通用的 `composables` ，如下代码：

此处注意如果不使用 `Proxy` 的话，会导致 v-model 只更改了子组件中的属性，而父组件中的属性没有更改，所以这里使用 `emit('update:xxx')` 来实现更新父组件中的属性。

```js
//useVModel.js
import { computed } from 'vue'

export function useVModel(props, propName, emit) {
  return computed({
    get() {
      console.log('get')
      return new Proxy(props[propName], {
        set(obj, name, value) {
          console.log('set', obj, name, value)
          emit('update:' + propName, {
            ...obj,
            [name]: value,
          })
          return true
        },
      })
    },
    set(val) {
      console.log('set')
      emit('update:' + propName, val)
    },
  })
}
```

调用时如下：

```js
//demo.vue
<script setup>
import { useVModel } from '@/composables/useVModel'

const props = defineProps({
  modelValue: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['update:modelValue'])
const model = useVModel(props, 'modelValue', emit)
</script>

<template>
  <input v-model="model.value" :placeholder="model.placeholder" />
</template>
```
