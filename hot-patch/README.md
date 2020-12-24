# React Native 热更新方案

在 iOS 应用开发中，由于 Apple 严格的审核标准和低效率，iOS 应用的发版速度极慢，这对于大多数团队来说是不能接受的，所以热更新对于 iOS 应用来说就显得尤其重要。而就在前不久，苹果严禁 WaxPatch、JSPatch 等热修复框架，不过庆幸的是采用 Js 热更新的 React Native 似乎并可没有收到多大影响。

热更新作为 React Native 的优势之一，相信很多人在选择使用 React Native 来开发应用，也是因为 React Native 具有的热更新特性。在热更新方案中，比较出名的有微软的 [CodePush](https://github.com/microsoft/react-native-code-push)，[React Native 中文网的 pushy](https://pushy.reactnative.cn/)等。

## 热更新

React Native 的热更新更像原生 App 的版本更新，需要实现一套完整的代码发布、管理、下载更新流程；

### 部署原理

![热更新](https://www.pianshen.com/images/221/4263edf89398b0a32b52beab32cfda25.png)

### 服务器端

实现版本代码的管理，包括添加、发布、环境切换、回滚、删除等功能；

![rgcmWV.png](https://s3.ax1x.com/2020/12/24/rgcmWV.png)


### 客户端

用户每次打开 APP 客户端，在 APP 启动的过程中请求服务器获取最新 bundle 版本号，同时从本地获取当前已安装 APP 的 bundle 版本号，将两者进行对比，如若一致，则无需更新继续执行其他逻辑；如果不一致，从服务器获取当前最新版本的 bundle 包下载到本地。解压缩最新版本的 bundle 包（服务器端存放的 bundle 包一般均为压缩文件）。如果采用全量更新方法，解压缩之后的 bundle 文件可直接使用；如果使用的是差量更新方法，解压缩之后还需要与当前本地使用的 bundle 包进行合并，合并成功之后方可使用。

![热更新流程](https://upload-images.jianshu.io/upload_images/2262256-d7a4236da60fc89b.png)

### 示例

一个完整的热更新流程如下图所示：

![热更新总体流程](https://s3.ax1x.com/2020/12/24/rgr9HI.png)

## Pushy

[Pushy](<(https://pushy.reactnative.cn/)>)是由国内团队[reactnativecn](https://reactnative.cn/)开发的 react-native 热更新解决平台。

### 优势

- 命令行工具&网页双端管理，版本发布过程简单便捷，完全可以集成 CI。
- 基于 bsdiff 算法创建的超小更新包，通常版本迭代后在 1-10KB 之间，避免数百 KB 的流量消耗。
- 支持崩溃回滚，安全可靠。
- meta 信息及开放 API，提供更高扩展性。
- 跨越多个版本进行更新时，只需要下载一个更新包，不需要逐版本依次更新。

### 劣势

- 代码需要上传到 Pushy 平台
- 有付费策略

## CodePush

由微软提供的热更新解决方案。默认是接入微软自己的平台[appcenter](https://appcenter.ms)

### 优势

- 除了服务器端代码，其他工具均开源且支持接入私有服务器
- 支持私有服务器搭建[code-push-server](https://github.com/lisong/code-push-server)
- [code-push](https://github.com/microsoft/code-push)开源的代码提交插件，(3.0 版本以下支持提交热更新代码到私有服务器)
- [react-native-code-push](https://github.com/microsoft/react-native-code-push)开源的客户端热更新插件

### 劣势

- 最新的代码仅实现与微软自己的服务器对接
- 私有服务器源码为 nodejs 编写
- 私有服务器源码有 1 年左右未维护(跨版本维护、增量更新功能需要二次开发)

### 部署方案
- [服务器](./code-push-server.md)
- [发布端](./code-push.md)
- [客户端插件](./react-native-code-push.md)

## 参考
- [React Native使用Code Push热更新完整解决方案](https://wddsss.com/main/displayArticle/267)