[返回首页](../index.md)

## flask-gunicorn-seqlog 实现python api服务日志自动更新到Seq

### 开发背景
近期公司开发一个内部工具，用到了python和c#同时进行开发，其中在python部分采用flask框架进行后端api编写，线上部署时用 [gunicorn](http://docs.gunicorn.org/en/latest/) 工具包装flask程序以增强其可用性。在临近上线前，有需求把python flask部分的日志和c#接口的日志合并存放在Seq服务上，本文主要讲述实现方法和在这个过程中遇到的一些问题。

### 实现方案
#### 1. 安装python库 [seqlog](https://github.com/tintoy/seqlog) ，这是一个实现python log 可以自动传到Seq服务器的工具

```bash
pip3 install seqlog
```

#### 2. 根据seqlog的文档进行logging配置，我们采用了配置文件的方式，其内容如下：

```yml
# This is the Python logging schema version (currently, only the value 1 is supported here).
version: 1

# Configure logging from scratch.
disable_existing_loggers: True

# Configure the root logger to use Seq
root:
  level: INFO
  handlers:
  - seq
  - console

handlers:
# Log to STDOUT
  console:
    class: seqlog.structured_logging.ConsoleStructuredLogHandler
    formatter: seq
    filters: [flask_filter]

# Log to Seq
  seq:
    class: seqlog.structured_logging.SeqLogHandler
    formatter: seq

    # Seq-specific settings (add any others you need, they're just kwargs for SeqLogHandler's constructor).
    server_url: 'http://172.16.0.250:5341/'
    json_encoder_class: omiga.service.tools.StrJSONEncoder

#    api_key: 'your_api_key_if_you_have_one'

formatters:
  seq:
    style: '{'

# you can config some filters there and use it in a handler config
filters:
  flask_filter:
    name: 'werkzeug'

```

在这里有几点需要额外说明：
- 如果不想有多余的信息显示在console，可以用log name `'werkzeug'` 来构建filter，并在handler console中引用该filter
- filters 需要在最外层进行定义，然后在定义handler时可以通过其名字引用定义好的filter
- 由于seqlog会调用http服务将log内容上传至Seq服务器，所以最好在配置handler seq时添加json_encoder_class， 以便其顺利完成json.dumps的工作。

然后可以在flask app中导入配置文件，其代码大致为：

```python3
from flask import Flask
from seqlog import configure_from_file

app = Flask(__name__)

configure_from_file('your-log-config-file-path.yml')
logger = logging.getLogger('flask.app.log')

@app.route('/api/test', methods=['GET'])
def first_api():
    data = 'hello world'
    logger.info('Log to Seq: {data}', data=data)
    return result, 200, {'Access-Control-Allow-Origin': '*'}

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```

运行程序，不久后我们就可以在seq界面中看到刚刚记录的日志信息了

当然，你也可以按照seqlog文档中的描述采取其他配置方式

#### 3. 禁用gunicorn logging文档配置
OK，现在万事大吉，我们可以把程序部署在服务器上运行了。但我们惊奇的发现，在测试环境中勤勤恳恳的日志信息在此时人间蒸发，踪迹难寻。那到底是哪里出了问题呢？
此时我们当然是去找gunicorn的文档问个究竟。

```
Logging can be configured by using various flags detailed in the configuration documentation or by creating a logging configuration file balabala...
```

问题大致找到了，gunicorn会对logging重新进行配置，这就覆盖了我们之前辛苦配置的seqlog。继续在网上查找，我们找到了关掉这个功能的办法：
在启动gunicorn时添加参数 `--access-logfile=-`，完整命令如下

```bash
gunicorn --workers=3 --bind 0.0.0.0:5000 --access-logfile=- app.wsgi:app
```

好了，让我们看一下，线上产生的log信息重新出现在了Seq页面，麻麻再也不用担心出了bug找不到原因了。

### 一些补充
- 本文写作时seqlog中关于json_encoder_class的部分还没正式发布，可以clone该项目并切换到feature/custom-json-encoder分支进行源码安装，但相信这一问题很快就会修复

```bash
git clone git@github.com:tintoy/seqlog.git
git checkout feature/custom-json-encoder
git pull origin feature/custom-json-encoder
cd seqlog
python3 setup.py install
```

- logging的作用范围是全局的，在普通应用中，应用程序处在调用栈的最上层，但在gunicorn中会从gunicorn引用应用程序，造成我们的配置被覆盖

如果有别的问题或有不同的看法，欢迎留言指正。
 
---------------------------------------------------------------
2018-09-15
