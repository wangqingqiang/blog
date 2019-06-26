`webpack`的构建环境分为`development`开发环境和`production`生产环境，两者存在较大差异。
## development
开发环境下的目标：
  - 强大的source map 快速定位错误
  - live reloading 页面刷新重加载
  - HotModuleReplacement 热更新
  - local server 本地服务
## production
生产环境下的目标：
- 更轻量的source map 不影响性能、体积
- 压缩bunlde
- css提取
- 公共代码提取
- 其他优化资源

## webpack-merge
由于目标不同所以两种模式的配置也对应不同，所以我们需要写两个配置文件，但是它们之间还有许多共同点，比如`entry、output、plugins`等。为了避免重复写两次共同的配置项，我们可以把公共配置项提取出一个单独文件`webpack.common.js`，然后用`webpack-merge`进行合并：
```js
// webpack.common.js
// ...
module.exports = {
    entry: {
        index: './src/index.js',
    },
    output: {
        path: './dist',
        filename: ‘[name][hash:5].js’
    },
    module: {
        rules: [
            // ...
            // js、css、image、font等文件loader
            // ...
        ]
    },
    // ...
}

// webpack.dev.js
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
    mode: 'development',
    devtool: 'inline-source-map',
    devServer: {
        contentBase: './dist',
        hot: true,
        open: true
    }
})

// webpack.prod.js
const common = require('./webpack.common.js');
const merge = require('webpack-merge');

module.exports = merge(common, {
    mode: 'production',
    optimization: {
        usedExports: true,
    }
});

// package.json 中script更改：
"dev": "webpack-dev-server --config webpack.dev.js"
"build": "webpack --config webpack.prod.js"

```
