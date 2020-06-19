---
title: react.js 打包优化  
date: 2020-06-12 14:53:27
tags: 
  - react.js
categories: web前端
---


## React .js 打包配置优化

- 优化前1m多，优化后300多kb，优化还是挺明显

## 1.使用`create-react-app`脚手架创建一个`React`应用。

`npx create-react-app my-app --typescript`

### 2.创建`webpack`配置

`yarn add react-app-rewired customize-cra-D`

- 将默认的 `package.json` 里面的 `scripts` 代码修改为一下

  ```json
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
  }
  ```

- 然后在根目录创建一个 `config-overrides.js` 用于修改默认配置

  ```js
  const {
      override,
      fixBabelImports,
      addLessLoader,
      addWebpackAlias,
      addWebpackPlugin,
  } = require("customize-cra");
  const { BundleAnalyzerPlugin } = require("webpack-bundle-analyzer");
  const ProgressBarPlugin = require("progress-bar-webpack-plugin");
  const CompressionWebpackPlugin = require("compression-webpack-plugin");
  // 查看打包后各包大小
  console.log(process.env.NODE_ENV)
  
  const isEnvProduction = process.env.NODE_ENV === "production";
  
  const addCompression = () => config => {
      if (isEnvProduction) {
          config.plugins.push(
              // gzip压缩
              new CompressionWebpackPlugin({
                  test: /\.(css|js)$/,
                  // 只处理比1kb大的资源
                  threshold: 1024,
                  // 只处理压缩率低于90%的文件
                  minRatio: 0.9
              })
          );
      }
      return config;
  };
  // 查看打包产物
  const addAnalyzer = () => config => {
      if (isEnvProduction) {
          config.plugins.push(new BundleAnalyzerPlugin());
      }
  
      return config;
  };
  // 配置cdn
  const addCdn = () => config => {
      if (isEnvProduction) {
          // 关闭sourceMap
          config.devtool = false;
          config.externals = {
              'react': 'React',
              'react-dom': 'ReactDOM',
          }
      }
      return config;
  }
  
  const addCommonsChunkPlugin = () => config => {
      if (isEnvProduction) {
          config.optimization.splitChunks = {
              chunks: 'all',
              name: "vender",
              cacheGroups: {
                  vendors: {
                      test: /[\\/]node_modules[\\/]/,
                      name: 'vendors',
                      minSize: 50000,
                      minChunks: 1,
                      chunks: 'initial',
                      priority: 1, // 该配置项是设置处理的优先级，数值越大越优先处理，处理后优先级低的如果包含相同模块则不再处理
                  },
                  commons: {
                      test: /[\\/]src[\\/]/,
                      name: 'commons',
                      minSize: 50000,
                      minChunks: 2,
                      chunks: 'initial',
                      priority: -1,
                      reuseExistingChunk: true, // 这个配置允许我们使用已经存在的代码块
                  },
                  lodash: {
                      name: 'lodash', // 单独将 lodash 拆包
                      priority: 20,
                      test: /[\\/]node_modules[\\/]lodash[\\/]/,
                      chunks: 'all',
                  },
                  reactLib: {
                      name: 'react-lib', // 单独将 lodash 拆包
                      priority: 20,
                      test: /[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]/,
                      chunks: 'all',
                  },
  
              },
          }
          return config;
      }
  }
  /* config-overrides.js */
  module.exports = override(
      addCdn(),
      addAnalyzer(),
      addCompression(),
      // addCommonsChunkPlugin(),
    addWebpackPlugin(
          // 终端进度条显示
          ProgressBarPlugin()
      ),
  );
  ```
  
  