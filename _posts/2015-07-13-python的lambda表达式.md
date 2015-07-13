---
layout: post
category : 技术
tags : [python,lambda]

---

## Python的lambda表达式
##### 无参数

```python
a = lambda: 'hello, world!'
print a() # hello, world!
print lambda: 'hello, world!' # <function <lambda> at 0x........>
```

##### 有参数
```python
sum = lambda x,y = 10: x + y
print sum(1) # 11
print sum(2, 2) # 4
```

-----
