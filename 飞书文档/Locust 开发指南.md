# Locust 开发指南

# Locust

## 安装locust

```Python
#安装python pip
sudo yum -y install python-pip
#通过Python自带的pip安装locust
pip install locustio
#查看locust版本
localhost:locust liaoxb$ locust --version
[2019-04-27 12:46:27,198] localhost/INFO/stdout: Locust 0.11.0
[2019-04-27 12:46:27,198] localhost/INFO/stdout:
```

## 参数说明

运行参数说明

```Python
--host      指定被测试的主机，采用以格式：http://192.168.21.25
-f          指定运行 Locust 性能测试文件，默认为: locustfile.py
–-headless  no-web 模式运行测试，需要 --user 和 -r 配合使用
--user      指定并发用户数，作用于headless模式。
-r          指定每秒启动的用户数，作用于headless模式。
-t          设置运行时间, 例如： (300s, 20m, 3h, 1h30m). 作用于headless模式
```

返回结果说明

```Python
Name：请求方式，请求路径；
reqs：当前请求的数量；
fails：当前请求失败的数量；
Avg：所有请求的平均响应时间，毫秒；
Min：请求的最小的服务器响应时间，毫秒；
Max：请求的最大服务器响应时间，毫秒；
Median：中间值，单位毫秒；
req/s：每秒钟请求的个数。
Total：各接口的汇总信息
```

## 运行locust脚本

locust脚本内容

```Python
from locust import HttpLocust, TaskSet, task

username = "test"
password = "Wise2c2019"

class UserBehavior(TaskSet):
    def on_start(self):
        """ on_start is called when a Locust start before any task is scheduled """
        r = self.client.post("/api/users/login", json={"username": username, "password": password})
        print("Response content:", r.json())
        global headers
        headers = {"Authorization": 'Bearer ' + r.json()['data']['token']}

    def on_stop(self):
        """ on_stop is called when the TaskSet is stopping """
        print("压测结束")

    @task(1)
    def team(self):
        self.client.get("/api/teams", headers=headers)

class WebsiteUser(HttpLocust):
    task_set = UserBehavior
    # min_wait = 1000
    # max_wait = 5000
```

locust命令执行

```Shell
localhost:locust liaoxb$ locust -f example.py  --host=http://139.159.228.59:8080 -c 200 -r 10 -t 60s --no-web

[2019-04-27 13:05:27,820] localhost/INFO/locust.main: Running teardowns...
 Name                                                          # reqs      # fails     Avg     Min     Max  |  Median   req/s
--------------------------------------------------------------------------------------------------------------------------------------------
 GET /api/teams                                                  3301     0(0.00%)     121      49    5521  |      63   66.60
 POST /api/users/login                                            200     0(0.00%)     201      94    1285  |     110    0.00
--------------------------------------------------------------------------------------------------------------------------------------------
 Total                                                           3501     0(0.00%)                                      66.60

Percentage of the requests completed within given times
 Name                                                           # reqs    50%    66%    75%    80%    90%    95%    98%    99%   100%
--------------------------------------------------------------------------------------------------------------------------------------------
 GET /api/teams                                                   3301     63     66     70     74    180    500    910   1100   5500
 POST /api/users/login                                             200    110    120    270    280    460    630    870   1200   1300
--------------------------------------------------------------------------------------------------------------------------------------------
 Total                                                            3501     63     68     74     83    220    500    900   1100   5500
```

```Shell
(Virtualenv-3.7) localhost% locust -f locustdemo/app.py -c 100 -r 1 -t 10 --no-web

Name                                                           # reqs    50%    66%    75%    80%    90%    95%    98%    99%   100%
--------------------------------------------------------------------------------------------------------------------------------------------
 POST /api/users/login                                              10    120    130    130    130    140    140    140    140    140
 GET /orchestration/api/1/application                               54     64     66     69     72     77     96    100    100    100
--------------------------------------------------------------------------------------------------------------------------------------------
 Aggregated                                                         64     66     69     77     96    120    130    130    140    140
```

理解下这个场景：
总共准备了100个并发用户，每秒只启动一个用户，总执行10秒时间。
所以统计显示只有10个用户登录了，并且10个用户总共只发出了54个请求（1\+2\+3\+\.\.\.\+9\+10）。
注：此时设置的min\\max\_wait值是1秒

