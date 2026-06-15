# Pytest测试项目实战

pytest是一款基于unitest开发的python测试框架，并且具有非常丰富的插件，能够作为编写和执行单元测试、集成测试以及功能、性能测试的首选工具。

# **关键特性**

**简单的测试编写：**

- 使用极简的`assert`语句，你可以不需要记住复杂的断言，直接对比期望和实际结果。

- 不需要编写类或方法，普通函数就可以是测试函数。

**自动测试发现：**

- `pytest` 默认自动发现以 `test_` 开头或`_test`结尾的文件和函数作为测试用例。

**详细的测试结果报告：**

- `pytest` 提供详细的错误报告，当测试失败时，会显示哪一行代码失败以及失败的详细信息。

**支持fixtures：**

- 通过使用`fixtures`，`pytest` 可以为测试提供创建/销毁和管理资源（如数据库连接、临时的文件等）。这种机制提供了可重用的功能以及设置和清除条件。

**参数化测试：**

- 使用 `pytest.mark.parametrize` 装饰器可以轻松地对单个测试实现多个参数组合的测试。

**插件系统：**

- `pytest` 有一个广泛的插件生态系统，允许通过安装第三方插件或创建自定义插件来扩展其功能。比如pytest\-rerunfailures、pytest\-xdist

- 插件可以通过钩子（hook）函数来扩展 `pytest` 的行为

**可集成性：**

- `pytest` 可与许多其他测试和开发工具集成，如测试覆盖率工具 `coverage.py`，持续集成服务等。

# 参数详解

[《pytest测试指南》\-\- 章节1\-3 pytest参数详解PART1 \| Transcendent](https://gavin-wang-note.github.io/2024/03/03/pytest_test_guide_part1_chapter1_3_1_pytest_params/#3-2-2-1-can-shu-code-k-code)

# 用例执行

## **用例命名规则**

1. 测试函数（function）必须以 test\_ 开头

2. 测试类（class）必须以"Test"开头，且不能包含 \_init\_ 方法，类中的方法必须以test\_开头

3. 测试文件（module）通常以 test\_\*\.py 文件或\*test\.py

4. 测试目录（pakege） 必须要有\_\_init\_\_\.py 文件

## 用例执行方式

### **main\(\)函数运行**

- 运行所有测试用例：`pytest.main()`

- 运行指定路径的用例：`pytest.main( ["./路径"] )`

- 运行指定模块的用例：`pytest.main( ["模块名.py"] )`

- 运行指定用例：`pytest.main( [ " ./路径/模块名::类名::方法"] )`

### **pytest命令行运行**

- 运行所有测试用例：`pytest`

- 运行指定路径的用例：`pytest ./路径`

- 运行指定模块的用例：`pytest 模块名`

- 运行指定用例：`pytest ./路径/模块名::类名::方法`

### **使用配置文件运行程序**

借助`pytest.ini`全局配置文件，更改`pytest`默认行为，用例在执行时，遵循如下参数读取规则：

- 如`pytest.ini`有该参数值，在执行的时候，先读取配置文件中的参数；

- 如没有，则取其他地方的（主函数/命令行中）。

```Python
[pytest]
addopts  =  -vs            # 命令行参数，多个命令用空格分隔
testpaths = ./             # 测试用例的路径
python_files = test*.py    # 配置测试搜索的模块文件名称（模块名的规则）
python_class = Test*       # 配置测试搜索的类名 （类名的规则）
python_function = test     # 配置测试搜索的测试函数名 （方法名的规则）
```

## 用例标记（mark）

标记是`pytest`测试框架中的一个重要特性，它允许开发者在测试用例中添加元数据，以便在运行测试时进行过滤、排序或执行特定的操作。

### **\-m 参数执行指定标记的用例**

Pytest 里面将`@pytest.mark.标记名` 放到测试函数或者类上面，执行测试时在命令行加入 `-m 标记名`

### 跳过测试

pytest\.mark\.skip  可以标记无法在某些平台上运行的测试功能，或者希望满足某些条件才执行某些测试用例，否则pytest会跳过运行该测试用例。

### 标记用例为预期失败

`@pytest.mark.xfail`是一个装饰器，可以用来标记一个测试用例，表示期望这个用例执行失败。当使用这个装饰器标记一个用例时，该用例会正常执行，只是在失败时不再显示堆栈信息。最终的结果有两个：用例执行失败时，结果标记为`XFAIL`（符合预期的失败）；用例执行成功时，结果标记为`XPASS`（不符合预期的成功），这可能是一个好消息，因为它意味着问题可能已经被解决了。

## 其它用例执行策略

### **各种不同失败执行方式的差异**

在下表中，将展示使用`pytest`时，各种不同失败执行方式的差异：

### **重复执行**

- 如果需要验证偶现问题，这个插件（pytest\-repeat）将很有用，可以一次又一次地运行相同的测试直到失败。

```Python
pytest --count=1000 -x test_file.py
```

- 如果要在代码中将某些测试用例标记为执行重复多次，可以使用 @pytest\.mark\.repeat\(count\) 

### 分布式执行

pytest\-xdist插件默认是无序执行的，可以通过 \-\-dist 参数来控制顺序。

\-\-dist=loadscope 

- 将按照同一个模块module下的函数和同一个测试类class下的方法来分组，然后将每个测试组发给可以执行的worker

- 按类class分组优先于按模块module分组

\-\-dist=loadfile 

按照同一个文件名来分组，然后将每个测试组发给可以执行的worker

# fixture 函数

`fixture` 是 `pytest` 中一个核心的概念，它充当了测试函数中所需资源和状态的外部提供者。无论是数据库连接、临时文件目录，还是自定义数据的加载，`fixture` 都能够优雅且高效地管理它们的生命周期。

这解放了开发者从繁琐的设置（`setup`）和销毁（`teardown`）流程，使得他们可以将关注点更多地放在测试逻辑本身，从而提升了测试代码的可读性、可维护性和可重用性。

## 关键特性

fixture的主要的目的是为了提供一种可靠和可重复性的手法去运行那些最基本的测试内容。

下面是一些关于 `fixture` 的关键特性：

- **重用性**： `fixture` 可以在多个测试中重用，包括不同的模块和类之间，避免了代码的重复，实现了共享。

- **作用域**： `fixture` 可以定义不同的作用域（scope），比如 `function`、`class`、`module` 或 `session`。不同作用域的 `fixture` 会在不同的测试阶段被调用和销毁。

- **参数化**： 可以对 `fixture` 进行参数化，允许在多个参数组合下测试相同的代码路径。

- **资源管理**： `fixture` 通常用来进行资源管理，如打开和关闭文件，连接和断开数据库连接。使用 `yield` 语法，`fixture` 可以在 `yield` 之前设置资源，在 `yield` 之后清理资源。

## fixture **参数**

@pytest\.fixture\(\)有5个参数，如下：

```Bash
@pytest.fixture(scope="", params="", autouse="", ids="", name="") 
```

**scope**

scope的层次及神奇的yield组合相当于各种setup和teardown，可以跨函数function，类class，模块module或整个测试session范围。

**params**

允许你在fixture中传入参数，可以是一个值、一个可迭代对象（如：列表，元组，字典列表\[\{\},\{\},\{\}\]，字典元组\(\{\},\{\},\{\}\)），甚至是一个函数。这意味着fixture函数将被多次调用，每次使用不同参数，这对于测试不同场景非常有用。

**autouse**

自动执行fixture，默认False表示不开启；设置为True表示自动调用fixture功能，这样测试用例就不用显式传参了

**ids**

当与params结合使用时，ids可以提供一个字符串列表，用于为生成的测试用例设置唯一的ID。这在参数化fixture中很有用，可以让你轻松区分测试用例的不同参数组合。

**name**

给@pytest\.fixture\(\)标记的函数取一个别名。当fixture生成多个测试用例时（通过params或者ids），测试用例的名称将会基于这个name参数进行生成。

## fixture **使用**

### 定义和使用

1. 定义fixture：通过`@pytest.fixture()`装饰器来定义函数为一个fixture

2. 调用fixture函数：有以下两种方式

    - 在测试用例/测试类上面加上：`@pytest.mark.usefixture(“fixture函数名”)`，但是该方式无法获取到被fixture装饰的函数返回值

    - 将fixture函数名，作为测试用例函数的参数，但是测试类无法这样使用

3. pytest在执行测试用例之前，优先执行fixture函数。fixture函数扮演”注入器“的角色，测试用例来“消费”这些fixture对象。

4. 用例执行完成后，执行yield之后的后置代码

### **数据清理**

#### yield

使用 `yield` 是实现 `fixture` 清理逻辑的推荐方法。在 `yield` 之前的代码是setup代码，之后的代码是teardown代码。

```Python
# conftest.py文件内容
import smtplib
import pytest

@pytest.fixture(scope="module")
def smtp_connection():
    smtp_connection = smtplib.SMTP("smtp.gmail.com",587,timeout=5)
    yield  smtp_connection # provide the fixture value 
    print("teardown smtp") 
    smtp_connection.close()
```

#### request\.addfinalizer方法

`addfinalizer` 允许你在 fixture 函数中注册一个或多个清理函数（finalizer），这些函数会在测试结束时自动执行。`addfinalizer` 的主要优势是清理代码的调用时机更灵活，它是在 `fixture` 函数之中注册的一个清理回调。

以下是smtp\_connection fixture函数更改为使用addfinalizer进行teardown：

```Python
# conftest.py文件内容
import smtplib
import pytest

@pytest.fixture(scope="module")
def smtp_connection(request):
    smtplib.SMTP("smtp.gmail.com",587,timeout=5)
    
    def fin(): 
        print("teardown smtp_connection") 
        smtp_connection.close()
        
    request.addfinalizer(fin) 
    return smtp_connection # provide the fixture value 
```

在此代码中，我们定义了一个清理函数 `fin`，使用 `request` 对象的 `addfinalizer()` 方法注册这个函数作为测试完毕后的回调。`addfinalizer` 允许注册多个独立的清理函数，并在 fixture 作用域结束时按先进后出的顺序运行。

以下是 `yield` 与 `addfinalizer` 在 `pytest` 中使用的对比：

#### 使用上下文管理器管理资源

在某些情况下，需要使用支持上下文管理（context manager）的资源。在这些情况下，可以结合 `with` 语句与 `fixture` 使用。with 语句帮助确保资源正确关闭，即使测试引发了异常。

```Python
# content of test_with_yield_fixture.py
# 使用 with 语句的 fixtureimport pytest

@pytest.fixture
def resource_with_context_manager():
    print("Setting up with context manager")
    with open('file.txt', 'w') as file:
        yield file
    # 文件在上下文退出时自动关闭

def test_example(resource_with_context_manager):
    assert not resource_with_context_manager.closed
```

在这个例子中，`fixture` 打开一个文件，并将其作为生成器返回。`with` 语句确保即使测试中出现异常，文件也会被正确关闭。

#### 其它方法

[关于pytest测试方法的tear down方式 · CuriousY](https://cuyu.github.io/python/2017/06/09/%E5%85%B3%E4%BA%8Epytest%E6%B5%8B%E8%AF%95%E6%96%B9%E6%B3%95%E7%9A%84tear-down%E6%96%B9%E5%BC%8F)

### fixture 嵌套使用

“嵌套”意味着一个`fixture`可以使用另一个`fixture`，这种情况下，内部的（嵌套的）`fixture`会先于外部的`fixture`执行其设置和清理过程。对于嵌套`fixture`的设置部分，会以嵌套顺序执行，而清理过程则会以相反的顺序执行。

### fixture 返回工厂函数

通过使用 `fixture` 返回工厂函数，可以根据测试函数中的具体要求动态生成合适的测试对象，相较于fixture直接返回数据，返回一个产生数据的工厂函数会更好。这种方式保持了测试代码的可读性和灵活性，还可以根据需要进行适当的资源清理。

这种方式在你需要根据测试用例的不同需求来重用和定制对象时非常有用，例如：

- 创建具有不同初始数据的数据库实例。

- 生成带有各种设置的模拟对象或服务。

- 动态构建带有特定配置的测试环境或上下文。

## **conftest\.py 文件**

`conftest.py` 是 `pytest` 测试框架中一个强有力的工具，它提供了一种组织和共享测试代码的优雅方式（如固件 \(fixtures\)，钩子函数 \(hook functions\)），同时也大大增强了 `pytest` 的配置灵活性和测试代码的再利用性。通过掌握合理地使用 `conftest.py` 文件，可以更高效地编写和管理测试代码，确保测试的高可维护性和扩展性。

### **使用场景**

以下是广泛使用 `conftest.py` 的几种典型场景：

- 共享固件：在 `conftest.py` 中定义固件可以被同一目录以及子目录中的所有测试用例共享使用。

- 存放自定义钩子函数：用户可以自定义钩子函数，用于扩展或修改 `pytest` 的内置行为。

- 注册自定义标记：在 `conftest.py` 中可以定义和注册自定义标记 \(markers\)，便于标记和过滤测试用例。

- 添加命令行选项：通过 `conftest.py`，可以向 `pytest` 命令行接口添加自定义选项。

- 集成外部的插件：加载和配置外部 `pytest` 插件。

### **不同位置conftest\.py文件的作用域**

一个项目可以包含多个 `conftest.py` 文件，每个文件的作用域仅限于它所在的目录和子目录里的测试模块。当运行测试时，`pytest` 会收集所有相关目录下的 `conftest.py` 文件，并相应地应用它们提供的配置。

- 在测试框架的根目录创建`conftest.py`文件，文件中的`fixture`的作用范围是所有测试模块

- 在某个单独的测试文件夹里创建`conftest.py`文件，文件中`fixture`的作用范围，就仅局限于该测试文件夹里的测试模块；该测试文件夹外的测试模块，或者该测试文件夹外的测试文件夹，是无法调用到这个`conftest.py`文件中的`fixture`。

- 如果测试框架的根目录和子包中都有`conftest.py`文件，并且这两个`conftest.py`文件中都有一个同名的`fixture`，实际生效的是测试框架中子包目录下的`conftest.py`文件中配置的`fixture`（即此场景下子目录下的`conftest.py`会覆盖父目录的`conftest.py`）



# 测试报告

## **pytest\-html**

pytest是借助pytest\-html插件生成测试报告，不用自己编写生成报告代码。

```Python
pytest --html=reportname.html test_user.py
```

## **allure\-pytest**

### Allure 报告

1. 运行测试并生成数据，存放在当前目录allure\-result文件夹下

```Python
pytest -s --alluredir ./allure-result
```

2. 生成allure报告

```Python
# Allure CLI 生成报告，自动打开
allure serve allure-result -h 10.11.10.137 -p 1688

# 先生成报告，指定输出到allure-report文件夹
allure generate allure-result/ -o ./allure-report -c
```

3. 打开测试报告

```Python
allure open ./allure-report
```

### Allure 属性

### Allure 实践

#### 清空之前的测试报告数据

当我们使用allure生成测试报告之后，我们再修改测试用例，然后再次运行生成测试报告会发现测试报告中保留了上一次用例的运行记录。【默认allure测试报告不会清理之前的原始数据；而原来的原始数据会在最新的allure测试报告中显示历史用例执行结果】

[pytest之allure\(八\)之清空上一次运行的记录\(\-\-clean\-alluredir\)【清空的是测试报告的原始数据\(json/text/attach\)，而不是generate生成测试报告后的re](https://www.cnblogs.com/hls-code/p/15164903.html)

# 参数化

`pytest` 通过 `pytest.mark.parameterize` 装饰器提供了参数化的能力，让测试者能够定义多组参数和值，然后 `pytest` 会自动为每组参数创建一个测试用例。结合使用数据驱动的方式将测试数据和测试逻辑分离，可通过外部文件（如 JSON、YAML 或 CSV）来管理测试数据，从而避免硬编码在测试用例中。

## 使用方法

pytest在如下几个级别上支持测试参数化：

- pytest\.fixture\(\)允许对fixture函数进行参数化

- @pytest\.mark\.parametrize允许在测试函数或类中定义多组参数和fixture

- pytest\_generate\_tests允许自定义参数化方案或扩展

## **indirect 参数**

当设置为 `False`（默认值）时，`argvalues` 直接应用到测试函数的参数。

如果设置为 `True`，则每个参数值都会作为参数传递给对应的 `fixture function`，这个 `fixture function` 通过`param` 接受参数，然后负责提供实际的测试值。

`indirect`参数常用于如下场景：

- **复杂的参数初始化** 当测试参数需要通过复杂的设置步骤进行初始化时，使用 `fixture` 可以使代码更加整洁。

- **共享资源配置** 当你需要对参数化的测试使用不同的配置时，`indirect` 参数可以确保每个测试能够使用配置后的资源。

- **上下文管理** 当需要在测试前后执行清理或其他上下文管理时，通过 `fixture` 管理这一逻辑可以让测试函数保持简洁。

# Pytest 测试实战

## 数据驱动用例的编写

1、测试数据存放在yaml或csv文件中，与用例彻底分离

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MDM0NTVhNDE5YTY1OWZiY2Y0YTExZDRhNDMwMjFiNTNfMDIyYWNkY2E5YjMxNzY2MTUxZjZlZGVjODhhYmU3ZDhfSUQ6NzQ1MTUzNjEyMTA1MzM0Nzg0Ml8xNzgxNTM3MDc0OjE3ODE2MjM0NzRfVjM)

2、用例层在执行时，通过函数（TestData）动态读取yaml中的测试数据即可

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YzJjYjM3OWNlY2FiYmNkN2M3YTM2NzkxNjI0OGUyOGJfYWZhMTk4ZWZhOTFkNjVlOGU3OGIwM2M5ZTZjN2FiYWVfSUQ6NzQ1MTUzNjExODc2MTU1MzkyNF8xNzgxNTM3MDc0OjE3ODE2MjM0NzRfVjM)

3、执行结果如下

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NDhlNGY1ZmZlYTM4OGIzMDhkNDBmMmVjYjgxMDIyYjNfYzVmNTY4N2M2YjAzMzM1NjBlOTMyMTc4YTNmZmYxOTFfSUQ6NzQ1MTUzNjEyMTYyMjU0NDM4N18xNzgxNTM3MDc0OjE3ODE2MjM0NzRfVjM)

## 使用fixture函数

### **测试用例的前置和后置**

- 此类fixture函数与setup/teardown的功能类似，即**数据准备与清理工作。**

- 此类fixture函数的复用性高，所以放在conftest\.py里，以实现多个测试模块**共享**fixture函数

- 通过`yield`关键字可以把**数据注入测试用例**。如下示例

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=ZTA2YzY1MjIxMWZkMWJkMmEyMDRlMmYyYWY1ZTBmN2FfZGRiOWQ1MmVmM2IwZGMzMjBmZmVjMzFkZjY3YzhmZWJfSUQ6NzQ0NDgwOTk5NjQzMDkwMTI1MV8xNzgxNTM3MDc0OjE3ODE2MjM0NzRfVjM)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MmNkYTgxNmFiYmRkOGQyNTAwNWUxZjZkNjIxNjc5N2ZfMDMxMzA5MTZkM2QxNTgwMTBhY2JkN2Y3NzlmNDdmZThfSUQ6NzQ0NDgxMjY1OTk4MTAwODkyNF8xNzgxNTM3MDc0OjE3ODE2MjM0NzRfVjM)

### 读取环境配置与初始化

- 此类fixture函数的scope一般会是全局执行，且不需要后置处理

- 如下代码示例，fixture函数`conf`的功能是根据host、stor参数，动态生成一份环境配置，供其它fixture函数使用。

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=MjNhYzM1Njk4OWUxNGZhY2RhMjFmYjM3ZDM1YTQyNzBfYzIwZTcwNjhkZjBmZTQ3Nzc4ZTM5MWI1OTZiODI2ZWRfSUQ6NzQ0NDgxODg3OTg4ODI2MTEyM18xNzgxNTM3MDc0OjE3ODE2MjM0NzRfVjM)

### 测试类/模块级别的初始化

- 此类fixture函数常声明于测试类和模块级别上使用，无需在每个测试方法参数中显式传递，确保在类或模块中的所有测试方法运行前先执行指定的 fixture

- 此类fixture函数一般不需要返回值。如示例代码

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NGI4MGZmYzFjNTcyYjFiZjQwNDExODhhZmUxMDY3MzFfYzhjMDAyZDRiM2UwM2NiNzU4MDUxOGMxYzk0YmUyOWNfSUQ6NzQ0NDgzMzMxNDA3NzA0ODg2MF8xNzgxNTM3MDc0OjE3ODE2MjM0NzRfVjM)

- 如果需要在类级别使用 `fixture` 的结果，可以参考如下两种方法实现

```Python
# test_example.py

@pytest.mark.usefixtures("ssh")
class TestSSHOperations:

    @pytest.fixture(autouse=True)
    def setup(self, ssh):
        self.ssh = ssh    # 通过 self.ssh 访问 ssh 实例

    def test_example1(self):
        output = self.ssh.run("uname -a")
        assert "Linux" in output

    def test_example2(self):
        output = self.ssh.run("hostname")
        assert output.strip() == "expected_hostname"
```

```Python
# test_example.py

@pytest.mark.usefixtures("ssh")
class TestSSHOperations:
    ssh = None

    @pytest.fixture(autouse=True)
    def setup(self, ssh):
        TestSSHOperations.ssh = ssh    # 存储 ssh fixture 的返回值

    def test_example1(self):
        output = TestSSHOperations.ssh.run("uname -a")
        assert "Linux" in output

    def test_example2(self):
        output = TestSSHOperations.ssh.run("hostname")
        assert output.strip() == "expected_hostname"
```

### fixture函数的参数化

- 通过 pytest 的参数化机制，可以动态生成不同的测试场景并将参数传递给 fixture

- 参数化的值通常从测试用例中传递给 fixture

- 每组参数会运行一次测试函数，并使用对应的参数。

```Python
import pytest

@pytest.fixture(scope='class')
def cce(tenant, ssh, request):
    # 参数准备
    params = getattr(request, 'param', {})
    name = random_data()
    flavor_name = params.get('flavor_name', 'cce.d6.large')  # 默认值
    password = params.get('password', 'admin1234@sugon')     # 默认值
    port = params.get('port', 22022)                        # 默认值

    try:
        flavor_id = get_id(tenant.flavor_get(name=flavor_name))
        assert_response(tenant.cce_create(tenant.id, name, flavor_id, tenant.net_id, tenant.subnet_id, tenant.volume_type, tenant.conf['cidr']))
        cce = tenant.assert_paas(name, interval=15)
        mfip = get_response(tenant.mfip_get(cce['vip']), 'mfip_address')[0]
        ssh.connect(mfip, pwd=password, port=port)
        cce['nodename'] = ssh.run("hostname")
        cce['mfip'] = mfip

        yield cce
    finally:
        tenant.assert_paas_deleted(name)


# 参数化的测试用例
@pytest.mark.parametrize(
    "cce",
    [
        {"flavor_name": "cce.d6.large", "password": "admin1234@sugon", "port": 22022},
        {"flavor_name": "cce.d6.xlarge", "password": "root@password", "port": 2222},
    ],
    indirect=True,  # 表示 pytest 会将参数传递给 cce fixture，而不是直接传递给测试函数
)
def test_cce_functionality(cce):
    assert cce['mfip'] is not None
    assert cce['nodename'] != ""
```

## pytest\.ini 的应用

`pytest.ini` 放在项目的根目录下。如果 `pytest.ini` 有该参数值，在执行的时候，先读取配置文件中的参数，然后取其他地方的（主函数/命令行中）。

`pytest.ini`可用于多种场景，包括但不限制于：

- **预设命令行参数**：设置运行测试时默认使用的命令行参数。

- **自定义测试发现规则**：定义`pytest`查找测试用例时的文件模式和Python类、函数命名规则。

- **增加自定义标记**：定义新的标记（markers），为测试用例添加额外的元数据。

- **日志配置**：设定测试中的日志记录行为。

- **修改插件行为**：添加或修改特定`pytest`插件的配置规则。



