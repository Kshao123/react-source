# 配置 React 源码的本地调试环境

1. creat-react-app <项目名称>
2. yarn run eject
3. clone 官方源码（目前是 master latest）小版本可能会有些许异同，可以根据命令行的报错信息再去搜索（镜像 react 仓库，clone 慢的可以使用这个）

###### 根目录中执行
```shell
git clone --depth=1 https://github.com.cnpmjs.org/facebook/react.git src/react
```

# 修改相关配置

1. 链接本地源码
`react/config/webpack.config.js`

```diff
resolve: {
    alias: {
        'react-native': 'react-native-web',
-       ...(isEnvProductionProfile && {
-         'react-dom$': 'react-dom/profiling',
-         'scheduler/tracing': 'scheduler/tracing-profiling',
-       }),
-       ...(modules.webpackAliases || {}),
        
+        'react': path.resolve(__dirname, '../src/react/packages/react'),
+        'react-dom': path.resolve(__dirname, '../src/react/packages/react-dom'),
+        'shared': path.resolve(__dirname, '../src/react/packages/shared'),
+        'react-reconciler': path.resolve(__dirname, '../src/react/packages/react-reconciler'),
         'react-events': path.resolve(__dirname, '../src/react/packages/events')
    }
}
```

2. 修改环境变量

> 这一步不同版本中可能会出现变量不对等的情况
> 如有报错可以看下源码中的 `.eslintrc.js` 里面的 `globals` 属性

`react/config/env.js`

```diff
  const stringified = {
    ....,
    
+    __DEV__: true,
+    SharedArrayBuffer: true,
+    spyOnDev: true,
+    spyOnDevAndProd: true,
+    spyOnProd: true,
+    __PROFILE__: true,
+    __UMD__: true,
+    __EXPERIMENTAL__: true,
+    __VARIANT__: true,
+    gate: true,
+    trustedTypes: true,
  };

```

2.1 项目根目录中创建 `.eslintrc.json` 文件

```json
{
  "extends": "react-app",
  "globals": {
    "SharedArrayBuffer": true,

    "spyOnDev": true,
    "spyOnDevAndProd": true,
    "spyOnProd": true,
    "__PROFILE__": true,
    "__UMD__": true,
    "__EXPERIMENTAL__": true,
    "__VARIANT__": true,
    "gate": true,
    "trustedTypes": true
  }
}
```
3. 忽略 `flow`
> webstrom 中可自动识别 flow，其他编辑器可能需要下载插件 

```shell
yarn add @babel/plugin-transform-flow-strip-types -D
```

3.1 添加配置

`react/config/webpack.config.js[babel-loader]`

```diff
plugins: [
+ require.resolve('@babel/plugin-transform-flow-strip-types'),
  [
    require.resolve('babel-plugin-named-asset-import'),
    {
      loaderMap: {
        svg: {
          ReactComponent:
            '@svgr/webpack?-svgo,+titleProp,+ref![path]',
        },
      },
    },
  ],
],
```

4. 导出 `HostConfig`

修改文件`/react/packages/react-reconciler/src/ReactFiberHostConfig.js`。

```diff
- invariant(false, 'This module must be shimmed by a specific renderer.');
+ export * from './forks/ReactFiberHostConfig.dom'
```

4.1 修改文件`/react/packages/shared/ReactSharedInternals.js`。`react`此时未`export`内容，直接从`ReactSharedInternals`拿值


```diff
- import * as React from 'react';

- const ReactSharedInternals =
-   React.__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED;

+ import ReactSharedInternals from '../react/src/ReactSharedInternals'

export default ReactSharedInternals;
```

4.2 关闭 `eslint` 扩展

`/react/.eslingrc.js[module.exports]`

```diff
  extends: [
-   'fbjs',
    'prettier'
  ],
```

# 相关报错

## invariant

```
Error: Internal React error: invariant() is meant to be replaced at compile time. There is no runtime version.
```
解决


```diff

export default function invariant(condition, format, a, b, c, d, e, f) {
+  return;

  throw new Error(
    'Internal React error: invariant() is meant to be replaced at compile ' +
      'time. There is no runtime version.',
  );
}

```

# 完结撒花

- gihub [源码地址](https://github.com/Kshao123/react-source)
- 版本更新复制 clone 代码即可，建议 pull，避免代码覆盖问题
