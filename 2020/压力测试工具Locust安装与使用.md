# 压力测试工具Locust安装与使用

## 安装&启动服务

镜像下载&容器创建，端口为8089

```
docker run -dit -p 8089:8089 --volume=/opt/locust:/mnt/locust -t locustio/locust -f /mnt/locust/locustfile.py --master --web-host=0.0.0.0
```

访问地址：http://192.168.126.207:8089 



命令行方式执行
```
locust -f locustfile.py --host=http://192.168.122.46 --headless -u 1
```

以工作进程方式启动，进程数量与CPU核心数量相同

```
docker exec -it 080 /bin/bash
cd /mnt/locust
locust -f locustfile.py --worker &
```


## 编写测试脚本

测试脚本映射目录为/opt/locust，脚本名称locustfile.py

```
cd /opt/locust
vi locustfile.py
```

locustfile.py示例：

```
from locust import HttpUser, between, task
import time, datetime,json,random


class WebsiteUser(HttpUser):
    wait_time = between(0, 0)
    
    def on_start(self):
        pass

    @task
    def synorder(self):
        t = time.time()
        urid = str(int(round(t * 1000000)))
        header = {"Content-Type":"application/json"}
        payload = {
            'signType': '0',
            'msg': {
                'head': {
                    'clientId': '2020061118065636',
                    'clientSecret': '2020061118065630',
                    'templateId': 'hash',
                    'channelID': '',
                    'sysTag':'HIS'
                },
                'body': {
                    'openId': openid[random.randint(0,3200)],
                    'isForwardSign':'0' ,
                    'subject': '来自数字医信测试医院的测试订单-软路由',
                    'urId': urid,
                    'patientName': '李四',
                    'patientAge': '19',
                    'patientSex': '男',
                    'patientCardType': 'SF',
                    'patientCard': '520101198604085966',
                    'recipeTime': '2020-11-19 12:46:00',
                    'dataCategory':'dataCategory',
                    'hashValue': 'LM0WEHidhjstcPCvVVdEMHSK7lwAAAAAAAAAAAAAAAA=',
                    'hashType':'SHA-256'
                }
            }

        }
        req=self.client.post("/gateway/recipe/synRecipeInfo", json.dumps(payload), header)

```

## 执行测试

重启容器重新加载脚本

```
docker restart 080
```

访问地址：http://192.168.126.207:8089 

