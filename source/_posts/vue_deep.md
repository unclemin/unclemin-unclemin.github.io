---
title: Vue深入浅出
date: 2020-04-12 14:53:27
tags: 
  - vue
categories: web前端
---


### 1.浏览器的渲染流程

浏览器加载html的文件时，会先将html文件解析成一个`dom`数，并识别加载`dom`样式，和dom树一起合并渲染。有了渲染树之后，渲染引擎将计算所有元素的位置信息，最后绘制在屏幕上。js引擎和渲染引擎是两个独立的线程，但是js引擎可以触发渲染引擎工作，当我们用js来改变`dom`中元素的外观时，会调用相关的`api`方法操作`dom`对象，会触发渲染引擎回流或者重绘。

- 回流： 当我们对`dom`的修改引发了元素尺寸的变化时，浏览器需要重新计算元素的大小和位置，最后将重新计算的结果绘制出来，这个过程称为回流。

- 重绘： 当我们对`dom`的修改只单纯改变元素的颜色时，浏览器此时并不需要重新计算元素的大小和位置，而只要重新绘制新样式。这个过程称为重绘
- 很显然回流比重绘更加耗费性能，当童通过js修改dom时，会触发渲染引擎的回流或者重绘，对性能的开销是非常大的，所以要尽可能的减少`dom`操作



### 2.Vnode

Vue渲染机制的优化上，引进了`virtual dom`的概念，它是用`Vnode`这个构造函数去描述一个`dom`节点,也就是操作`dom`时会优先操作`virtual dom`这个js对象，最后通过对比，将要改动的部分通知并更新到真实的`dom`中，进而减少渲染引擎绘制的次数，`virtual dom`就是一个中间桥梁


### keep- alive

#### 基本用法和参数
- 保持组件的状态，避免重复渲染造成的性能损失
- 只需要在动态组件的外面包裹就好

- 可接受三个参数进行匹配，对应的组件即可进行缓存
  - `include` 包含的组件(可以为字符串，数组，以及正则表达式,只有匹配的组件会被缓存)
  - `exclude` 排除的组件(以为字符串，数组，以及正则表达式,任何匹配的组件都`不`会被缓存)
  - `max` 缓存组件的最大值(类型为字符或者数字,可以控制缓存组件的个数，超出会移出第一个缓存)


  ```vue
  <!-- 只缓存组件 name 为 c 或者 j 的组件-->
  <keep-alive include="c,j">
    <component></component>
  </keep-alive>
  
  <!-- 组件 name为 m 的组件不缓存 -->
  <keep-alive exclude="m"> 
    <component></component>
  </keep-alive>

  ```

  - 如果参数里面是正则表达式或者方法的话，就需要动态绑定
  - 同时使用了`include`和`exclude`，`exclude`的优先级会更高，以`exclude`的条件为主


  ```vue
  <!-- 使用正则表达式，需使用 v-bind -->
  <keep-alive :include="/c|j/">
    <component></component>
  </keep-alive>
  <!-- 动态判断 -->
  <keep-alive :include="includedComponents">
    <router-view></router-view>
  </keep-alive>

  ```


  ```vue
  <!-- 动态判断，exclude 的优先级会更高,只缓存 c 组件-->
  <keep-alive :include="c,m" exclude="m" max="5">
    <component></component>
  </keep-alive>

  ```

  

  ### `keep-alive`的生命周期钩子函数
  ##### 页面第一次进入时的执行顺序
    - `created` > `mounted` > `activated`(进入时触发的钩子函数)
    - 退出时触发`deactivated`
    - 当再次进入（前进或者后退）时，只触发activated，并且：`activated`,`deactivated`这两个生命周期函数一定是要在使用了`keep-alive`组件后才会有的，否则则不存在
  ##### 缓存路由里面的页面

  ```
  <!-- 所有路径匹配到的视图组件都会被缓存,像 elemnt-admin 这些框架，多级路由就不会全部缓存，需要参考官方文档 -->
  <keep-alive>
      <router-view>
      </router-view>
  </keep-alive>
  ```


- 只缓存router-view里面的某个组件

  - 可以用`include`和`exclude`，但是如果组件太多的话会很麻烦，不是很好的选择


  ```vue
  <!-- 只有路径匹配到的 include 为 c 的组件会被缓存 -->
  <keep-alive include="c">
      <router-view>
      </router-view>
  </keep-alive>
  ```

  

  - 使用路由里面的 meta  属性，更方便扩展和管理

  - app.vue,通过路由的属性来判断

    

  ```vue
  <keep-alive>
      <router-view v-if="$route.meta.keepAlive"></router-view>
  </keep-alive>
  <router-view v-if="!$route.meta.keepAlive"></router-view>
  ```

  

  ```js
  {
          path: '/',
          name: 'C',
          component: C,
          meta: {
              keepAlive: true // 需要被缓存
          }
  }
  {
          path: '/',
          name: 'J',
          component: J,
          meta: {
              keepAlive: fslse // 不需要被缓存
          }
  }
  ```

    

