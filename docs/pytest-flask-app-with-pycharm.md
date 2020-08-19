[返回首页](../index.md)

## 利用pycharm+pytest测试flask接口

近期开发flask接口时利用pycharm配合pytest进行了代码测试，本文将对环境搭建和测试过程作出说明

### 1. 环境搭建

- 安装pycharm应用，并修改默认测试引擎为 `py.test`, 修改方式如下所示，也可以 [参考官方文档](https://www.jetbrains.com/help/pycharm/choosing-your-testing-framework.html)
```
Settings/Preferences -> Tools -> Python Integrated Tools -> Default Test Runner ->  (select) py.test
```

- 安装pytest 及 coverage

```bash
pip3 install pytest
pip3 install coverage
```

### 2. 编写flask api接口

根据需求编写flask api接口函数，一个基本的接口函数如下

- api.py

```python3
@app.route('/api/test', methods=['GET'])
def first_api():
    data = 'hello world'
    return data, 200, {'Access-Control-Allow-Origin': '*'}
```

### 3. 利用flask client编写测试用例

接口写好后，我们可以根据接口的定义（应该接收什么，返回什么，什么时候报什么样的错等）编写测试用例。当然，在编写接口之前先写测试用例也是一个不错的选择，这样你可以清楚得定义接口要实现的功能，在具体实现时拥有更清晰的思路。我们一般会新建一个测试文件，然后定义一个全局的client，然后在每个测试函数中调用这个全局变量。在创建测试文件时，一般用 `'test_'` 加文件名／函数名来表明要测试的对象。以下为代码实例：

- test_api.py

```python3
import pytest
import api

@pytest.fixture
def client():
    api.app.config['TESTING'] = True
    test_client = api.app.test_client()
    yield test_client
    
def test_first_api(client):
    r = client.get('http://localhost:5000/api/test')
    assert r.status_code == 200
    assert r.data = b'hello world'
```

### 4. 运行测试用例并分析代码

编写完测试用例后，可以右键文件名，选择 Run py.test in api.py 运行测试用例，此时界面会自动变成测试界面，说明我们的测试配置成功了。

你也可以选择 Run py.test in api.py with coverage 来展示代码覆盖情况，并根据结果编写代码覆盖率更高的测试用例。

### 5. 补充说明

在配置时我遇到了无法显示 Run py.test 选项的情况，这可能是ide配置未做刷新引起的，只要删除在项目根目录下的 `.idea` 文件并重启pycharm即可解决 

```bash
rm -rf .idea
```

---------------------------------------------------------------
2018-09-15