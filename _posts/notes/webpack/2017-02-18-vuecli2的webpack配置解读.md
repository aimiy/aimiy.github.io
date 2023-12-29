---
layout: post
title:  vuecli2的webpack配置解读.md
date:   2017-02-18 14:00:00 +0800
categories: 笔记
tag: vuecli2
---
* content
{:toc}

## 目录结构

```markdown
├── README.md
├── build
│   ├── build.js
│   ├── check-versions.js
│   ├── dev-client.js
│   ├── dev-server.js
│   ├── utils.js
│   ├── vue-loader.conf.js
│   ├── webpack.base.conf.js
│   ├── webpack.dev.conf.js
│   └── webpack.prod.conf.js
├── config
│   ├── dev.env.js
│   ├── index.js
│   └── prod.env.js
├── index.html
├── package.json
├── src
│   ├── App.vue
│   ├── assets
│   │   └── logo.png
│   ├── components
│   │   └── Hello.vue
│   └── main.js
└── static
```

## build目录，webpack打包

主要对build目录下的webpack配置做详细分析

### webpack.base.conf.js

入口文件entry

```js
entry: {
  app: '.src/main.js'
}
```

#### 输出文件output

config的配置在config/index.js文件中

```js
output: {
  path: config.build.assetsRoot, //导出目录的绝对路径
  filename: '[name].js', //导出文件的文件名
  publicPath: process.env.NODE_ENV === 'production'? config.build.assetsPublicPath : config.dev.assetsPublicPath //生产模式或开发模式下html、js等文件内部引用的公共路径
}
```

#### 文件解析resolve

主要设置模块如何被解析。

```js
resolve: {
  extensions: ['.js', '.vue', '.json'], //自动解析确定的拓展名,使导入模块时不带拓展名
  alias: {   // 创建import或require的别名
    'vue$': 'vue/dist/vue.esm.js',
    '@': resolve('src')
  }
}
```

#### 模块解析module

如何处理项目不同类型的模块。

```js
module: {
  rules: [
    {
      test: /\.vue$/, // vue文件后缀
      loader: 'vue-loader', //使用vue-loader处理
      options: vueLoaderConfig //options是对vue-loader做的额外选项配置
    },
    {
      test: /\.js$/, // js文件后缀
      loader: 'babel-loader', //使用babel-loader处理
      include: [resolve('src'), resolve('test')] //必须处理包含src和test文件夹
    },
    {
      test: /\.(png|jpe?g|gif|svg)(\?.*)?$/, //图片后缀
      loader: 'url-loader', //使用url-loader处理
      query: {  // query是对loader做额外的选项配置
        limit: 10000, //图片小于10000字节时以base64的方式引用
        name: utils.assetsPath('img/[name].[hash:7].[ext]') //文件名为name.7位hash值.拓展名
      }
    }，
    {
      test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/, //字体文件
      loader: 'url-loader', //使用url-loader处理
      query: {
        limit: 10000,  //字体文件小于1000字节的时候处理方式
        name: utils.assetsPath('fonts/[name].[hash:7].[ext]') //文件名为name.7位hash值.拓展名
      }
    }
  ]
}
```

注: 关于query 仅由于兼容性原因而存在。请使用 options 代替。

### webpack.dev.conf.js

开发环境下的webpack配置，通过merge方法合并webpack.base.conf.js基础配置

```js
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
module.exports = merge(baseWebpackConfig, {}）
```

#### 模块配置

```js
module: {
  //通过传入一些配置来获取rules配置，此处传入了sourceMap: false,表示不生成sourceMap
  rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
}
```

在util.styleLoaders中的配置如下

```js
exports.styleLoaders = function (options) {
  var output = [] //定义返回的数组，数组中保存的是针对各类型的样式文件的处理方式
  var loaders = exports.cssLoaders(options) // 调用cssLoaders方法返回各类型的样式对象(css: loader)
  for (var extension in loaders) {  //循环遍历loaders
    var loader = loaders[extension] //根据遍历获得的key(extension)来得到value(loader)
    output.push({     //
      test: new RegExp('\\.' + extension + '$'), // 处理的文件类型
      use: loader  //用loader来处理，loader来自loaders[extension]
    })
  }
  return output
}
```

上面的代码中调用了exports.cssLoaders(options),用来返回针对各类型的样式文件的处理方式,具体实现如下

```js
exports.cssLoaders = function (options) {
  options = options || {}

  var cssLoader = {
    loader: 'css-loader',
    options: {  //options是loader的选项配置
      minimize: process.env.NODE_ENV === 'production', //生成环境下压缩文件
      sourceMap: options.sourceMap  //根据参数是否生成sourceMap文件
    }
  }
  function generateLoaders (loader, loaderOptions) {  //生成loader
    var loaders = [cssLoader] // 默认是css-loader
    if (loader) { // 如果参数loader存在
      loaders.push({
        loader: loader + '-loader',
        options: Object.assign({}, loaderOptions, { //将loaderOptions和sourceMap组成一个对象
          sourceMap: options.sourceMap
        })
      })
    }
    if (options.extract) { // 如果传入的options存在extract且为true
      return ExtractTextPlugin.extract({  //ExtractTextPlugin分离js中引入的css文件
        use: loaders,  //处理的loader
        fallback: 'vue-style-loader' //没有被提取分离时使用的loader
      })
    } else {
      return ['vue-style-loader'].concat(loaders)
    }
  }
  return {  //返回css类型对应的loader组成的对象 generateLoaders()来生成loader
    css: generateLoaders(),
    postcss: generateLoaders(),
    less: generateLoaders('less'),
    sass: generateLoaders('sass', { indentedSyntax: true }),
    scss: generateLoaders('sass'),
    stylus: generateLoaders('stylus'),
    styl: generateLoaders('stylus')
  }
}
```

#### 插件配置

```js
plugins: [
  new webpack.DefinePlugin({ // 编译时配置的全局变量
    'process.env': config.dev.env //当前环境为开发环境
  }),
  new webpack.HotModuleReplacementPlugin(), //热更新插件
  new webpack.NoEmitOnErrorPlugin(), //不触发错误,即编译后运行的包正常运行
  new HtmlWebpackPlugin({  //自动生成html文件,比如编译后文件的引入
    filename: 'index.html', //生成的文件名
    template: 'index.html', //模板
    inject: true
  }),
  new FriendlyErrorsPlugin() //友好的错误提示
]
```

### webpack.prod.conf.js

生产环境下的webpack配置，通过merge方法合并webpack.base.conf.js基础配置

#### module的处理,主要是针对css的处理

同样的此处调用了utils.styleLoaders

```js
module: {
  rules: utils.styleLoaders({
    sourceMap: config.build.productionSourceMap,
    extract: true
  })
}
```

#### 输出output

```js
output: {
  //导出文件目录
  path: config.build.assetsRoot,
  //导出的文件名
  filename: utils.assetsPath('js/[name].[chunkhash].js'),
  //非入口文件的文件名，而又需要被打包出来的文件命名配置,如按需加载的模块
  chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
}
```

#### 插件plugins

```js
var path = require('path')
var utils = require('./utils')
var webpack = require('webpack')
var config = require('../config')
var merge = require('webpack-merge')
var baseWebpackConfig = require('./webpack.base.conf')
var CopyWebpackPlugin = require('copy-webpack-plugin')
var HtmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin')
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
var env = config.build.env
plugins: [
  new webpack.DefinePlugin({
    'process.env': env //配置全局环境为生产环境
  }),
  new webpack.optimize.UglifyJsPlugin({ //js文件压缩插件
    compress: {  //压缩配置
      warnings: false  // 不显示警告
    },
    sourceMap: true //生成sourceMap文件
  }),
  new ExtractTextPlugin({ //将js中引入的css分离的插件
    filename: utils.assetsPath('css/[name].[contenthash].css') //分离出的css文件名
  }),
  //压缩提取出的css，并解决ExtractTextPlugin分离出的js重复问题(多个文件引入同一css文件)
  new OptimizeCSSPlugin(),
  //生成html的插件,引入css文件和js文件
  new HtmlWebpackPlugin({
    filename: config.build.index, //生成的html的文件名
    template: 'index.html', //依据的模板
    inject: true, //注入的js文件将会被放在body标签中,当值为'head'时，将被放在head标签中
    minify: {  //压缩配置
      removeComments: true, //删除html中的注释代码
      collapseWhitespace: true,  //删除html中的空白符
      removeAttributeQuotes: true  //删除html元素中属性的引号
    },
    chunksSortMode: 'dependency' //按dependency的顺序引入
  }),
  //分离公共js到vendor中
  new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',  //文件名
    minChunks: functions(module, count) { // 声明公共的模块来自node_modules文件夹
      return (module.resource && /\.js$/.test(module.resource) && module,resource.indexOf(path.join(__dirname, '../node_modules')) === 0)
    }
  }),
  //上面虽然已经分离了第三方库,每次修改编译都会改变vendor的hash值，导致浏览器缓存失效。原因是vendor包含了webpack在打包过程中会产生一些运行时代码，运行时代码中实际上保存了打包后的文件名。当修改业务代码时,业务代码的js文件的hash值必然会改变。一旦改变必然会导致vendor变化。vendor变化会导致其hash值变化。
  //下面主要是将运行时代码提取到单独的manifest文件中，防止其影响vendor.js
  new webpack.optimize.CommonsChunkPlugin({
    name: 'mainifest',
    chunks: ['vendor']
  }),
  // 复制静态资源,将static文件内的内容复制到指定文件夹
  new CopyWebpackPlugin([{
    from: path.resolve(__dirname, '../static'),
    to: config.build.assetsSubDirectory,
    ignore: ['.*']  //忽视.*文件
  }])
]
```

#### 额外配置

```js
if (config.build.productionGzip) { //配置文件开启了gzip压缩

  //引入压缩文件的组件,该插件会对生成的文件进行压缩，生成一个.gz文件
  var CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]', //目标文件名
      algorithm: 'gzip', //使用gzip压缩
      test: new RegExp( //满足正则表达式的文件会被压缩
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240, //资源文件大于10240B=10kB时会被压缩
      minRatio: 0.8 //最小压缩比达到0.8时才会被压缩
    })
  )
}
```

### npm run dev

有了上面的配置之后，下面看看运行命令npm run dev发生了什么

在package.json文件中定义了dev运行的脚本

```js
"scripts": {
   "dev": "node build/dev-server.js",
   "build": "node build/build.js"
},
```

当运行`npm run dev`命令时，实际上会运行dev-server.js文件

该文件以express作为后端框架

```js
// nodejs环境配置
var config = require('../config')
if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
var opn = require('opn') //强制打开浏览器
var path = require('path')
var express = require('express')
var webpack = require('webpack')
var proxyMiddleware = require('http-proxy-middleware') //使用代理的中间件
var webpackConfig = require('./webpack.dev.conf') //webpack的配置

var port = process.env.PORT || config.dev.port //端口号
var autoOpenBrowser = !!config.dev.autoOpenBrowser //是否自动打开浏览器
var proxyTable = config.dev.proxyTable //http的代理url

var app = express() //启动express
var compiler = webpack(webpackConfig) //webpack编译

//webpack-dev-middleware的作用
//1.将编译后的生成的静态文件放在内存中,所以在npm run dev后磁盘上不会生成文件
//2.当文件改变时,会自动编译。
//3.当在编译过程中请求某个资源时，webpack-dev-server不会让这个请求失败，而是会一直阻塞它，直到webpack编译完毕
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})

//webpack-hot-middleware的作用就是实现浏览器的无刷新更新
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {}
})
//声明hotMiddleware无刷新更新的时机:html-webpack-plugin 的template更改之后
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})

//将代理请求的配置应用到express服务上
Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  if (typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})

//使用connect-history-api-fallback匹配资源
//如果不匹配就可以重定向到指定地址
app.use(require('connect-history-api-fallback')())

// 应用devMiddleware中间件
app.use(devMiddleware)
// 应用hotMiddleware中间件
app.use(hotMiddleware)

// 配置express静态资源目录
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
app.use(staticPath, express.static('./static'))

var uri = '<http://localhost>:' + port

//编译成功后打印uri
devMiddleware.waitUntilValid(function () {
  console.log('> Listening at ' + uri + '\n')
})
//启动express服务
module.exports = app.listen(port, function (err) {
  if (err) {
    console.log(err)
    return
  }
  // 满足条件则自动打开浏览器
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    opn(uri)
  }
})
```

### npm run build

由于package.json中的配置，运行此命令后会执行build.js文件

```js
process.env.NODE_ENV = 'production' //设置当前环境为production
var ora = require('ora') //终端显示的转轮loading
var rm = require('rimraf')  //node环境下rm -rf的命令库
var path = require('path')  //文件路径处理库
var chalk = require('chalk')  //终端显示带颜色的文字
var webpack = require('webpack')
var config = require('../config')
var webpackConfig = require('./webpack.prod.conf') //生产环境下的webpack配置

// 在终端显示ora库的loading效果
var spinner = ora('building for production...')
spinner.start()

// 删除已编译文件
rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
  //在删除完成的回调函数中开始编译
  webpack(webpackConfig, function (err, stats) {
    spinner.stop() //停止loading
    if (err) throw err

    // 在编译完成的回调函数中,在终端输出编译的文件
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false,
      chunks: false,
      chunkModules: false
    }) + '\n\n')
  })
})
```

参考文档:
[vue-cli中的webpack配置](https://www.cnblogs.com/libin-1/p/6596810.html)

## 配置文件

### .postcssrc.js/postcss.config.js

文件内容

```js
module.exports = {
  "plugins": {
    "postcss-import": {},      //用于@import导入css文件
    "postcss-url": {},           //路径引入css文件或node_modules文件
    "postcss-aspect-ratio-mini": {},   //用来处理元素容器宽高比
    "postcss-write-svg": { utf8: false },    //用来处理移动端1px的解决方案。该插件主要使用的是border-image和background来做1px的相关处理。
    "postcss-cssnext": {},  //该插件可以让我们使用CSS未来的特性，其会对这些特性做相关的兼容性处理。
    "postcss-px-to-viewport": {    //把px单位转换为vw、vh、vmin或者vmax这样的视窗单位，也是vw适配方案的核心插件之一。
     viewportWidth: 750,    //视窗的宽度
     viewportHeight: 1334,   //视窗的高度
     unitPrecision: 3,    //将px转化为视窗单位值的小数位数
     viewportUnit: 'vw',    //指定要转换成的视窗单位值
     selectorBlackList: ['.ignore', '.hairlines'],    //指定不转换视窗单位值得类，可以自定义，可以无限添加
     minPixelValue: 1,    //小于等于1px不转换为视窗单位
     mediaQuery: false   //允许在媒体查询中使用px
    },
    "postcss-viewport-units":{}, //给vw、vh、vmin和vmax做适配的操作,这是实现vw布局必不可少的一个插件
    "cssnano": {    //主要用来压缩和清理CSS代码。在Webpack中，cssnano和css-loader捆绑在一起，所以不需要自己加载它。
     preset: "advanced",   //重复调用
     autoprefixer: false,    //cssnext和cssnano都具有autoprefixer,事实上只需要一个，所以把默认的autoprefixer删除掉，然后把cssnano中的autoprefixer设置为false。
     "postcss-zindex": false   //只要启用了这个插件，z-index的值就会重置为1
    }
  }
}
```

### .eslintrc.js

eslint是用来管理和检测js代码风格的工具，可以和编辑器搭配使用，如vscode的eslint插件 当有不符合配置文件内容的代码出现就会报错或者警告。
[官方文档](https://zh-hans.eslint.org/docs/latest/use/configure/configuration-files)

一些配置项如下TODO:自己找项目测一下

rules 启用的规则及其各自的错误级别，详细可以参考 eslint规则。
parser 配置解释器，默认为 Espree，我们也可以配置 babel-eslint 作为解释器。
parserOptions 解析器选项，能帮助 ESLint 确定什么是解析错误。
env 指定想启用的环境，并设置它们为 true。
globals 定义全局变量，这样源文件中使用的全局变量就不会发出警告。
plugins 配置第三方插件，插件名称可以省略 eslint-plugin- 前缀。
settings 添加共享设置，提供给每一个将被执行的规则。
root 设置为 true，停止在父级目录中查找。
extends 可以从基础配置中继承已启用的规则。
overrides 可以实现精细的配置，比如，如果同一个目录下的文件需要有不同的配置。

```js
module.exports = {
    // 停止在父级目录中查找
    "root": true,
    // 启用一系列核心规则
    "extends": "eslint:recommended",
    // 启用的规则及其各自的错误级别
    "rules": {
        "semi": ["error", "always"],
        "quotes": "error",
        // 启用插件专用的规则
        "example-plugin/eqeqeq": "off"
    },
    // 解析器选项
    "parserOptions": {
        // ECMAScript 版本
        "ecmaVersion": 6,
        // 设置为 "script" (默认) 或 "module"（如果你的代码是 ECMAScript 模块)
        "sourceType": "module",
        // 使用的额外的语言特性
        "ecmaFeatures": {
            // 启用 JSX
            "jsx": true
        }
    },
    // 一个对Babel解析器的包装，使其能够与 ESLint 兼容
    "parser": "babel-eslint",
    // 全局变量
    "globals": {
        "document": false,
        "window": false,
        "HTMLInputElement": false,
        "HTMLDivElement": false,
        "localStorage": true
    },
    // 指定想启用的环境
    "env": {
        "browser": true,
        "commonjs": true,
        "es6": true,
        "jest": true,
        "node": true,
        // 指定插件专用环境
        "example-plugin/browser": true
    },
    // 配置插件
    "plugins": [
        "example-plugin"
    ],
    // 匹配特定的 glob 模式的文件
    overrides: {
        // 匹配所有.ts .tsx 文件
        files: ['**/*.ts', '**/*.tsx'],
        // 指定解析器
        parser: '@typescript-eslint/parser',
        // 指定解析器选项
        parserOptions: {
          ecmaVersion: 2018,
          sourceType: 'module',
          ecmaFeatures: {
            jsx: true,
          }
        }
    }
}
```

参考文档:
[使用Eslint和Editorconfig规范代码](https://www.jianshu.com/p/6611eb8fa9ff)

#### .editorconfig

editorconfig 保证了我们在不同的环境，使用不用的编辑器都有统一的规范。比如在 window 上默认换行使用 CRLF, 而在 mac 上的换行风格是 IF ; 有的编辑器默认缩进使用 Tab, 而有的编辑器使用 Space 等。

vscode 环境需要安装 EditorConfig for VS Code 插件，并在根目录下增加配置文件 .editorconfig

```json
# 标记此配置文件为最顶层
root = true

# 匹配所有文件
[*]
# 缩进格式 空格
indent_style = space
# 缩进格式字符大小
indent_size = 2
# 换行格式
end_of_line = lf
# 字符编码
charset = utf-8
# 去除每行末尾多余空格
trim_trailing_whitespace = true
# 在代码末尾添加新行
insert_final_newline = true
```

### .babelrc/babel.config.js

TODO:BABEL
[babel官方文档](https://www.babeljs.cn/docs/)
 这个文件是用来设置转码的规则和插件的
[.babelrc配置详解](https://www.jianshu.com/p/0d608955979d?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
[Vue .babelrc配置](https://www.cnblogs.com/jisa/p/11819023.html)
[babel6 快速指南](https://blog.csdn.net/weixin_34072159/article/details/91429686)
[坐下来，聊聊babel](https://blog.csdn.net/keader01/article/details/120920813)
[前端系列——快速理解babel6配置过程](https://segmentfault.com/a/1190000012753134)
