# React Native 拆包方案

RN 作为一款非常优秀的移动端跨平台开发框架，在近几年得到众多开发者的认可。纵观现在接入 RN 的大厂，如 qq 音乐、菜鸟、去哪儿，无疑不是将 RN 作为重点技术栈进行研发。

不过，熟悉 RN 的开发者也知道，早期的 RN 版本中打出来的包都只有一个 jsbundle，而这个 jsbundle 里面包含了所有代码（RN 源码、第三方库代码和自己的业务代码）。如果是纯 RN 代码倒没什么关系，但大部分的大厂都是在原生应用内接入 RN 的，而且一个 RN 中又包含许多不同的业务，这些不同的业务很可能是不同部门开发的，这样一个库中就有许许多多的重复的 RN 代码和第三方库代码。

所以，一般做法都是将重复的 RN 代码和第三方库打包成一个基础包，然后各个业务在基础包的基础上进行开发，这样做的好处是可以降低对内存的占用，减少加载时间，减少热更新时流量带宽等，在优化方面起到了非常大的作用。

## 几种拆包方案

### moles-packer

[moles-packer](https://github.com/ctripcorp/moles-packer) 是由携程框架团队研发的，与携程 moles 框架配套使用的 React Native 打包和拆包工具，同时支持原生的 React Native 项目。

特点：重写了 react native 自带的打包工具，适合 RN0.4.0 版本之前的分包。维护少，现在基本没有多少人使用，兼容性差。

### diff patch

diff patch 大致的做法就是先打个正常的完整的 jsbundle，然后再打个只包含了基础引用的基础包，比对一下 patch，得出业务包，这样基础包和业务包都有了，更新时更新业务包即可。差分包的工具可以 [google-diff-match-patch](http://google.com/p/google-diff-match-patch)

### metro bundle

目前，最好的 RN 分包方案还是 facebook 官方提供的 metro bundle，此方案是 fb 在 0.50 版本引入的，并随着 RN 版本的迭代不断完善。也即是说，只要你使用的是 0.50 以上的 RN 版本，就可以使用 [metro bundle](https://facebook.github.io/metro/docs/en/getting-started) 进行差分包进行热更新。

[查看详细方案](./metro-bundle.md)
