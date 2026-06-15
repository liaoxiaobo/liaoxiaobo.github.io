# Selenium

# Selenium

**Selenium用法** 主要就4步：

1. 创建WebDriver对象；

2. 用WebDriver对象的**get方法**打开网址；

3. 用WebDriver对象的**find\_element方法**去找到网页上你需要的元素，返回WebElement对象（找到对象是重点）；

4. 对WebElement对象进行操作，一般就是获取对象的内容、填写入内容或者click操作。



**find\_element 和 find\_elements 的区别**

- find\_elements：找出符合条件的所有元素， 如果没有，返回空列表

- find\_element：找出符合条件的第一个元素，如果没有，抛出NoSuchElementException 异常

## docker\-selenium

docker\-selenium项目（ [镜像仓库](https://hub.docker.com/u/selenium/) , [代码仓库](https://github.com/SeleniumHQ/docker-selenium/) ）是将 selenium、webdriver、VNC server、chrome（或者firefox）集成在一个docker镜像里的项目。提供如下的功能：

- 代替原有的 remote webdriver

- 单个容器就能提供全套 selenium\+webdriver\+headless 浏览器的功能

- 几个容器配合就能完全代替 selenium grid

- 包含 VNC server（远程桌面），方便远程调试 headless 浏览器

- 全部在 linux 环境下执行，无需设置 windows 节点机，方便自动化

- 方便自定义 Dockerfile ，用户可以自己制作镜像

### **Standalone**

`docker run -d -p 4444:4444 -p 7900:7900 --shm-size="2g" -e SE_NODE_MAX_SESSIONS=10 -e SE_NODE_SESSION_TIMEOUT=3600 selenium/standalone-chrome:4.10.0-20230607`

### **Hub and Nodes**

`docker compose up -d`

```YAML
# To execute this docker-compose yml file use `docker-compose -f docker-compose-v2.yml up`
# Add the `-d` flag at the end for detached execution
# To stop the execution, hit Ctrl+C, and then `docker-compose -f docker-compose-v2.yml down`
version: '2'
services:
  chrome:
    image: selenium/node-chrome:4.10.0-20230607
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
    ports:
      - "5901:5900"    # VNC客户端使用
      - "7901:7900"    # noVNC使用

  edge:
    image: selenium/node-edge:4.10.0-20230607
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
    ports:
      - "5902:5900"
      - "7902:7900"

  firefox:
    image: selenium/node-firefox:4.10.0-20230607
    shm_size: 2gb
    depends_on:
      - selenium-hub
    environment:
      - SE_EVENT_BUS_HOST=selenium-hub
      - SE_EVENT_BUS_PUBLISH_PORT=4442
      - SE_EVENT_BUS_SUBSCRIBE_PORT=4443
    ports:
      - "5903:5900"
      - "7903:7900"

  selenium-hub:
    image: selenium/hub:4.10.0-20230607
    ports:
      - "4442:4442"
      - "4443:4443"
      - "4444:4444"

```

### Selenium Grid

1. **docker\-compose创建**

按照如上yaml内容创建容器

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=NjVmMGM1MjRkYjliODI5YmUzN2NjNzkwNDA3YTQ3YmRfNGJkOWRmZTM3YmFkZTg5NzMwM2RiMTE2NjNiYTAzNDdfSUQ6NzQxMjMxOTY4NzgwOTUzMTkwN18xNzgxMzc0Nzk1OjE3ODE0NjExOTVfVjM)

服务起来了，可以在浏览器中查看，结果如下：

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=OThlMmFhMjljYTNmYmMyMzBlN2RkM2VkMjlmNzM1YWRfN2ZmMzEyYzc0MzZlYmE2YTRlNDQyN2ZhYTcwNGNmNGVfSUQ6NzQxMjMxOTY4ODU3MjYxNjcwOF8xNzgxMzc0Nzk1OjE3ODE0NjExOTVfVjM)

点开版本号上面的1，可以看到有如下信息（这些信息有用的，记一下，代码里会用到这些参数）：

```YAML
{"browserName":"firefox","browserVersion":"114.0","platformName":"linux","se:noVncPort":7900,"se:vncEnabled":true}
```

2. 使用Python调用Selenium Grid

跟平常的Selenium WebDriver一样的，唯一区别是需要通过`remote`方法去调用selenium\_grid地址

```Python
selenium_grid_url = 'http://172.22.12.241:4444/wd/hub'
option = webdriver.ChromeOptions() 
wd = webdriver.Remote(command_executor=selenium_grid_url, options=option)
# wd = webdriver.Chrome(options=option)
```

3. 远程观看

**VNC客户端**

vnc viewer是一款优秀的远程控制工具软件。
官网下载地址：[https://www\.realvnc\.com/en/connect/download/viewer/](https://www.realvnc.com/en/connect/download/viewer/)

安装好以后 File\-\>New connection，在弹出的界面中输入node的ip和端口号（比如我这就是43\.142\.94\.65:5900，Name随意），保存后双击启动。
连接，会要求输入密码，默认密码：secret

连接上后，再运行Python selenium的代码，就能看到运行过程了。

**使用浏览器（noVNC）**

http://\{服务器ip\}:7900

默认密码：secret



## 常见问题

### 测试报告如何截图

错误用例截图代码示例：

[pytest进阶之html测试报告 \- linux超 \- 博客园](https://www.cnblogs.com/linuxchao/p/linuxchao-pytest-report.html)

### 处理 element click intercepted 异常

[selenium\.common\.exceptions\.ElementClickInterceptedException: Message: element click intercepted](https://www.zhihu.com/tardis/zm/art/626704447?source_id=1003)

![Image](https://internal-api-drive-stream.feishu.cn/space/api/box/stream/download/authcode/?code=YmNhYWJlMjZjYmM1Y2U0N2QxNGRkMjI5YjQ0MTFlMGRfYmFhMDIyNDFmNzA5YjViYmY3ZGZjMmUyZmE5YzkwMDFfSUQ6NzQxMjMxOTY4ODk5MTk5Nzk4MF8xNzgxMzc0Nzk1OjE3ODE0NjExOTVfVjM)

### **处理 NoSuchElementException 的异常**

因为有2种情况，都会有有这个异常：

- 情况一：找不到此元素

- 情况二：代码执行速度比网站响应快

1\)\.如果是第二种情况

虽然用了 `wd.implicitly_wait(10)` ，但有时候还是会有这个问题。

我一般这样处理这个异常，get方法之后sleep一下（时间不是固定，看网络和站点的响应情况），然后如果报这个异常，就再sleep多等待一下再查找这个元素。

```Python
wd.get(site_url)
time.sleep(2) # 等待2秒
try:
    element = wd.find_element(By.ID,'search_icon')
except selenium.common.exceptions.NoSuchElementException:
    time.sleep(5) # 等待5秒
    element = wd.find_element(By.ID,'search_icon')
```

2\)\.如果是第一种情况

就直接在语句后面加个 if\(element\)的判断好了，看看false的时候（没找到元素）需要做什么处理或者提示。





