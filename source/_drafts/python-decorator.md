---
title: Python语言进阶：装饰器函数介绍及实际应用
tags:
  - Python
categories:
  - Python
date: 2019-01-06 22:36:00
---

## 引言
在Python开发中，装饰器（Decorator）是一种强大而优雅的语法特性，它允许在**不修改原函数代码**的前提下，动态地为函数添加额外功能（如日志记录、性能分析、参数校验等）。本文将从闭包的基础概念出发，深入解析装饰器的实现原理，并通过实际示例展示其应用场景。

---

## 闭包：装饰器的基石
在理解装饰器之前，必须先了解**闭包（Closure）**的概念。闭包是指在函数内部定义的函数（嵌套函数），该函数可以访问外层函数的变量，即使外层函数已经执行完毕。

```python
def outer():
    x = 10
    def inner():
        print(f"通过闭包访问外层变量x的值: {x}")
    return inner	# 返回闭包

closure = outer()
closure()  # 输出: 通过闭包访问外层变量x的值: 10
```
在这个例子中，`outer` 返回 `inner`，后者可以访问并使用 `outer` 中的变量 `x`

---

## 装饰器：闭包的应用
装饰器的本质是一个接收函数作为参数的外层函数，其内部定义一个闭包（通常命名为`wrapper`），用于包裹原函数并增强其功能。

```python
# 装饰器模版
def decorator(func):          # 外层函数（装饰器）
    def wrapper(*args, **kwargs):  # 内层闭包
        # do something
        return func(*args, **kwargs)  # 引用外层函数的参数 `func`
    return wrapper  # 返回闭包
```
在上面的例子中，decorator()就是一个装饰器。它接受一个函数作为参数，并返回另一个函数。

除了函数装饰器，还可以通过类实现装饰器。类装饰器需实现`__call__`方法。

```python
# 类装饰器模版
class Decorator(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print(f"函数 {self.func.__name__} 被调用")
        return self.func(*args, **kwargs)

@Decorator
def say_hello():
    print("Hello!")

say_hello()  # 输出: 函数 say_hello 被调用 → Hello!
```
## 装饰器应用场景

1.日志记录：在大型项目中，使用装饰器可以轻松地为多个函数添加日志功能，而无需在每个函数中重复编写日志代码。

2.性能分析：通过装饰器可以方便地测量函数的执行时间，帮助开发者识别性能瓶颈。

3.访问控制：在Web应用中，装饰器可以用于实现用户认证和授权，确保只有具有特定权限的用户才能访问某些功能。

4.缓存机制：对于计算密集型函数，使用缓存装饰器可以显著提高程序的执行效率，避免重复计算。

5.输入验证：在数据处理应用中，装饰器可以用于检查函数输入的有效性，提高代码的健壮性。

## 装饰器代码示例

### 日志记录装饰器

```python
import logging
from functools import wraps

def log(func):
    @wraps(func)  # 保留原函数的元数据（如函数名、文档）
    def wrapper(*args, **kwargs):
        # logging.getLogger().setLevel(logging.INFO)
        logging.info(f"调用函数: {func.__name__}, 参数: {args}, {kwargs}")
        result = func(*args, **kwargs)
        logging.info(f"函数返回: {result}")
        return result	# 保证原函数 func 的返回值不丢失（若原函数存在return语句）
    return wrapper

@log
def add(x, y):
    """两数相加"""
    return x + y

add(3, 5)  # 自动记录日志
```

- 解析：
  - `@wraps(func)`：保留原函数的元数据，避免调试时丢失信息。
  - `*args, **kwargs`：接受任意参数，增强装饰器的通用性。

### 计时装饰器

```python
class Timer:
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        import time
        start = time.time()
        result = self.func(*args, **kwargs)
        print(f"耗时：{time.time()-start}秒")
        return result

@Timer  # 用类作为装饰器
def heavy_task():
    # 模拟耗时操作
    import time
    time.sleep(2)

heavy_task()  # 输出：耗时：2.000123秒
```

### 重试装饰器
若需要为装饰器传递参数（如重试次数），需定义一个**装饰器工厂函数**，它可以根据参数“生产”出不同的装饰器。

```python
def repeat(num_times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(num_times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(num_times=3)
def greet(name):
    print(f"Hello {name}!")

greet("Alice")  # 输出3次"Hello Alice!"
```

从以上例子中可以看到，在带参数的装饰器函数中一共有三层函数嵌套，其实剥离出最外面的一层，我们可以发现和简单（不带参数）的装饰器是一样的。

- **解析**：
  - `repeat(3)`返回一个装饰器`decorator`。
  - `@repeat(num_times=3)`等价于`greet = repeat(3)(greet)`。

## 总结：装饰器让代码更优雅
装饰器就像乐高积木，可以自由组合各种功能模块。掌握它之后：

✅ 不用复制粘贴重复代码

✅ 功能模块随意搭配

✅ 代码更简洁易维护
