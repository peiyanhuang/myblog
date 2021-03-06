---
layout: post
title:  Webpack 遇到的error
date:   2016-10-07 13:58:00 +0800
categories: 前端工具
tag: 前端工具
---

* content
{:toc}

### Error: Cannot resolve 'file' or 'directory'

错误原因：获取的文件没有后缀名

必须要检查 `webpack` 的 `resolve` 配置信息，如果没有配置 `resolve`，必须添加在 `webpack` 配置文件中如下信息：

```js
//  用来配置应用层的模块解析，即要被打包的模块
resolve: {
    //  第一项扩展非常重要，千万不要忽略，否则经常出现模块无法加载错误
    extensions: ['', '.js', '.es6', '.vue','.css']
},
```

通过以上配置，webpack会自动为请求的文件添加后缀名，从而保证请求的路径正确无误。

对上面的配置进行说明，即请求js、es6、vue文件时不需要添加后缀名，直接请求即可，如下。

    import {add} from './MathUtils'

### CSS

webpack提供两个工具处理样式表，`css-loader` 和 `style-loader`，二者处理的任务不同，css-loader 使你能够使用类似 `@import` 和 `url(...)` 的方法实现 `require()` 的功能， `style-loader` 将所有的计算后的样式加入页面中，二者组合在一起使你能够把样式表嵌入 webpack 打包后的JS文件中。

### 4. 支持 browserHistory

1. `webpack-dev-server`: 设置 `historyApiFallback: true`。

2. `webpack-dev-middleware`: 如下设置

```js
// browserHistory
app.use("*", (req, res, next) => {
    const filename = path.resolve(compiler.outputPath, "index.html");
    compiler.outputFileSystem.readFile(filename, (err, result) => {
        if (err) {
            return next(err);
        }
        res.set("content-type", "text/html");
        res.send(result);
        res.end();
    });
});
```

注意：这要放在 `webpack-dev-middleware` 和 `webpack-hot-middleware` 中间件之后。

由于 `webpack-dev-middleware` 只是将文件打包在内存里，所以你没法在 `express` 里直接 `sendFile(dist/index.html')`，因为这个文件实际上还不存在。还好 `webpack` 提供了一个 `outputFileStream`，用来输出其内存里的文件，我们可以利用它来做路由。

3. `connect-history-api-fallback`：

```js
// BrowserRouter
app.use('/', require('connect-history-api-fallback')());
app.use('/', express.static('../src'));  // 开发的 index.html 目录
```

注意：这要放在 `webpack-dev-middleware` 和 `webpack-hot-middleware` 中间件之前。