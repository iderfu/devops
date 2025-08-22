> 转载自： https://www.jianshu.com/p/bbed916d16ea

如果你还在使用 GOPATH 模式来开发Golang程序，那么你可以参考本文来告别 GOPATH，并带给你一个方便的包管理工具。

关于 go mod 的说明和简单使用，可以参考：

1、[Go1.1.1新功能module的介绍及使用](https://blog.csdn.net/benben_2015/article/details/82227338)

2、[Introduction to Go Modules](https://roberto.selbach.ca/intro-to-go-modules/)

3、[Go 1.11 Modules 官方说明文档](https://github.com/golang/go/wiki/Modules)

go mod 就是go中的maven，官方包依赖管理工具

通过go mod help查看相关命令：

```
go mod download 下载模块到本地缓存，缓存路径是 $GOPATH/pkg/mod/cache
go mod edit 是提供了命令版编辑 go.mod 的功能，例如 go mod edit -fmt go.mod 会格式化 go.mod
go mod graph 把模块之间的依赖图显示出来
go mod init 初始化模块（例如把原本dep管理的依赖关系转换过来）
go mod tidy 增加缺失的包，移除没用的包
go mod vendor 把依赖拷贝到 vendor/ 目录下
go mod verify 确认依赖关系
go mod why 解释为什么需要包和模块
```

 



使用go mod 管理项目，就不需要非得把项目放到GOPATH指定目录下，你可以在你磁盘的任何位置新建一个项目，比如：

新建一个名为 wserver 的项目，项目路径 D:\test\wserver （注意，该路径并不在GOPATH里）

![img](https://gitee.com/clay-wangzhi/blogImg/raw/master/blogImg/15777556-ab8969798c924c1e.webp)

进入项目目录 D:\test\wserver 里，新建一个 go源码文件： main.go

![img](https://gitee.com/clay-wangzhi/blogImg/raw/master/blogImg/15777556-402ecc223228de7b.webp)

然后在 D:\test\wserver 里打开终端执行命令： **go mod init wserver** （go mod init 后面需要跟一个名字，我这里叫wserver）

![img](https://gitee.com/clay-wangzhi/blogImg/raw/master/blogImg/15777556-00170c75adbdd1ab.webp)

看到提示 “go: creating new go.mod: module wserver” 说明 go mod 初始化成功了，会在当前目录下生成一个 go.mod 文件。

包含go.mod文件的目录也被称为模块根，也就是说，go.mod 文件的出现定义了它所在的目录为一个模块。

执行上述命令之后，其实你已经可以开发编译运行此项目了，比如我们随便使用github上的一个包，在终端打印一下

![img](https://gitee.com/clay-wangzhi/blogImg/raw/master/blogImg/15777556-a4810728fc350448-1594804465517.webp)

运行一下，会看到输出结果： { false false false} ，同时项目目录下多出了一个文件 go.sum 。go.sum 是记录所依赖的项目的版本的锁定。

![img](https://gitee.com/clay-wangzhi/blogImg/raw/master/blogImg/15777556-058c3b86e859af9e.webp)

在 main.go 里如果需要使用这个包，需要使用这个包的 模块内的绝对路径来导入，比如：

![img](https://gitee.com/clay-wangzhi/blogImg/raw/master/blogImg/15777556-fc02bb753612d905.webp)

"wserver/route" 导入这个包的地方是 模块内的绝对路径，就是要从go.mod所在的目录开始。

另外，如果我们想把这个项目放到GOPATH下面，不使用go mod模式，而是想使用GOPATH模式的话，，只需要把这个项目移到GOPATH环境变量包含的任意一个目录下面的src目录里，就可以启用GOPATH模式了（前提是 GO111MODULE 这个环境变量的值必须是auto 或 off）。

比如：gotest 目录是GOPATH环境变量里的其中一个目录，我们将上面的代码复制到 src 目录下，删除原来的go.mod 、go.sum 两个文件（也可以不删除），代码一样可以运行：

![img](https://gitee.com/clay-wangzhi/blogImg/raw/master/blogImg/15777556-982bd5a0f9382128.webp)

**小总结：**

使用go mod ，利用Go 的 module 特性，你再也不需要关心GOPATH了（当然GOPATH变量还是要存在的，但只需要指定一个目录，而且以后就不用我们关心了）， 你可以任性的在你的硬盘任何位置新建一个Golang项目了。

好了，本文就是个非常简单的小示例，只是告诉初次使用go mod的人，如何快速开始使用go mod 。详细的概念和使用方法，请仔细阅读开头列出的3篇文章。



> go mod init有时会报错：go: cannot determine module path for source directory .......
> xxxxxxx\example (outside GOPATH, no import comments)
> 执行时加上module名称即可：go mod init example