
# 参考

工程：[vue-element-admin](https://github.com/PanJiaChen/vue-element-admin)

文章：[Webpack 大法之 Code Splitting](https://zhuanlan.zhihu.com/p/26710831)

文章：[Webpack Freestyle 之 Long Term Cache](https://zhuanlan.zhihu.com/p/27710902)

# 分析插件

 webpack bundle analyzer 

# 代码分离

Code Splitting 其实就是把代码分成很多独立文件 (又称作 chunk)。

主要有 2 种方式：

- **分离业务代码和第三方库（ vendor ）** 第三方库代码更新迭代相对较慢且可以锁定版本，所以可以充分利用浏览器的缓存来加载这些第三方库；
- **按需加载（利用 import() 语法）** 按需加载的场景，比如说「访问某个路由的时候再去加载对应的组件」，用户不一定会访问所有的路由，所以没必要把所有路由对应的组件都先在开始的加载完；

## 1. 明确有哪些第三方库分离打包

```js
// webpack.config.js
module.exports = {
  entry: {
    app: './src/main.js',
    // 明确有哪些公共模块要单独打包
    vendor: ['vue', 'axios'],
  },
}
//问题出现： app.js 和 vendor.js 中都还存在 vue 和 axios 这些公共包
```

**使用 CommonsChunkPlugin**

```js
// webpack.config.js
new webpack.optimize.CommonsChunkPlugin({
  // vendor 中明确的模块如果其他入口（app）中有被依赖，不会打包进其他入口（app）
  name: 'vendor',
}),
//问题出现： 公共库很可能会越来越多，每次引入了新的第三方库都需要在 vendor 手动增加对应的包名。
// vendor: ['vue','axio','vue-router','vuex','element-ui',]
```

## 2. 自动将被依赖2次以上的第三方库分离打包 

```js
// webpack.config.js
entry: {
  // vendor: ['vue', 'axios'] // 删掉!
},
new webpack.optimize.CommonsChunkPlugin({
  // 模块是来自 node_modules 目录下，.js 文件名， 将自动分离到 vendor.js
  name: 'vendor',
  minChunks: ({ resource }) => (
    resource &&
    resource.indexOf('node_modules') >= 0 &&
    resource.match(/\.js$/)
  ),
}),
```

## 3. 「按需加载」路由组件

```js
// router.js
const Emoji = () => import(
  /* webpackChunkName: "Emoji" */
  './pages/Emoji.vue')

const Photos = () => import(
  /* webpackChunkName: "Photos" */
  './pages/Photos.vue')


// webpack.config.js
output: {
  chunkFilename: '[name].chunk.js',
}


// .babelrc
// 需要安装插件： babel plugin syntax dynamic import 来解析 import() 语法
{
  "plugins": ["syntax-dynamic-import"]
}

// 多了 2 个 async chunk ，分别对应着我们 2 个路由的组件。但是！他们居然都包含了 axios 。

// webpack.config.js
new webpack.optimize.CommonsChunkPlugin({
  async: 'common-in-lazy',
  minChunks: ({ resource } = {}) => (
    resource &&
    resource.includes('node_modules') &&
    /axios/.test(resource)
  ),
}),

// 除了axios,如果其他vue业务组件被引用多次呢？

new webpack.optimize.CommonsChunkPlugin({
  async: 'used-twice',
  minChunks: (module, count) => (
    count >= 2
  ),
})

```

# 持久性缓存

通过更新时调整缓存的文件名，哪个模块更新了破坏他的缓存，没更新的模块继续利用缓存。

```js
// 给我们打包后的静态文件生成随机的唯一的名字
// webpack.config.js
module.exports = {
  output: {
    //...
    filename: '[name].[chunkhash:8].js',
    chunkFilename: '[name].[chunkhash:8].chunk.js',
    //...
  },
}
```

```js
// manifast chunk 把 webpack 的 runtime 代码提取出来
// 当 app.vue代码调整时， 只有 app 和 manifast chunk 的 hash 变化了。
new webpack.optimize.CommonsChunkPlugin({ 
  name: ['manifast'] 
}),
// 但是，假如我们给 App.vue 随便引入一个模块的话，所有 chunk 的 hash 全部都又变了！
```

```js
// 我们需要用一种新的方式来计算 module id 
// 引入 HashedModuleIdsPlugin
plugins: [
  new webpack.HashedModuleIdsPlugin(),
  // ...
],

// OK
```
