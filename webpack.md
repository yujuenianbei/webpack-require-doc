# webpack

## 基本概念

1. 入口(entry)
2. 输出(output)
3. loader
4. 插件(plugins)

### 入口[entry]
入口起点指示webpack应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后webpack会找出有哪些模块和库是入口起点（直接和间接）依赖的　　
可通过在webpack配置中配置entry属性，来指定一个入口起点（或多个入口起点）。默认为./src

    module.export = {
        entry: './path/entry/file.js'
    }

### 出口[output]
output属性告诉webpack在哪里输出它所创造的bundles，以及如何命名这些文件，默认值为　./dist。　基本上整个应用程序结构都会被编译到你指定的输出路径的文件夹中

    const path = require('path');

    module.exports = {
        entry: './path/entry/file.js',
        output: {
            path: path.resolve(__dirname, 'dist'),
            filename: 'my-first-webpack.bundle.js'
        }
    }

上面例子中，通过output.filename和output.path属性，告诉webpack.bundle的名称，以及bundle生成(emit)到哪里。path为node核心模块

### loader
loader让webpack能够去处理那些非js文件(webpack自身只理解js)，loader可以将所有类型的文件转化为webpack能够处理的有效模块，然后用webpack打包对他们进行处理，本质上webpack loader将所有类型的文件，转换为应用程序的依赖图（或最终的bundle）可以直接引用的模块.  
注意，loader 能够 import 导入任何类型的模块（例如 .css 文件），这是 webpack 特有的功能，其他打包程序或任务执行器的可能并不支持。我们认为这种语言扩展是有很必要的，因为这可以使开发人员创建出更准确的依赖关系图

更高层面上，在webpack的配置loader中有两个目标:

#### test属性，用于标识出对应的loader进行转换的某个或某些文件

#### use 属性，表示进行转换时，应该使用哪个loader  

    const path = require('path');
    const config = {
    output: {
        filename: 'my-first-webpack.bundle.js'
    },
    module: {
        rules: [
        { test: /\.txt$/, use: 'raw-loader' }
        ]
    }
    };
    module.exports = config;


以上配置中，对一个单独的module对象定义了rules属性，里面包含两个必须属性：　test和use　告诉webpack编译器如下信息:  
“嘿，webpack 编译器，当你碰到「在 require()/import 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先使用 raw-loader 转换一下。”  
注: 在 webpack 配置中定义 loader 时，要定义在 module.rules 中，而不是 rules。然而，在定义错误时 webpack 会给出严重的警告。为了使你受益于此，如果没有按照正确方式去做，webpack 会“给出严重的警告”

### 插件
loader被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。可以处理各种各样的任务。  
插件的使用: 
1. require() 这个插件
2. 把这个插件添加到plugins数组中

多数插件可以通过option自定义。也可以在一个配置文件中因为不同目的而多次使用同一个插件，此时需要使用new操作符来创建它的一个实例

    const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
    const webpack = require('webpack'); // 用于访问内置插件

    const config = {
    module: {
        rules: [
        { test: /\.txt$/, use: 'raw-loader' }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({template: './src/index.html'})
    ]
    };

    module.exports = config;

可访问<a href="https://www.webpackjs.com/plugins/">webpack插件库</a>获取需要使用的插件

### 模式
通过选择 development 或 production 之中的一个，来设置 mode 参数，你可以启用相应模式下的 webpack 内置的优化

    module.exports = {
        mode: 'production'
    };


## 入口起点[entry points]
### 单个入口（简写）语法
用法：entry: string|Array(string)

    const config = {
        entry: './path/to/my/entry/file.js'
    };

    module.exports = config;

entry属性的单个入口语法是下面的简写

    const config = {
        entry: {
            main: './path/to/my/entry/file.js'
        }
    }

当你向 entry 传入一个数组时会发生什么？向 entry 属性传入「文件路径(file path)数组」将创建“多个主入口(multi-main entry)”。在你想要多个依赖文件一起注入，并且将它们的依赖导向(graph)到一个“chunk”时，传入数组的方式就很有用

当你正在寻找为「只有一个入口起点的应用程序或工具（即 library）」快速设置 webpack 配置的时候，这会是个很不错的选择。然而，使用此语法在扩展配置时有失灵活性。

### 对象语法
用法：entry:([entryChunkName: string]: string[Array(string)])

    const config = {
        entry:{
            app: './src/app.js',
            vendors: './src/vendors.js'
        }
    }
对象语法会比较繁琐。然而这是应用程序中定义入口的最可扩展的方式

代码解释：  
这是什么？  
从表面上看，这告诉我们webpack从app.js和venders.js开始创建依赖图(dependency graph)。这些依赖图是彼此完全分离，相互独立的（每个bundle中都有一个webpack引导(bootstrap)）。这种方式比较常见于只有一个入口起点（不包括vender）的单页应用

为什么？此设置允许你使用commonChunkPlugin从应用程序bundle中提取vendor引用到vendor bundle,并把引用vendor的部分替换为__webpack_require__()调用。如果应用程序bundle中没vendor代码，那么可以在webpack中实现被称为长效缓存的通用模式

### 多页面应用

    const config = {
        entry: {
            pageOne: './src/pageOne/index.js',
            pageTwo: './src/pageTwo/index.js',
            pageThree: './src/pageThree/index.js'
        }
    }
这是什么？我们告诉 webpack 需要 3 个独立分离的依赖图（如上面的示例）。  

为什么？在多页应用中，（译注：每当页面跳转时）服务器将为你获取一个新的 HTML 文档。页面重新加载新文档，并且资源被重新下载。然而，这给了我们特殊的机会去做很多事：  

使用 CommonsChunkPlugin 为每个页面间的应用程序共享代码创建 bundle。由于入口起点增多，多页应用能够复用入口起点之间的大量代码/模块，从而可以极大地从这些技术中受益。

## 输出[output]
配置output选项可以控制webpack如何向硬盘写入编译文件。
注：即使可以存在多个入口起点，但是只指定一个输出配置

用法:
webpack配置output属性最低要求是:将它的值设置为一个对象，包含以下两点：  
1 filename　用于输出文件的文件名   
2 目标输出目录path的绝对路径:    

    const config = {
        output: {
            filename: 'bundle.js',
            path: './home/public/assets'
        }
    };
    module.exports = config;

此配置将一个单独的 bundle.js 文件输出到 /home/proj/public/assets 目录中。

### 多个入口起点
如果配置创建了多个单独的‘chunk’(例如，使用多个入口起点或使用像CommonsChunkPlugin这样的插件)，则应该使用占位符来确保每个文件具有唯一的名称

    {
        entry:{
            app: './src/app.js',
            serach: './src/search.js'
        },
        output:{
            filename: '[name].js',
            path：　__dirname + '/dist'
        }
    }
    // 写入到硬盘：./dist/app.js, ./dist/search.js

### 高级进阶

使用cdn和资源hash

    output: {
        path: "/home/proj/cdn/assets/[hash]",
        publicPath: "http://cdn.example.com/asserts/[hash]"
    }
编译时不知道最终输出文件的publicPath的情况下pulicPath可以留空，并且在入口起点文件运行时动态设置。如果不知道pulicPath可以先忽略它，并且在入口起点设置　__webpack_public_path__。　

    __webpack_public_path__ = myRuntimePub显式
    // 剩余的应用程序入口

## 模式[mode]
提供mode配置选项，告知webpack使用相应模式的内置优化。
string

详情参见<a href="https://www.webpackjs.com/concepts/mode/">webpack模式</a>

## loader
loader用于对模块的源代码进行转换。loader可以使你在import或“加载”模块时预处理文件。因此loader类似于其他构建工具中“任务(task)”，并提供了处理前段构建步骤的强大方法。loader可以将文件从不同的语言(ts)转换为js，或将内联图像转换为data URL。loader甚至允许直接在js中import CSSwenjian 

使用方法:  
加载css,ts转为js
1. 使用npm安装对应的loader

    npm install --save-dev css-loader  
    npm install --save-dev ts-loader

2. 然后指示 webpack 对每个 .css 使用 css-loader，以及对所有 .ts 文件使用 ts-loader：

webpack.config.js

    module.exports = {
        module: {
            rules: [
                {test:/\.css$/, use: 'css-loader'},
                {test:/\.ts$/, use:‘ts-loader’}
            ]
        }
    }
使用loader
1. 配置（推荐）：在webpack.config.js文件中指定loder
2. 内联：在每个import语句中显式指定loader
3. CLI：在shell命令中指定他们

### 插件[configuration]
module.rules允许在webpack配置中指定多个loader。此方法有助于使代码变得简洁，同时对各个loader有个全局概览

    module:{
        rules: [
            {
                test: /\.css$/,
                use:[
                    {loader:'style-loader'},
                    {
                        loader:'css-loader',
                        options:{
                            modules: true
                        }
                    }
                ]
            }
        ]
    }
### 内联
可以在import语句或任何等效于import的方式中指定loader。使用 ! 将资源中的loader分开。分开的部分相当与当前目录解析

    import Styles from 'style-loader!css-loader?modules!./styles.css';
通过前置所有规则及使用　! ,可以对应覆盖到配置中的任意loader。  
选项可以传递查询参数，例如?key=value&foo=bar,或者一个JSON对象，例如?{"key":"value","foo":"bar"}。

### CLI
可以通过使用CLI使用loader

    webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'
这会对 .jade 文件使用 jade-loader，对 .css 文件使用 style-loader 和 css-loader。

### loader 特性
loader 支持链式传递。能够对资源使用流水线(pipeline)。一组链式的 loader 将按照相反的顺序执行。loader 链中的第一个 loader 返回值给下一个 loader。在最后一个 loader，返回 webpack 所预期的 JavaScript。  
loader 可以是同步的，也可以是异步的。  
loader 运行在 Node.js 中，并且能够执行任何可能的操作。  
loader 接收查询参数。用于对 loader 传递配置。  
loader 也能够使用 options 对象进行配置。  
除了使用 package.json 常见的 main 属性，还可以将普通的 npm 模块导出为 loader，做法是在 package.json 里定义一个 loader 字段。
插件(plugin)可以为 loader 带来更多特性。  
loader 能够产生额外的任意文件。  
loader 通过（loader）预处理函数，为 JavaScript 生态系统提供了更多能力。 用户现在可以更加灵活地引入细粒度逻辑，例如压缩、打包、语言翻译和其他更多。

## 解析loader
loader遵循标准模块解析（同node）


## 插件[plugin]
插件目的在于解决loader无法实现的其他事

代码解析：  
webpack插件是一个具有apply属性的js对象。apply属性会被webpack complier调用，并且complier对象可在整个编译生命周期访问。  
ConsoleLogOnBuildWebpackPlugin.js

    const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

    class ConsoleLogOnBuildWebpackPlugin {
        apply(compiler) {
            compiler.hooks.run.tap(pluginName, compilation => {
                console.log("webpack 构建过程开始！");
            });
        }
    }

用法：  
由于插件可以携带参数/选项，必须在webpack配置中，必须向plugin属性传入new实例。

### 配置方法

    const HtmlWebpackPlugin = require('html-webpack-plugin');
    const webpack = require('webpack');
    const path = require('path');

    const config = {
        entry: './path/to/my/entry/file.js',
        output: {
            filename: 'webpack-bundle.js',
            path: path.resolve(__dirname, 'dist');
        },
        module: {
            rules:[
                {
                    test:/\.(js|jsx)$/,
                    use: 'babel-loader'
                }
            ]
        },
        plugins:[
            new webpack.optimize.UglifyJsPlugin(),
            new HtmlWebpackPlugin({template: './src/index.html'})
        ]
    };
    module.exports = config;

## 配置
webpack的配置文件，是导出一个对象的JS文件。此对象，由webpack根据对象定义的属性进行解析  
因为webpack配置是标准的Node.jsCommonJs模块可以做以下事情：
1. 通过require(...) 导入其他文件
2. 通过rquire(...)使用npm的工具函数
3. 使用js控制流表达式　例如?:操作符
4. 对常用值使用常亮或变量
5. 编写并执行函数来生成部分配置

此部分参考<a href="https://www.webpackjs.com/configuration/">官网配置专项</a>

## 模块
webpack模块：

1. ES2015 import 语句
2. CommonJS require() 语句
3. AMD define 和 require 语句
4. css/sass/less 文件中的 @import 语句。
5. 样式(url(...))或 HTML 文件(<img src=...>)中的图片链接(image url)

通过使用loader支持各种语言和预处理器编写模块

1. CoffeeScript
2. TypeScript
3. ESNext (Babel)
4. Sass
5. Less
6. Stylus

