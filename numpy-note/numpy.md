Numpy直接以数组、矩阵为粒度计算并且支持大量的数学函数，而Python需要用for循环从底层实现。

Numpy的数组存储效率和输入输出计算性能，比Python使用List或者嵌套List好很多。

```python
def python_sum(n):
   '''Python 实现数组的加法'''
   '''n: 数组的长度'''
   a = [i**2 for i in range(n)]
   b = [i**3 for i in range(n)]
   c = []
   for i in range(n):
      c.append(a[i]+b[i])
   return c

def numpy_sum(n):
   '''numpy 加法'''
   '''n: 数组的长度'''
   a = np.arange(n)**2
   b = np.arange(n)**3
   return a+b
```

