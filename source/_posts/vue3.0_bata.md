---
title: Vue 3.0 bate版尝鲜
date: 2020-04-22 19:53:27
tags: 
  - vue.js
  - vue3.0
categories: web前端
top: 1
---




4月21日晚，`Vue`作者尤雨溪在哔哩哔哩直播分享了`Vue.js 3.0 Beta`最新进展，也建议前端开发者们不要在大项目中使用，小项目是可以的，怀着激动心情，体验一下`3.0`版本，目前并未全局安装`vue-next`

- 官方相关文档参考
- https://composition-api.vuejs.org/#summary
- https://github.com/vuejs/vue-next

### 1.新建文件夹,克隆项目

```shell
git clone https://github.com/vuejs/vue-next-webpack-preview.git
```

### 2.进入项目目录，安装依赖

```shell
yarn install
```

![](https://cdn.jsdelivr.net/gh/unclemin/images/Img/微信截图_20200423104225.png)

![](https://cdn.jsdelivr.net/gh/unclemin/images/Img/code.png)

`demo`运行成功示例，用过`react hooks`的应该不陌生，数据初始化很类似`useState`，`vue3.0`的可以把逻辑相同的地方抽离出来，方便代码量大了后维护



### 3.创建一个获取鼠标位置组件

-  项目下新建`test.js`文件

```js
import { ref, onMounted, onUnmounted } from 'vue' // 按需引入使用模块

export function useMousePosition() {
  const x = ref(0)
  const y = ref(0)
	// 初始化下x，y
  function update(e) {
    x.value = e.pageX
    y.value = e.pageY
  }
	// 绑定监听鼠标移动事件
  onMounted(() => {
    window.addEventListener('mousemove', update) 
  })

  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })

  return { x, y }
}
```

- 引入组件

```js
<template>
  <img src="./logo.png">
  <h2>X:{{x}}</h2>
  <h2>y:{{y}}</h2>
</template>
<script>
import { useMousePosition } from './test'
export default {
  setup() {
    const { x, y } = useMousePosition()
    return {
      x,
      y
    }
  }
}
</script>
```

- 运行效果

![](https://cdn.jsdelivr.net/gh/unclemin/images/Img/微信截图_20200423110049.png)