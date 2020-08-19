[返回首页](../index.md)

# Numpy中rand和randn的区别

`numpy.random`中`rand`和`randn`函数都有批量生成随机数的功能，它们接受的参数为几个维度的大小，最终返回设定维度的随机数数据集。

```python
>>> import numpy as np
>>> np.random.rand(3, 2)
array([[0.09794152, 0.32535162],
       [0.79178359, 0.86480239],
       [0.692685  , 0.50751617]])
```

这里需要注意的一点是`randn`的`n`是normal的意思，它返回的一组数据符合标准正态分布(standard normal distribution)

```python
import matplotlib.pyplot as plt
plt.hist(np.random.rand(10000), bins=100)
```

![rand](https://snailkn-images.s3.ap-east-1.amazonaws.com/blog/rand_randn/rand.png)

```python
plt.hist(np.random.randn(10000), bins=100)
```

![rand](https://snailkn-images.s3.ap-east-1.amazonaws.com/blog/rand_randn/randn.png)
 
---------------------------------------------------------------
2020-03-12
