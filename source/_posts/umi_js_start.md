---
title: 蚂蚁 umi.js 初体验
date: 2020-06-12 14:53:27
tags: 
  - react.js
categories: web前端
---


## `umi js` 初体验

`umi js`乌米是蚂蚁团队开发的前端应用框架，完全开箱即用，支持可配置路由和约定式路由，还有`dva` 的整合，状态管理使用更方便,相比官方的脚手架，基本使用的配置都已经配置好了

[官方文档](https://umijs.org/)

## 安装`umi js`,并运行

这里使用的`yarn`

```shell
yarn create @umijs/umi-app

yarn

yarn start
```

## 配置路由

这里使用的是配置路由，当然你也可以使用约定式路由，删除`routes`即可

如果没有 `routes` 配置，`Umi` 会进入约定式路由模式，然后分析 `src/pages` 目录拿到路由配置。

[路由文档](https://umijs.org/zh-CN/docs/convention-routing)

```js
routes: [
    {
      path: '/',
      component: '@/pages/index',
    },
    {
      path: '/user',
      component: '@/pages/user/user',
    },
    {
      path: '/*',
      component: '@/pages/error/404',
    },
  ],
```

## 状态管理`dva`

- 按照目录约定注册`model`
- 文件名字就是`namespace`
- 只需要通过配置开启即可使用
- https://umijs.org/zh-CN/plugins/plugin-model

```js
// .umirc.js or ts 里面配置打开dva  
dva: {
    immer: true,
    hmr: true,
  },
```

-  注意

model 分两类，一是全局 model，二是页面 model。全局 model 存于 `/src/models/` 目录，所有页面都可引用；页面 model 不能被其他页面所引用

​	`src`目录下面新建一个`models`文件夹，`user.ts`文件

```ts
export default {
    namespace: 'user',
    state: [
      { name: 'dva', id: 'dva' },
      { name: 'antd', id: 'antd' },
    ],
    reducers: {
      delete(state: any[], { payload: id }: any) {
        return state.filter(item => item.id !== id);
      },
    },
  };

  // namespace: '', // 表示在全局 state 上的 key
  // state: {}, // 状态数据
  //reducers: {}, // 管理同步方法，必须是纯函数
  //effects: {}, // 管理异步操作，采用了 generator 的相关概念
  //subscriptions: {}, // 订阅数据源
```

​	`user.tsx`里面使用

```tsx
import React, { useEffect } from 'react';
import { connect } from 'dva'

const   User = ({ dispatch, user }:any) => {
  function handleDelete(id: any) {
    dispatch({
      type: 'user/delete',
      payload: id,
    });
  }
  useEffect(()=>{
    console.log(user)
  },[])
  return (
    <div>
      <h2>List of Products</h2>
    </div>
  );
};

export default connect(({ user }:any) => ({
    user,
}))(User);
```

## 跨域配置

在`.umirc.ts`里面配置

```js
  proxy: {
    '/api': {
      target: 'http://服务器地址',
      pathRewrite: { '^/api': '' },
      changeOrigin: true
    }
  },
```

## 请求封装 `umi-request`

```JS

import { extend } from 'umi-request';
import { notification } from 'antd';

const codeMessage = {
  200: '服务器成功返回请求的数据。',
  201: '新建或修改数据成功。',
  202: '一个请求已经进入后台排队（异步任务）。',
  204: '删除数据成功。',
  400: '发出的请求有错误，服务器没有进行新建或修改数据的操作。',
  401: '用户没有权限（令牌、用户名、密码错误）。',
  403: '用户得到授权，但是访问是被禁止的。',
  404: '发出的请求针对的是不存在的记录，服务器没有进行操作。',
  406: '请求的格式不可得。',
  410: '请求的资源被永久删除，且不会再得到的。',
  422: '当创建一个对象时，发生一个验证错误。',
  500: '服务器发生错误，请检查服务器。',
  502: '网关错误。',
  503: '服务不可用，服务器暂时过载或维护。',
  504: '网关超时。',
};

/**
 * 异常处理程序
 */
const errorHandler = (error:any) => {
  const { response } = error;

  if (response && response.status) {
    let statusa = response.status
    const errorText =  response.statusText;
    const { status, url } = response;
    notification.error({
      message: `请求错误 ${status}: ${url}`,
      description: errorText,
    });
  }

  return response;
};
const request = extend({
  errorHandler,
  prefix:process.env.NODE_ENV == 'development' ? '/api' : 'http://www.aiyu97.online', // 分环境api  prefix地址记得加上http://
  // 默认错误处理
  credentials: 'include', // 默认请求是否带上cookie

});

// request拦截器, 改变url 或 options.
request.interceptors.request.use((url,options) =>  {
  console.log(process.env.NODE_ENV)
  let c_token = localStorage.getItem("x-auth-token");
  if (c_token) {
    const headers = {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
      'x-auth-token': c_token
    };
    return (
      {
        url: url,
        options: { ...options, headers: headers },
      }
    );
  } else {
    const headers = {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
      'x-auth-token': c_token
    };
    return (
      {
        url: url,
        options: { ...options },
      }
    );
  }

})

// response拦截器, 处理response
request.interceptors.response.use((response, options) => {
  console.log(options)
  console.log(response)

  let token = response.headers.get("x-auth-token");
  if (token) {
    localStorage.setItem("x-auth-token", token);
  }
  return response;
});


export default request;

```

使用

```js
import request from '@/utils/request';

// 测试登录
export async function Login(params: any) {
	return request('/data', {
		// 请求方式
		method: 'GET',
		// 用data包裹参数是官方指定写法，如果data有参数umi-request会默认读取data里面参数。
		data: params
	})
}
```

## `.umirc.ts`配置

```js
  base: '/web/',  //部署到非根目录时才需配置
  targets: { //配置浏览器最低版本,比如兼容ie11
   ie: 11
  },
  hash: true,  //开启打包文件的hash值后缀
  treeShaking: true, //去除那些引用的但却没有使用的代码
```

