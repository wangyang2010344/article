# 搭建mve的webpack开发环境

因为目前mve的两个包：mve-core/mve-dom是纯粹的npm包，使用任何模块打包器都应该能很好支持。这里使用webpack为示例，主要是webpack目前稍主流，解决方案多。我本人却并不是特别熟悉webpack，勉强用其搭出了mve的typescript开发环境以为示例，有不足处请见谅。



## 搭建webpack开发环境

按照官方文档操作即可



## 配置webpack的typescript开发环境

这里也根据官方文档配置ts-loader即可。

这里我遇到的问题：因为mve-core/mve-dom是同时上传了ts/d.ts/js的，所以resolve的extensions里js放在ts之前。

## 加载mve-core/mve-dom

直接npm install mve-dom --save即可。

mve-dom依赖了mve-core。

然后可以愉快地开发了。