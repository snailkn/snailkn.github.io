```python3
import torch

def prepare_surv_data(*args, t=None):
    """
    用来进行batch train之前的数据准备，可以有多组数据
    t: Tensor, 观察时间数据列
    """
    assert t is not None
    idx = t.argsort(descending=True)
    return (x[idx] for x in args)
  
 def negative_average_log_likelihood_loss(risk, event):
    """
    DeepSurv所使用的损失函数，计算前提是数据必须已按观察时间从大到小排序
    risk: Tensor, 模型计算得出的结果
    event: Tensor, 事件发生/删失标签, 标签数据
    """
    epsilon = 1e-7
    partial_hazard = torch.exp(risk)
    return -(torch.sum((risk - torch.log(torch.cumsum(partial_hazard, dim=0))) * event) + epsilon) / (torch.sum(event) + epsilon)
```