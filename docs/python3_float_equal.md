[返回首页](../index.md)

# Python浮点数的精度问题以及判断两个float数字是否相等

Python中浮点数由于精度的原因，它的实际值可能和你想象的不一样。比如

```python3
>>> 1 == 0.9999999
False
>>> 1 == 0.9999999999999999999
True
```

再比如：

```python3
>>> 1 - 0.2 == 0.8
True
>>> 1 - 0.8 == 0.2
False
>>> 1-0.8
0.19999999999999996
```

其它还好，我们可以接受精度的一点点损失，但在测试时得到的值和我们想象的值差那么一丢丢无法“相等”就比较难受了。
所幸python里预留了忽略这种微小差异的比较方法，那就是 `math` 里的 `isclose` 函数。

```python3
>>> import math

>>> math.isclose(1 - 0.8, 0.2)
True
>>> math.isclose(0.3, 1/3)
False
>>> math.isclose(0.3333333333, 1/3)
True
```
 
 有了这个函数，我们就能够放心地在测试时判断两个float是否相等了。
 
---------------------------------------------------------------
2019-04-13