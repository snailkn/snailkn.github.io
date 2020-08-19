[返回首页](../index.md)

# python环境离线迁移

先在一个有网的环境下安装所有依赖包

然后运行如下代码

```bash
pip3 freeze > requirements.txt
pip3 download -r requirements.txt -d path/to/download
```

pip会首先生成一个`requirements`文件来保存该环境信息，该文件可以直接用在有网时的环境迁移。然后，pip会根据`requirements`文件的信息把python安装包下载在设定的目录下

将`requirements`文件和存放安装包的文件夹一起打包，传到离线环境中，然后运行如下代码

```bash
pip3 install –no-index –find-links=path/to/download/packages -r requirements.txt
```

然后就可以完成依赖环境的迁移了
 
---------------------------------------------------------------
2020-02-25
