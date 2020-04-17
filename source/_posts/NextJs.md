---
title: next.js  +  ts+ ant使用
date: 2020-04-12 14:53:27
tags: 
  - next.js
categories: web前端
---


`Next.js`来实现`React`的服务端渲染主要是便捷、高效，因为`Next.Js`能够简单的实现服务端的渲染

#### 基础环境搭建

- 新建文件夹 `mkdir demo`
- 进入新建的文件目录，为你的项目安装 `next`、`react` 和 `react-dom` 

```js
npm install next react react-dom
```

安装完后的项目文件，pages是自己新建的，next默认的名称必须是pages


![微信截图_20200416170503.png](https://i.loli.net/2020/04/16/wtMf56UbelGOBqm.png)


-  在 `package.json` 文件并添加 `scripts` 配置运行脚本：

```js
"scripts": {
  "dev": "next",
  "build": "next build",
  "start": "next start"
}
```

-  运行 `npm run dev` 在浏览器中查看

#### 支持css文件

```ssh
yarn add @zeit/next-css
#or
npm install @zeit/next-css
```

下载完后在根目录新建一个`next.config.js` 文件，这个是`next`的总配置文件，添加如下代码就可以了

```js
const withCss = require('@zeit/next-css')

if(typeof require !== 'undefined'){
    require.extensions['.css']=file=>{}
}

module.exports = withCss({})
```

#### 引入 `ant design`UI组件

- 安装`antd`，实现按需加载

  ```js
  npm install antd
  ```

- 再安装`babel-plugin-import`

  ```js
  npm install babel-plugin-import
  ```

- 在根目录下新建`.babelrc`文件，并加入如下配置

  ```js
  {
    "presets": ["next/babel"],
    "plugins": [
      [
        "import",
        {
          "libraryName": "antd",
          "style": "css" // `style: true` 会加载 less 文件
        }
      ]
    ]
  }
  ```

- 页面中可以直接引入组件，不需要多引入`ant`的`css`文件，按需加载

  ```js
  import { Button } from 'antd';
  import About from "./about";
  
  const Index = () => (
    <div>
        <About/>
        <Button type="primary">Primary</Button>
    </div>
  )
  export default Index
  ```

  #### 引入`TypeScript`到`next`中

  - 根目录新建`tsconfig.json`文件并执行`npm run dev`,出现以下提示

  ```ssh
  npm run dev
  # You'll see instructions like these:
  #
  # Please install typescript, @types/react, and @types/node by running:
  #
  #         yarn add --dev typescript @types/react @types/node
  #
  # ...
  ```

  - 根据提示执行`yarn add --dev typescript @types/react @types/node` or  `npm install --dev typescript @types/react @types/node`

  - `tsconfig.json`会生成如下配置，就可以使用`tsx`愉快的编写啦

  ```js
  {
    "compilerOptions": {
      "target": "es5",
      "lib": [
        "dom",
        "dom.iterable",
        "esnext"
      ],
      "allowJs": true,
      "skipLibCheck": true,
      "strict": true,
      "forceConsistentCasingInFileNames": true,
      "noEmit": true,
      "esModuleInterop": true,
      "module": "esnext",
      "moduleResolution": "node",
      "resolveJsonModule": true,
      "isolatedModules": true,
      "jsx": "preserve"
    },
    "exclude": [
      "node_modules"
    ],
    "include": [
      "next-env.d.ts",
      "**/*.ts",
      "**/*.tsx", "pages/index.js"
    ]
  }
  ```

    