# Metro bundle 方案

自 2015 年 Facebook 发布 React Native 以来，目前已经是跨平台开发的主流产品，但是随着业务复杂度越来越高，单一的 JSBundle 越来越大，已经不能满足用户需求，需要拆分业务包&基础包。

## React Native 打包过程分析

RN 官方从 0.46.0 版本开始，出于对打包速度、可靠性和更好的可拓展性等方面的考虑，将 RN 的打包模块 packager 从 RN 项目中分离，重新命名为 Metro Bundler。
![拆包流程](https://s3.ax1x.com/2020/12/30/rLRQL8.jpg)

**JSBundle 命令**

```shell
react-native bundle --entry-file index.js --platform ios --dev false --bundle-output ./index.ios.bundle --sourcemap-output ./index.ios.map --assets-dest ./asserts
```

- entry-file：即入口文件，打包时以该文件作为入口，一步步进行模块分析处理。
- platform：用于区分打包什么平台的 bundle (ios or android, 默认 ios)
- dev：用于区分 bundle 使用环境，非 dev 时，会对代码进行 minified
- bundle-output：打包产物输出地址，即打包好的 bundle 存放地址
- sourcemap-output：打包时生成对应的 sourcemap 文件存放地址，在跟踪查找错误或崩溃时，能帮助开发快速定位到代码
- assets-dest：bundle 中使用的静态资源文件存放地址
- --verbose: 打印详情
- --reset-cache: 清除缓存文件
- --config: CLI 文件路径

**Polyfill**

Metro Bundler 在进入正式拆包前，先在 js 解释器上挂载 global.DEV 变量，该变量主要用于区分打包执行环境，同时定义了模块系统函数，包括 \_\_d（即 define）函数、require 函数，这样 RN 也就有了自身的模块系统。另外，RN 对部分 es6、es7 的方法 Polyfill 支持，包括 Array、Object 和 Number 等。

## React Native JSBundle 结构

以最基础的 RN 项目的 Bundle 为例，可以看到 Bundle 文件中大致定义了四个模块：

- var 声明的变量，对当前运行环境的定义，Bundle 的启动时间、Process 进程环境相关信息
- (function() { })() 闭包中定义的代码块，其中定义了对 define（\_\_d）、 require（\_\_r）、clear（\_\_c） 的支持，以及 module（React Native 及第三方 dependences 依赖的 module） 的加载逻辑
- \_\_d 定义的代码块，包括 RN 框架源码 js 部分、自定义 js 代码部分、图片资源信息，供 require 引入使用
- \_\_r 定义的代码块，找到 \_\_d 定义的代码块 并执行

总结以上，Bundle 文件大致包含三部分：
![bundle文件结构](https://s3.ax1x.com/2020/12/30/rLRwLT.jpg)

- Polyfills：最先执行的一些 function，ES6 特性支持，定义模块声明方法等
- modules：模块声明，以 \_\_d 开头，定义各式各样的 module，其中包括了 React Native 的 module（StatusBar、View、Text ...），引入的第三方 module 等
- require：执行 InitializeCore 和 Entry File，最后一行执行 require(0)
  \_\_d 的执行对应于 nodule_modules / metro / lib / polyfills / require.js 文件中的 define 方法
  require 的执行对应于 nodule_modules / metro / lib / polyfills / require.js 文件中的 metroRequire 方法

## React Native 拆包

了解了 JSBundle 的打包过程以及 JSBundle 的结构，下面我们讨论一下拆包。

随着业务不断增加，模块化自然是首要考虑的问题。
![多业务模式](https://s3.ax1x.com/2020/12/30/rLR6Y9.jpg)

然后业务 1，2，3 分别用 RN 容器加载,但是这样的结构有显而易见两个问题：

- 每个业务界面打开是会有明显的白屏
- 每个业务中 jsbundle 会有很多重复模块

根据上面 JSBundle 的结构, 可以将 JSBundle 拆分成以下模块：
![拆包后结构](https://s3.ax1x.com/2020/12/30/rLRfOK.jpg)

- base module，是可复用基础包，里面包括 React Native 模块，view, text, button 和一些可复用的三方模块。
- 业务 1，2，3 module，是纯 js 业务代码，可动态加载。

## 多 JSBundle 打包

目前推荐官方提供[Metro Bundle](<(https://facebook.github.io/metro/docs/en/configuration)>)打包服务基础上做拆包，如果团队人员充沛，可考虑自研打包服务。

主要是序列化的时候：

- 配置 createModuleIdFactory 函数生成自定义的 module id, 生成 module id 可以用文件绝对路径，也可以做一次 hash 或者 md5，保证唯一。
- 因为 metro 打包时，会以递增的方式给每个模块分配一个 ID，在文件调用时直接调用对应的 ID 号。在拆分 bundle 后，如果我们的基础包有依赖模块的变动，整个模块调用的 3, ID 都会错位。所以要采用更加稳定健壮的 ID 生成方式。
- 配置 processModuleFilter 过滤包，函数传入的是 module 信息，返回是 boolean 值，如果是 false 就过滤不打包, 例如业务不需要打 node_module 中的包。
- 静态资源处理。因为打包生成的静态资源根目录是固定的 assets，为了方便灵活组织资源内容，我们添加对自定义静态资源根目录的功能支持。
- 多 bundle 静态资源最终会合并到一个目录下去，这是最节约资源的一种结构，防止重复资源出现。

```
|- assets/
    |- base/
    |- node_modules/
    |- bundle1_resource/
    |- bundle2_resource/
```

推荐第三方解决方案[react-native-multibundler](https://github.com/smallnew/react-native-multibundler)

## 多业务打包部署

![自动化部署](https://s3.ax1x.com/2020/12/30/rLRTFH.jpg)

每个业务包，都包含自己的基础依赖 base.js：

1. 业务(1，2，3)发布一个版本，push tag 触发 CI server；
2. CI 触发 WBRNpackage 服务；
3. WBRNpackage 服务根据 base.js 合成公用的 base.js；
4. 分别打出对应业务的 JSBundle, 基础 base.jsbundle, 并合成静态资源文件；

## 总结

截止目前我们解决了几个问题：

- 模块业务分离，每个业务是一个单独的包
- 由于之前加载了基础包，原生容器打开对应的业务包，可以达到秒开的效果
- 在多容器的情况下，共用一个基础包，防止 bundle 资源重复
- 利用 CI 自动化自动部署 JSBunlde 和静态资源

## 问题

1. 如果业务 A 需要引用业务 B？
   需要把业务 B 引用的模块注册到业务 A 中，或基础模块中，目前无法解决；

2. 如果业务 A 需要集成原生代码逻辑，例如地图、拍照等？
   原生模块中的 js 代码，必须加入到基础模块中；原生部分的代码必须安装新版 App 才能支持；

3. 

## 参考

[React Native 分包哪家强？看这文就够了](https://blog.csdn.net/csdnsevenn/article/details/86513931)
