如果我们在webpack配置文件名的时候用到了`[hash]`那么现在肯定就会发现一个问题：`dist`目录中的文件越来越多，因为之前打包的旧文件一直留在目录中，这样造成目录内杂乱无章，想找本次打包的资源都有些费劲。  
所以我们迫切的想在每次构建前先清空`dist`目录，[clean-webpack-plugin](//github.com/johnagan/clean-webpack-plugin)便可以满足我们这个需求。
## clean-webpack-plugin
`clean-webpack-plugin`可以帮助我们清空构建输出目录，默认是 `output.path` 目录
### Usage
```js 
// 安装
npm install -D clean-webpack-plugin

// webpack.config.js中引入
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

// 在plugins中使用
const webpackConfig = {
    plugins: [
        new CleanWebpackPlugin(), // 默认清空构建输出目录，但不是删除目录
    ],
};
```
现在在重新构建一次我们发现之前的文件都会被删除，`output.path`内只保留了本次构建的结果。  
**clean-webpack-plugin`新版本的引用方式改变了，如果发现报错检查下是不是用的旧的引用方式**
```js
// 旧的引用方式
const CleanWebpackPlugin = require('clean-webpack-plugin');
```
