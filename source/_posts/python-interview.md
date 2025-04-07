---
title: Python面试问题汇总
tags:
  - Python
categories:
  - Python
date: 2020-01-22 20:30:10
---
# Python 面试题

## 数据结构

### 列举 Python 中的基本数据结构

| 数据结构   | 特点                                                 | 应用场景                                           |
| ---------- | ---------------------------------------------------- | -------------------------------------------------- |
| **列表**   | 有序、可变、允许重复元素                             | 动态存储异构数据（如同时存储整数、字符串等）       |
| **元组**   | 有序、不可变、允许重复元素                           | 存储固定数据（如坐标、配置参数），比列表更节省内存 |
| **字典**   | 无序（Python 3.7+ 有序）、可变、键值对（key: value） | 快速通过键查找值（如用户信息映射）                 |
| **集合**   | 无序、可变、唯一性（自动去重）                       | 数学集合运算（交集、并集）、快速成员判断           |
| **字符串** | 有序、不可变、字符序列                               | 文本处理、格式化输出                               |

> 基本结构通常指列表、元组、字典、集合，而字符串可能作为序列类型单独列出

### 如何将字符串 "ilovechina" 进行反转

```python
s = 'ilovechina'
s = s[::-1]
```

### 如何将 "1,2,3" 分割成 ["1","2","3"]？

```python
s = "1,2,3"
stdout = s.split(',')
```

### Python 中的字符串格式化方式你知道哪些？

Python3.6之后的版本提供了三种字符串格式化的方式

```python
# 1. %s占位符
def foo(name):
  return 'hello %s' % name

# 2. format() 
def foo(name):
  return 'hello {}'.format(name)

# 3.f-string
def foo(name):
  return f'hello {name}'
```

### 列表元素怎么去重

自己动手写函数实现（保留原顺序）

```Python
old_list = [2, 3, 4, 5, 1, 2, 3]
new_list = []
for i in old_list:
    if i not in new_list:
        new_list.append(i)
print(new_list) # [2, 3, 4, 5, 1]
```

用字典dict去重（保留原顺序）

```Python
# 使用list元素值创建一个dict，这将自动删除任何重复项
old_list = [2, 3, 4, 5, 1, 2, 3]
new_list = list(dict.fromkeys(old_list))
print(new_list) # [2, 3, 4, 5, 1]
```

使用sort函数来去重（保留原顺序）

```Python
lst1 = [2, 1, 3, 4, 1]
lst2.sort(key=lst1.index)
print(lst2)    # [2, 1, 3, 4]
```

使用内置set方法（不保证原顺序）

```Python
# 将list转化为set再转化为list
list1 = [1, 3, 4, 3, 2, 1]
list2 = list(set(list1))
print(list2)    # {1, 2, 3, 4}
```

### 两个列表怎么转成字典

- 动手写一个函数

```Python
list1 = ['name', 'age', 'sex']
list2 = ['张三', 18, '男']
dict = {}
for i in range(len(list1)):
    dict[list1[i]] = list2[i]
print(dict, type(dict)) # {'name': '张三', 'age': 18, 'sex': '男'} <class 'dict'>
```

- 使用内置函数zip

```Python
list1 = ['name', 'age', 'sex']
list2 = ['张三', 18, '男']
d = dict(zip(list1, list2))
print(d)    # {'name': '张三', 'age': 18, 'sex': '男'}
```

### 如何合并两个列表 

```python
a = [1,5,7,9]
b = [2,2,6,8]

# 方法1
a.extend(b)

# 方法2
a[0:0] = b

# 方法3
a += b
```

### 举例 sort 和 sorted 的区别

```
sort()与sorted()的不同在于，sort是在原位重新排列列表，而sorted()是产生一个新的列表
```

### 字典操作中 del 和 pop 有什么区别？

```
del 操作删除键值对，不返回值；
pop 操作删除键值对的同时，返回键所对应的值。
```

### 如何合并两个字典 

```python
# python3合并字典有三种方式
# 1.
a = {'a':1,'b':2}
b = {'c':3,'d':4}
c = {}
c.update(a)
c.update(b)

# 2.
c = dict(a,**b)

# 3.
c = {**a,**b} # 官方推荐这种方式
```

### 如何把 元组 ("a","b") 和 元组(1,2)，变为字典{"a":1,"b":2}？

```python
a = ('a','b')
b = (1,2)
z = zip(a,b)
c = dict(z)
```

### 三元运算写法和应用场景

三元运算符的作用就是判断，可以理解为if条件判断的简化版。

```
条件语句比较简单时可以使用三元运算符，最常见的就是 
res = 'test True' if expression is True else 'test False'
```

## 文件处理

### 在读文件操作的时候会使用 read、readline 或者 readlines，简述它们各自的作用

read将整个文本都读取为一个字符串，占用内存大，readline读取为一个生成器，支持遍历和迭代，占用空间小。

readlines将文本读取为列表，占用空间大。

### Python 字典和 json 字符串相互转化方法

```
json.dumps()   将Python中的对象转换为JSON中的字符串对象
json.loads()   将JSON中的字符串对象转换为Python中的对象
```

## 高级特性

### 函数装饰器有什么作用？请列举说明

函数装饰器可以在不修改原函数的条件下，为原函数添加额外的功能，例如记录日志，运行性能，缓存等

### 简述 @classmethod 和 @staticmethod 用法和区别

| **对比项**     | **`@classmethod`**                                       | **`@staticmethod`**            |
| -------------- | -------------------------------------------------------- | ------------------------------ |
| **参数**       | 必须有 `cls`（类本身）                                   | 无特殊参数，类似普通函数       |
| **访问权限**   | 可访问/修改类属性                                        | 无法访问类或实例属性           |
| **依赖性**     | 依赖类的上下文                                           | 与类或实例无关                 |
| **典型用途**   | 工厂方法（替代构造函数来创建对象）、操作类属性、多态场景 | 工具函数、逻辑相关但独立的功能 |
| **调用灵活性** | 推荐通过类名调用                                         | 推荐通过类名调用               |

### 你了解 Python 中的反射吗？

通过字符串映射object对象的方法或者属性

1. hasattr(obj,name_str): 判断objec是否有name_str这个方法或者属性
2. getattr(obj,name_str): 获取object对象中与name_str同名的方法或者函数
3. setattr(obj,name_str,value): 为object对象设置一个以name_str为名的value方法或者属性
4. delattr(obj,name_str): 删除object对象中的name_str方法或者属性

举个栗子

```python
import requests

class Http(object):

    def get(self,url):
        res = requests.get(url)
        response = res.text
        return response

    def post(self,url):
        res = requests.post(url)
        response = res.text
        return response

# 使用反射后
url = "https://www.jianshu.com/u/14140bf8f6c7"
method = input("请求方法>>>:")
h = Http()

if hasattr(h,method):
    func = getattr(h,method)
    res = func(url)
    print(res)
else:
    print("你的请求方式有误...")
```

### copy 和 deepcopy 的区别是什么？

copy 即所谓的浅拷贝，赋值的时候非递归地复制子对象的引用；
deepcopy 即所谓的深拷贝，赋值的时候递归复制子对象。

```python
xs = [1,2,[2,3,4],3]
ys = xs # 浅拷贝
zs = deepcopy(xs) # 深拷贝
xs[2][0] = 5
print(ys)
[1,2,[2,3,4],3]
print(xs)
[1,2,[5,3,4],3]

print(zs)
[1,2,[5,3,4],3] # 由于深拷贝已经递归复制了子对象，所以内部的List也发生改变
```

## 算法题目

### 一行代码输出 1-100 之间的所有偶数

```python
[x for x in range(101) if x %2 ==0]
```

### 请写一个 Python 逻辑，计算一个文件中的大写字母数量。

```python
import re
with open('a.txt','r') as f:
    f_content = f.read()
    len_cap = len(re.compile(r'[A-Z]').findall(f_content))
```

### 根据当前时间输出N分钟、小时后的时间

```python
from datetime import datetime, timedelta

def get_query_time(minutes=10, hours=None):
    """获取查询的时间段，默认返回最近的10分钟"""
    now_time = datetime.now()
    start_time = now_time.strftime("%Y-%m-%d %H:%M:%S")
    end_time = (now_time + timedelta(minutes=minutes)).strftime("%Y-%m-%d %H:%M:%S")
    if hours:
        end_time = (now_time + timedelta(hours=hours)).strftime("%Y-%m-%d %H:%M:%S")
    return start_time, end_time
```
