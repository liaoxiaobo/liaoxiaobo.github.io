# Pytest框架分享

# Pytest框架

## Pytest特点

pytest是一个非常成熟的全功能的Python测试框架，主要特点有以下几点：

- 简单灵活，容易上手，文档丰富；

- 支持参数化，可以细粒度地控制要测试的测试用例；

- 能够支持简单的单元测试和复杂的功能测试，还可以用来做selenium/appnium等自动化测试、接口自动化测试（pytest\+requests）;

- pytest具有很多第三方插件，并且可以自定义扩展，比较好用的如pytest\-selenium（集成selenium）、pytest\-html（完美html测试报告生成）、pytest\-rerunfailures（失败case重复执行）、pytest\-xdist（多CPU分发）等；

- 测试用例的skip和xfail处理；

- 可以很好的和CI工具结合，例如jenkins

## 用例编写原则

1. 文件名以 test\_\_\.py 文件和\_\_test\.py

2. 以 test\_ 开头的函数

3. 以 Test 开头的类，不能包含 **init** 方法

4. 以 test\_ 开头的类里面的方法

5. 所有的包 pakege 必须要有\_\_init\_\_\.py 文件

## pytest参数用法

```python
pytest 文件名.py                            # 执行单独一个pytest模块
pytest 文件名.py::类名::方法名               # 运行某个模块里面某个类里面的方法
pytest 文件名.py::类名                      # 运行某个模块里面某个类
pytest -k "add"                            # 匹配名称中包含add的用例,可以使用and、or、not等逻辑运算 
pytest -k "not add"                        # 匹配名称中不包含add的用例
pytest -m ［标记名］                        # @pytest.mark.［标记名］将运行有这个标记的测试用例                      

pytest -q                                  # 只打印测试结果
pytest -s                                  # 带控制台输出结果.也是输出详细
pytest -v                                  # 打印详细运行日志信息
pytest -x 文件名.py                         # 一旦运行到报错就停止运行
pytest --maxfail=［num］                    # 当运行错误达到num的时候就停止运行
```

## mark用法

### 自定义mark

pytest\.mark主要是用来对test方法进行标记用的一个装饰器。标记的作用就是在使用pytest跑测试代码的时候可以选择性地执行部分test方法。

```python
# test_with_mark.py 

@pytest.mark.finished
def test_func1():
    assert 1 == 1


@pytest.mark.unfinished
def test_func2():
    assert 1 != 1
```

```python
$ pytest -m finished test_with_mark.py
============================= test session starts =============================
platform win32 -- Python 3.6.4, pytest-3.6.1, py-1.5.2, pluggy-0.6.0
rootdir: F:\self-repo\learning-pytest, inifile:
collected 2 items / 1 deselected

tests\test-function\test_with_mark.py .                                  [100%]

=================== 1 passed, 1 deselected in 0.10 seconds ====================
```

### 内置mark函数

```python
skip — pytest执行时会跳过所在的test方法。
skipif — 传递一个条件判断式，满足时会跳过所在方法。
xfail — 如果test fail了则认为是pass的，反之亦然。
parametrize — 给test方法添加参数，供跑测试时填充到test方法中。这个参数可以是多组参数组成的列表，并且可以用多个pytest.mark.parametrize装饰器来装饰test方法，pytest会组合所有的参数可能性来执行test方法；如果parametrize的参数名称和fixture名称一样，会覆盖掉fixture。
usefixtures — 对给定test方法执行给定的fixtures（和直接用fixture一样，只不过不需要把fixture名称作为参数放在方法声明当中，并且可以对class使用（fixture暂时不能用于class））。
tryfirst — 使所在test方法可以尽早地被执行（实际情况下如果有fixture的parametrize，执行顺序会比较复杂）。
trylast — 和上面相反。
```

#### Parametrize：参数化数据驱动

测试代码如下：

```python
import pytest
from resource.user import User

class TestLogin:

  # 装饰器函数实现参数化用例，test方法会执行两遍
  @pytest.mark.parametrize('username',['admin','admin@wise2c.com'])
  @pytest.mark.parametrize('password',['Wise2c2019'])
  def testlogin_success(self,username,password):
    resp = User().user_login(username,password)
    assert resp.status_code == 200

  # 列表里有三对元祖，test方法会执行三次
  @pytest.mark.parametrize('username,password',[('wrong','Wise2c2019'),
                          ('admin','wise2c2019'),
                          ('wrong','wise2c2019')])
  def testlogin_failed(self,username,password):
    resp = User().user_login(username,password)
    assert resp.status_code == 200
    assert resp.json()['errorMsg'] == '用户名或密码错误'
```

结果输出：

```python
============================= test session starts ==============================
platform darwin -- Python 3.7.2, pytest-5.3.5, py-1.8.1, pluggy-0.13.1 -- /Users/liaoxb/Tools/virtualenv-3.7/bin/python
cachedir: .pytest_cache
metadata: {'Python': '3.7.2', 'Platform': 'Darwin-18.2.0-x86_64-i386-64bit', 'Packages': {'pytest': '5.3.5', 'py': '1.8.1', 'pluggy': '0.13.1'}, 'Plugins': {'html': '2.0.1', 'allure-pytest': '2.8.9', 'rerunfailures': '8.0', 'xdist': '1.31.0', 'forked': '1.1.3', 'metadata': '1.8.0'}}
rootdir: /Users/liaoxb/Tools/pycharm-workspace/wisecloud/apicase
plugins: html-2.0.1, allure-pytest-2.8.9, rerunfailures-8.0, xdist-1.31.0, forked-1.1.3, metadata-1.8.0
collecting ... collected 5 items

test_login.py::TestLogin::testlogin_success[Wise2c2019-admin] 
test_login.py::TestLogin::testlogin_success[Wise2c2019-admin@wise2c.com] 
test_login.py::TestLogin::testlogin_failed[wrong-Wise2c2019] 
test_login.py::TestLogin::testlogin_failed[admin-wise2c2019] 
test_login.py::TestLogin::testlogin_failed[wrong-wise2c2019] 

============================== 5 passed in 1.25s ===============================
```

#### usefixtures

对给定test方法执行给定的fixtures（和直接用fixture一样，只不过不需要把fixture名称作为参数放在方法声明当中，并且可以对class使用（fixture暂时不能用于class））。

## Pytest fixtures

Fixture是一些函数，pytest会在执行测试用例之前（或之后）加载运行它们。

- fixtures具有明确的名称,测试用例可以通过在其参数中使用fixtures名称来接收fixture对象。

- fixtures以模块化方式实现,因为每个fixture名称都会触发调用fixture函数,该fixture函数本身可以使用其它的fixtures。

- fixtures允许测试用例能轻松引入预先定义好的初始化准备函数,而无需关心导入/设置/清理方法的细节。 其中fixture函数扮演”注入器“的角色,测试用例来“消费”这些fixture对象。

- fixture可以直接定义在测试脚本中。但更多时候，Pytest使用文件 conftest\.py 集中管理fixture。

- fixture函数的发现顺序从测试类开始,然后是测试模块,然后是conftest\.py文件,最后是内置和第三方插件。

如果要使用数据文件中的测试数据,最好的方法是将这些数据加载到fixture函数中以供测试用例注入使用。这利用到了pytest的自动缓存机制。另一个好方法是在tests文件夹中添加数据文件。 还有社区插件可用于帮助处理这方面的测试,例如：pytestdatadir和pytest\-datafiles。
\</quote\-container\>

可以认为测试分为四个步骤：

1. **Arrange**

2. **Act**

3. **Assert**

4. **Cleanup**

**Arrange** 是我们为测试做准备的地方。即前置数据

**Act** 是启动 **行为** 我们想测试一下。这一行为实现了被测系统\(SUT\)状态的改变，也是我们可以查看改变后的状态，以便我们对行为做出判断。这通常采用函数/方法调用的形式。

**Assert**是我们观察结果状态的地方。这是我们收集证据来证明行为是否符合我们预期的地方。这个 `assert` 在我们的测试中，我们在那里进行测量/观察，并对其应用我们的判断。

**Cleanup**是清理测试过程中的数据，从而保证其他测试不会意外地受到它的影响。
\</quote\-container\>

### 作用域

在定义fixture时，通过scope参数声明作用域共享fixture。可选项有：

function: 函数级（默认范围），每个测试函数都会执行一次fixture函数；

class: 类级别，每个测试类执行一次，所有方法都可以复用；

module: 模块级，每个模块执行一次，模块内函数和方法都可复用；

session: 会话级，一次测试只执行一次，所有被找到的函数和方法都可复用。

### 自动执行

目前为止，所有fixture的使用都是手动指定，或者作为参数，或者使用 usefixtures。如果我们想让fixture自动执行，可以在定义时指定 autouse 参数。

### 重命名

fixture的名称默认为定义时的函数名，如果不想使用默认，可以通过 name 选项指定名称：

```python
# test_rename.py

@pytest.fixture(name='age')
def calculate_average_age():
    return 28

def test_age(age):
    assert age == 28
```

### 参数化

可以对Fixture方法函数进行参数化,在这种情况下,它们将被多次调用,每次执行一组相关测试,即依赖于此Fixture方法的测试。Fixture方法参数化有助于为可以以多种方式配置的组件编写详尽的函数测试。

```python
@pytest.fixture(params=[
    ('redis', '6379'),
    ('elasticsearch', '9200')
])
def param(request):
    return request.param

@pytest.fixture(autouse=True)
def db(param):
    print('\nSucceed to connect %s:%s' % param)

    yield

    print('\nSucceed to close %s:%s' % param)

def test_api():
    assert 1 == 1
```

执行结果：

```python
$ pytest -s tests/fixture/test_parametrize.py
============================= test session starts =============================
platform win32 -- Python 3.6.4, pytest-3.6.1, py-1.5.2, pluggy-0.6.0
rootdir: F:\self-repo\learning-pytest, inifile:
collected 2 items

tests\fixture\test_parametrize.py
Succeed to connect redis:6379
.
Succeed to close redis:6379

Succeed to connect elasticsearch:9200
.
Succeed to close elasticsearch:9200

========================== 2 passed in 0.10 seconds ===========================
```

### yield

Pytest 使用 `yield` 关键词将fixture分为两部分，`yield` 之前的代码属于预处理，会在测试前执行；`yield` 之后的代码属于后处理，将在测试完成后执行。

```python
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

我们还可以使用with语句无缝地使用yield语法，测试结束后,smtp\_connection连接将关闭,因为当with语句结束时,smtp\_connection对象会自动关闭。

```python
# conftest.py文件内容
import smtplib
import pytest

@pytest.fixture(scope="module")
def smtp_connection():
    with  smtplib.SMTP("smtp.gmail.com",587,timeout=5) as  smtp_connection:
    yield  smtp_connection # provide the fixture value 
```

请注意,如果在setup代码期间\(yield关键字之前\)发生异常,则不会调用teardown代码\(在yield之后\)。 执行teardown代码的另一种选择是利用请求上下文对象的addfinalizer方法来注册teardown函数。 以下是smtp\_connection fixture函数更改为使用addfinalizer进行teardown：

```python
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

yield和addfinalizer方法在测试结束后调用它们的代码时的工作方式类似,但addfinalizer相比yield有两个主要区别：1\. 使用addfinalizer可以注册多个teardown函数。

1. 无论fixture中setup代码是否引发异常,都将始终调用teardown代码。 即使其中一个资源无法创建/获取,也可以正确关闭fixture函数创建的所有资源：

## 测试报告

### pytest\-html

pytest是借助pytest\-html插件生成测试报告，不用自己编写生成报告代码。

```python
pip install pytest-html 
pytest --html=reportname.html test_user.py
```

错误用例截图和添加描述的代码示例： [https://www\.cnblogs\.com/linuxchao/p/linuxchao\-pytest\-report\.html](https://www.cnblogs.com/linuxchao/p/linuxchao-pytest-report.html)

### allure2生成报告

```python
pip install allure-pytest
nohup pytest -s --alluredir ./allure-result &
# 生成测试结果json格式,指定存放在当前目录allure-result文件夹下

allure generate allure-result/ -o ./allure-report -c
# json格式测试报告转为html格式的Allure测试报告，指定输出到allure-report文件夹里

allure serve allure-result/ 
nohup allure serve allure-result -h 10.11.10.137 -p 1688 &
# 另外一种办法，在cmd命令行中执行 allure serve 测试结果目录名，就会生成allure报告
```

[https://blog\.51cto\.com/king15800/3089636](https://blog.51cto.com/king15800/3089636)

# 使用指南

## jenkins集成allure测试报告

1、安装allure插件

2、系统管理\-全局工具配置，新增Allure Commandline

3、使用jenkinsfile创建流水线，pipeline配置内容如下，根据测试结果目录生成report目录。

4、查看测试报告

## 环境巡检的应用场景

- 环境重新部署后，对环境做基本检查\+测试数据初始化

- 测试活动期间，测试人员需要对接某个存储池、上传测试镜像（环境按需对接）

- 研发更新环境后，需要快速跑一次巡检，确认是否引入了新的致命问题（类似于冒烟、提测卡点机制）

- 高可用测试，可以利用环境巡检做一些辅助验证的工作

## 如何执行全量回归测试

1、类似于测试用例，**自动化测试也是需要前置条件的**，如下

- 环境已导入许可，许可导入方法参考 [测试环境部署指南](http://172.22.5.6:8090/pages/viewpage.action?pageId=37486661)

- 环境预检流水线执行通过，确保测试环境正常，且已成功对接存储池

- Autotest项目配额足够（至少需要50C、100GB、系统盘2T、数据盘2T）

- jenkins虚机和测试环境网络互通

2、选择**目标测试环境**\-\>**存储池类型→**点击 **build**即可运行，总共需要执行 **40min\-60min**

\*\*关于选项的几点说明\*\*

- host、stor下拉选项值是可以按需配置的，点击左侧菜单→配置，修改后保存即可

- 选择不同的stor值，需要提前手工对接第三方存储的方法会有不同，具体参考如下

    - **xstor**：全自动，无需手动对接

    - **ceph**：需要手工创建存储池，且名字固定为 ceph\-test（若无xsky环境，需要根据[部署文档](https://xyzlabs.feishu.cn/file/JgH0bqn1Bo8VGHxmTWXcWy2dnbe)步骤提前部署）

    - **zbs**：需要手工创建存储池，且名字固定为 zbs\-test （若无zbs环境，需要根据部署文档步骤提前部署）

    - **local**：在运维\-存储\-本地存储页面，手工创建本地存储（无需创存储池），且名字固定为 local\-test

    - **usan**：在运维\-存储\-usan管理页面，手工创建usan存储（无需创存储池），且名字固定为 usan\-test （若无usan环境，需要根据[部署文档](http://172.22.5.6:8090/pages/viewpage.action?pageId=86674438)提前部署）

- 以上需要手工创建的存储池，设置存储容量时，建议**系统盘、数据盘容量分配2T以上**

3、自动化结束后，会通过飞书群推送测试结果和测试报告，点击链接可以查看报告详情

## 如何只运行上一次失败的用例

执行时，勾选RUN\_LAST\_FAILED 即可

## 如何保证自动化测试执行时的稳定性

1. 自动化运行期间，保持环境稳定。不做节点异常操作，不做同步存储池操作（同步存储池会重启gova、cinder等核心组件）

2. 不在Autotest项目下开展手工测试

3. 自动初始化的测试资源（主机集合、VPC网络、镜像、存储池），禁止去修改和删除

## 失败用例应该如何分析，谁去分析

**谁去分析：**

执行自动化的人员和写测试代码的可能不是同一个人，所以可以优先把失败用例转给模块接口人去定位分析

模块接口分工（暂定）：

```
计算&存储：廖小波
网络模块：齐春华、任新双
```

**如何分析：**

**只有三种根因会导致用例失败：产品缺陷、用例问题、环境问题**

1、测试报告\-点击查看测试用例结果详情，定位到测试用例的失败点在哪（通常会是在断言处失败）

2、结合测试报告提供的log内容、接口响应内容，进一步分析原因

3、结合用例的执行上下文，进一步判断是否是其它用例对其产生了影响

4、（可选）本地IDE调试运行该用例、或对失败的用例全部重新执行一次，以判断是否是偶现问题

## 可以在一套环境并行跑多个回归任务吗

可以，但是不建议超过两个以上的并行任务。且相互可能有影响，比如存储池同步配置无法保证是同一时刻执行，那就会导致接口大面积失败。

> (注：内容由 AI 生成，请谨慎参考）
