[返回首页](../index.md)

# 在bash中引用json文件记录的参数

把项目配置参数记录到json里是一个常见的做法，但怎么在bash中导入这些参数呢？一种方案是利用jq命令，但它的不足是需要额外安装jq。在网上找到一个比较取巧的基于python的办法，但他给的示例比较模糊。现对该方法详细说明。

该方法的原理是利用`eval`命令，配合python读取json文件，再把它组合成bash命令，并print出来。以下是具体实例

首先，有一个 *config.json* 文件，用来保存一些配置

```json
{
    "hello": "snailkn"
}
```

然后我们的bash脚本

```bash
eval `python3 -c "
import json
with open('config.json', 'r') as f:
    config = json.load(f)
    print('export HELLO={}'.format(config['hello']))
"`

echo hello $HELLO
```

执行该脚本，可用看到出现 `hello snailkn` 的输出。这样，我们就可以把json里记录的参数传入bash使用了
 
---------------------------------------------------------------
2020-06-09
