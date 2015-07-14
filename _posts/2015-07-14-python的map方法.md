---
layout: post
category: 技术
tags: [python]

---

#python的map方法
> map(function, sequence[, sequence, ...]) -> list

###空方法
```python

list1 = [1, 2, 3]
list2 = [4, 5, 6]
list3 = [7, 8, 9]
'''
空方法，单参数
返回自身，此调用没什么意义
'''
print map(None, list1) # [1, 2, 3]
print list1 == map(None, list1) # True

'''
空方法，多参数
将返回一个tuple的list
tuple中是传入数据对应位置的值的一个tuple
'''
result = map(None, list1, list2, list3)
print type(result) # list
print type(result[0]) # tuple
print result # [(1, 4, 7), (2, 5, 8), (3, 6, 9)]

```
###方法调用
```python

list1 = [1, 2, 3]
list2 = [4, 5, 6]
list3 = [7, 8, 9]
'''
方法调用
不方便，方法参数必须与map中的参数数目相对应，否则报错
'''
def addOne(x):
    return x+1

print map(addOne, list1) # [2, 3, 4]

def add(x, y, z):
    return x + y + z

print map(add, list1, list2, list3) # [12, 15, 18]
print map(add, list1, list2) # add() takes exactly 3 arguments (2 given)
```
###lambda表达式
```python

list1 = [1, 2, 3]
list2 = [4, 5, 6]
list3 = [7, 8, 9]
'''
lambda表达式
方便
lambda的参数数量需要与map中参数数目相对应，否则报错
'''
print map(lambda x:x + 1, list1) # [2, 3, 4]
print map(lambda x, y, z: x + y + z, list1, list2, list3) # [12, 15, 18]
print map(lambda x, y, z: x + y + z, list1, list2) # <lambda>() takes exactly 3 arguments (2 given)

```