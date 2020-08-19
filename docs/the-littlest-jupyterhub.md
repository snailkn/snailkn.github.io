[返回首页](../index.md)

# 用The-littlest-jupyterhub搭建线上jupyter服务器

jupyter（notebook、lab）是当今流行的数据分析软件，它与其他编程IDE最大的不同之处在于可以存储代码运行的中间状态，以方便代码调试，同时它可以和`markdown`语句混合编排，成为一个实验笔记本，这些特性使得它特别适合作为学习或者探索的工具。

另一方面，安装各种依赖环境、更改各种配置对初学者而言宛如噩梦，同时也会把自己的系统搞得杂乱不堪。一个良好的解决方案是在服务器上开启jupyter服务，同时在客户端通过浏览器访问，这特别适合例如代码培训这样希望大量初学者获得实践机会的场景。`jupyterhub`给了我们这样的解决方案，本文旨在记录我们借助`the-littlest-jupyterhub`进行jupyter server搭建的过程，以供参考。

## 服务端安装jupyterhub

本文参照官方文档基于The-littlest-jupyterhub进行安装的部分，这样做比起一般常规方式安装的好处在于系统会自动完成很多配置，省去了我们安装后修改配置的麻烦。注意把`<admin-user-name>`替换为你想预设的管理员账号名

```shell
sudo apt update
sudo apt install python3 python3-dev git curl
curl https://raw.githubusercontent.com/jupyterhub/the-littlest-jupyterhub/master/bootstrap/bootstrap.py | sudo -E python3 - --admin <admin-user-name>
```

安装完成后我们在浏览器输入服务器的ip地址就能访问`jupyterhub`，输入刚刚预设的管理员账号，并输入想要给它设置的密码即可登录（用户第一次登录输入的密码会被系统记录为用户对应的密码）

另外需要注意的是本文后续所有命令都应该在浏览器端该admin账号的`terminal`中运行

用`The-littlest-jupyterhub`安装的`jupyter-lab`可能不是最新版本，需要进行更新(\*注意此处是`pip`不是`pip3`)

```shell
sudo -E pip install -U jupyterlab
```

安装完毕后需要重启服务器以使更改可用

## 安装应用库和kernel

`jupyterhub`的一个优势在于所有用户共享底层的依赖包，同时代码在各自的空间维护，从而节省资源。安装完成后我们可以借助conda安装常用依赖

```shell
sudo -E conda install -c conda-forge gdal
sudo -E pip install there
```

请注意`sudo -E`非常重要，它保证我们安装的内容可以被所有用户访问

我们也可以像单机安装时那样添加一个kernel以支持不同的语言，比如一个js的kernel。当然`sudo -E`同样是必需的。

```shell
sudo -E npm install -g --unsafe-perm ijavascript
sudo -E ijsinstall --install=global
```

## 分享文件

在用户间分享文件，是多用户服务中的一个常见需求。这一操作在官方文档中也有详解

```shell
sudo mkdir -p /srv/data/my_shared_data_folder
```

我们可以把要分享的文件传入`my_shared_data_folder`文件夹，然后执行如下命令

```shell
cd /etc/skel
sudo ln -s /srv/data/my_shared_data_folder my_shared_data_folder
```

此后**新建**的用户都可以直接访问被分享的文件夹，而已存在的用户需要在用户对应的home目录手动运行此命令

## 默认打开jupyter-lab接口

系统默认提供的是`jupyter-notebook`的前端接口，但现在`jupyter-lab`拥有更加强大的功能，通过如下命令可以把默认的展示接口切换为`jupyter-lab`

```shell
sudo tljh-config set user_environment.default_app jupyterlab
sudo tljh-config reload hub
```

完成后重启服务可以使改动生效，如果想切换回`jupyter-notebook`模式，请把地址从`\lab`修改为`\tree`

## 允许用户自行注册登录

在实际使用中我们需要开放注册权限以使更多用户自助登录使用，首先在admin用户下允许如下命令

```shell
sudo tljh-config set auth.type nativeauthenticator.NativeAuthenticator
sudo tljh-config set auth.NativeAuthenticator.open_signup true
sudo tljh-config reload
```

此后新用户可以访问`server-host/signup`进行账号注册，这里需要注意的一点是目前发现进行上述操作后已有的用户密码会失效，所以所有用户包括`admin`用户需要重新注册登录

## 调整服务清理时间

对每个用户的server，系统会每隔一段时间检查server是否活动，如果不活跃时间超过预设延时，系统会自动清理该server以空出服务器资源。系统预设值为每60s检查，最长不活跃时间600s，但这样的体验很不好，需要频繁重启，所以我们可以把这两个值适当调大

```shell
sudo tljh-config set services.cull.every <number-of-sec-this-check-is-done>
sudo tljh-config set services.cull.timeout <max-idle-sec-before-server-is-culled>
sudo tljh-config reload
```

## 额外内容 - 修改docmanager扩展的默认设置

目前产品内并未找到一个方便的功能来修改一些juyter相关的默认扩展设置。暂时以markdown默认展示方式为例提供一些思路

jupyter lab可以通过设置`Document Manager`来设置某些特定格式文件的展示方式，正常的jupyterlab在设置后可以记忆设置，但在jupyterhub中无法自动将该设置应用于所有用户。想解决这个问题，我们可以通过修改jupyterlab程序文件来达到此目的。

打开`/opt/tljh/user/share/jupyter/lab/staging/node_modules/@jupyterlab/docmanager-extension/schema/plugin.json`及`/opt/tljh/user/share/jupyter/lab/schemas/@jupyterlab/docmanager-extension/plugin.json`，修改其中需要修改的项目的`default`的值，就可以修改jupyterlab的默认配置，实现所有用户间的设置共享。

 
---------------------------------------------------------------
2019-11-14
