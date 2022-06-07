## webpack

### webpack是什么
从功能上看，webpack是前端打包工具，输入文件路径，通过loader加载和plugin解析后输出产后处理后的文件，通过`loader`我们可以将高版本es转换为低版本es, 预处理css转换为标准css, 从而实现使用模块化，es6+等新特性开发，而不用考率宿主环境无法匹配等问题，因为最终都可与会转化为低版本，提高了代码的编写效率和可维护性；通过`plugin`可以对输出文件进行合并，拆分，树摇，公共部分提取等手段合理的优化资源体积和数量，对于加载性能有极大地提升；同时devServer也为我们当前这种`前后端分离`中的前端开发解决跨域问题; `热更新机制`也让我们更新静态资源不必刷新浏览器，只更新对应模块部分，这对我们开发表单等需要向页面输入值得场景帮助巨大，不需要在刷新页面重现填写所有内容

### webpack原理
1. 本质上webpack是一个基于事件流的编程范式，内部定义了一系列hooks，loader和plugin通过订阅这些回调并作出相应的处理
2. 这些hooks大致可以分为一下9个阶段:
    | 阶段 | 说明 |
    | -- | -- |
    | enter-option | 初始化option |
    | run | 初始化option |
    | make | 从entery开始分析依赖，对每个依赖模块进行build |
    | before-resolve | 堆模块位进行解析 |
    | build-module | 开始构建某个模块 |
    | normal-module-loader | 将loader加载完成的模块进行编译，生成AST树 |
    | program | 遍历AST，当遇到require等表达式时，收集依赖 |
    | seal | 所有依赖build完毕，开始优化 |
    | emit | 输出到dist目录 |

### webpack构建速度和体积优化
1. 分别使用speed-measure-webpack-plugin分析速度，webpack-bundle-analyzer分析体积
2. 多实例/进程构建，thread-loader, hapypack, parallel-webpack
3. 多进程并行压缩代码 `prallel-uglify-plugin`, `uglify-js-plugin`, `terser-webpack-plugin`, 设置`parllel`参数开启
4. 进一步分包
    | 措施 | 思路 | 方法 |
    | -- | -- | -- |
    | 设置Externals | 分离基础包，不做解析 | 可以通过`HtmlWebpackExternalsPlugin`设置cdn等方式记载 |
    | 预编译资源模块 | 将基础包打包成一个文件 | 使用DLLPlugin进行分包，使用DLLReferencePlugind对manifest.json引用 |
    | 利用缓存提升二次构件速度 | 开启loader，plugin的缓存 | babel-loader, terser-webpack-plugin开启缓存，使用cache-loader或者hard-source-webpack-plugin |
    | 缩小构建目标 | 不解析第三方包 |  |
    | treeShake | 擦除未使用的js, css | es6模块webpack默认支持，css使用purifyCss/uncss |
    | 图片压缩 |  | image-webpack-loader |
    | 使用动态PolyFill | 根据当前浏览器仅返回浏览器需要的PolyFill | 搭建动态Polyfill服务 |