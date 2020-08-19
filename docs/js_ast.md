[返回首页](../index.md)

# JS - AST基本概念

AST是Abstract Syntax Tree的缩写，可直译为抽象语法树。AST是对js代码逻辑的一种抽象分解，它把代码的计算逻辑细化，并按计算顺序以树的形式存储。利用AST，我们可以实现很多需求，例如语法高亮、语法检查、代码混淆等，babel就是以AST作为基础对js代码进行处理的。关于AST的操作总结下来无非3种：

- 把js代码解析为AST
- 按照需求修改AST
- 把AST格式化为js代码

好在js生态里已经有成熟的工具帮我们实现

### 解析 - esprima

[esprima](https://github.com/jquery/esprima) 库可以帮助我们把js代码解析成标准的AST json，用法如下。

```js
let code = "your js code";
let ast = esprima.parseScript(code);
```

如下为AST解析的一个实例，js代码为：

```js
// sample code
function hello() {
    console.log("hello world");
}

```

对应的AST结果为：

```json
{
  "type": "Program",
  "body": [
    {
      "type": "FunctionDeclaration",
      "id": {
        "type": "Identifier",
        "name": "hello"
      },
      "params": [
        
      ],
      "body": {
        "type": "BlockStatement",
        "body": [
          {
            "type": "ExpressionStatement",
            "expression": {
              "type": "CallExpression",
              "callee": {
                "type": "MemberExpression",
                "computed": false,
                "object": {
                  "type": "Identifier",
                  "name": "console"
                },
                "property": {
                  "type": "Identifier",
                  "name": "log"
                }
              },
              "arguments": [
                {
                  "type": "Literal",
                  "value": "hello world",
                  "raw": "\"hello world\""
                }
              ]
            }
          }
        ]
      },
      "generator": false,
      "expression": false,
      "async": false
    }
  ],
  "sourceType": "script"
}
```

需要注意的是，注释会被解析器所忽略，所以在最终的AST结构里是找不到对应的注释信息的。

### 修改 - estraverse

[estraverse](https://github.com/estools/estraverse)实质上是一个AST的遍历工具，它也可以对节点进行操作，我们可以借助它对js AST进行修改。比如我们可以利用它把所有的异步函数修改为同步，删除对应的`async` `await` 关键词。

```js
// 消除所有的异步调用
function async2sync(ast) {
    estraverse.traverse(ast, {
        enter: function (node) {
            if (node.type === "FunctionDeclaration") node.async = false;
        }
    });
    estraverse.replace(ast, {
        leave: function (node) {
            if (node.type === 'AwaitExpression') return node.argument;
        }
    });
    return ast;
}

// 转换前
async function hello() {
    await console.log("hello world");
}

// 转换后
async function hello() {
    await console.log("hello world");
}
```

estraverse主要包含traverse和replace两个操作，一个用来遍历AST，一个用来替换AST节点，有需要可以看一下官方文档。

### 格式化 - escodegen

[escodegen](https://github.com/estools/escodegen)用来把AST重新格式化为js函数，这一步相对来说比较简单。

```js
let code = escodegen.generate(ast);
```

----------------------

AST是编译原理的一个基础，并不限于js所专有。利用AST不仅可以按需求在格式上改变代码，也可以更加方便得进行代码性能优化。有机会可以更加深入地了解关于代码执行的原理。
 
---------------------------------------------------------------
2019-10-15
