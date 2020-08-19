[返回首页](../index.md)

# R package 的离线环境迁移

为完成R的离线迁移，首先要准备一个有网的环境，并尽量保证进行迁移的两个环境保持一致（至少保证同一平台，WIN、LINUX、MAC OS）
在有网的环境中开启R，运行如下代码

```R
library(tools)

# 列出所有需要用到的库
target_pkgs <- c('pkg1, 'pkg2')
# 递归查询所有上述需要的库的依赖库，并和target_pkgs何在一起，作为需要下载的依赖包列表
all_pkgs <- c(target_pkgs, unique(stack(package_dependencies(target_pkgs, recursive = T))$values))
# 下载所有依赖包
download.packages(all_pkgs, 'path/to/download', type = 'binary')
```

需要注意的是，以上代码下载的依赖包都是二进制包，这是因为实际使用中发现用源码包进行编译安装时有部分包需要额外进行网络请求才可以完成编译。
所以保证进行迁移的环境之间保持一致十分重要。

下载完成后，把下载好的安装包文件传入到需要迁移的系统，然后在新环境的R中执行如下代码

```R
install.packages(list.files('path/to/download', full.names = T), repos = NULL, type = "binary")
```

待所有安装执行完毕，用 `library(pkg_name)` 验证所需依赖包是否安装成功
 
---------------------------------------------------------------
2020-02-26
