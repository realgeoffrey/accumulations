# [webpack](https://github.com/webpack/webpack)学习笔记

## 目录
1. [总结](#总结)
1. [`webpack.config.js`](#webpackconfigjs)
1. [心得](#心得)
1. [Rollup与webpack对比](#rollup与webpack对比)

---
### 总结
>webpack: module building system.

1. 能使用各种方式表达依赖关系

    CommonJS、ES6 module、AMD、CSS的`@import`、样式的`url()`、HTML的`<img src="">`。都被转化为CommonJS规范的实现（各种方式引入效果相同）。

    >```javascript
    >import list from './list';
    >// 等价于：
    >var list = require('./list');
    >```
2. 所有文件都当作是**模块（module）**：`脚本（js、jsx、tsx、coffee）`、`样式（css、scss、sass、less）`、`模版（html、tpl）`、`JSON`、`图片`、`字体`。

    webpack自身只理解JS，使用`loader`把所有类型的文件（链式）转换为模块。
3. 从入口起点（`entry`）开始进入文件进行解析，递归地构建一个依赖图（dependency graph），这个依赖图包含着应用程序所需的每个模块，模块通过`加载器`解析完毕，之后经过`插件`二次处理，最终将所有这些模块打包为少量的bundle由浏览器加载。
4. 概念

    1. chunk

        （中间过程的）代码块，块的名字在`entry`设置，数字是id。

        >每次修改一个模块时，webpack会生成两部分：`manifest.json`（新的编译hash和所有的待更新chunks目录）；更新后的chunks（.js）。
    2. vendor

        >使用`SplitChunksPlugin`进行自动化选择某些资源避免重复依赖。

        独立于经常改动的业务代码，额外提取出chunk成为单独的bundle用于用户缓存。每次构建时都需要进行构建。
    3. dll

        >`DLLPlugin`和`DLLReferencePlugin`配合设置动态链接库（还需要在html中插入），指定资源避免重复依赖。

        独立于经常改动的业务代码，额外提取出chunk成为单独的bundle用于用户缓存（只有修改了dll引用的内容才需再次构建dll）。

        ><details>
        ><summary>dll作为所有入口的共用内容，也可作为「桥」的作用</summary>
        >
        >存放入口之间连接交互的代码：引入「桥」后对其设置对象属性，这样所有入口引用「桥」后都能获得对象属性，同时入口之间更新代码不互相影响。
        >>e.g. 针对特殊的需求：一个项目中拆分3个JS，第一个dll（包括「桥」），第二、三个为入口。第二入口向「桥」添加属性A，第三入口就能通过「桥」引用获得属性A，并且入口A（或B）更新代码不影响入口B（或A）的代码。
        ></details>

    >vendor和dll二选一即可。

    4. bundle

        多个chunk的最终合并产出物。

### `webpack.config.js`
1. 关键点
    1. `entry`：定义整个编译过程的起点
    2. `output`：定义整个编译过程的终点
    3. `module`：加载器定义模块（文件）的处理方式

        >`use`有顺序之分，数组逆序执行。
    4. `plugin`：插件对编译完成后的内容（加载器处理之后）进行二度加工

        >每个plugin都会绑定各种webpack生命周期钩子进行执行plugin内某些具体内容，用户一般不用关注各plugins之间执行顺序。
    5. `resolve.alias`：定义路径的别名
2. `webpack-dev-server`的热更新配置：`devServer.hot: true`（需要插件`new webpack.HotModuleReplacementPlugin()`配合）

    利用websocket实现，websocket-server识别到html、css和js的改变，就向websocket-client发送一个消息，websocket-client判断若是html和css则操作dom，实现局部刷新，若是js则整体刷新。
3. 插件`DefinePlugin`在构建结束后向项目代码中注入变量：`new webpack.DefinePlugin({键-值})`

    若项目代码中要使用的Node.js的环境变量，建议都用此方式注入后再使用，而不要直接使用由webpack额外处理的Node.js环境变量。
4. 可以导出数组，分别进行配置，串行执行多个webpack任务（如：前后端同构任务）

    ```javascript
    module.exports = [配置1, 配置2]
    ```

### 心得
1. 引用仓库

    1. 若引用仓库，则会按照仓库设置好的路径引用仓库文件。
    2. 若引用仓库中某文件，则会按照该文件的引用链路去引用仓库文件。
    3. 仓库中没有被引用到的文件不会打包进最终bundle。

    >e.g. 可以用`import debounce from 'lodash/debounce'`代替`import { debounce } from 'lodash'`，这样最终打包的结果不会引用整个lodash，而只会引用debounce的引用链路文件（可以用[webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)分析并可视化构建后的打包文件进行对比）。

### [Rollup](https://github.com/rollup/rollup)与webpack对比
1. Rollup：

    1. 文件很小，几乎没多余代码；执行很快；可方便输出CommonJS、ES6 Module、IIFE（用于`<script>`引用）格式。
    2. 必须用ES6 Module格式的代码才可以打包。
2. webpack：

    1. 拥有强大、全面的功能，更好的社区。
    2. 在进行资源打包时会产生很多冗余的代码。

- 总结：App级别的应用——webpack，JS库级别的应用——Rollup。
