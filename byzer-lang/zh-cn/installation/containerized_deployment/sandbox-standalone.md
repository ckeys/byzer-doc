# Sandbox 独立部署

Sandbox 包含了 Byzer 两大组件 Byzer Notebook 和 Byzer Lang，您可以快速体验 Byzer 功能。

### 前置条件

#### 安装 Docker Desktop

Docker 桌面版是一个适用于 MacOS 和 Windows 机器的应用程序，用于构建和共享容器化应用程序和微服务。它提供
了非常丰富便捷的管理平台，方便我们快速部署、管理 Sandbox 镜像和容器。

从 [Docker 官网](https://www.docker.com/products/docker-desktop) 下载适配您操作系统的安装包，安装并使用。

适用于 Linux 的 Docker 桌面社区还在开发中，可以参考 [Docker 官网安装文档](https://docs.docker.com/engine/install/ubuntu/)
在 Linux 上安装 Docker 引擎。

### Sandbox 独立部署 Byzer

下载 Byzer docker repo 最新的image（基于 Spark 3）：

```shell
docker pull byzer/byzer-sandbox:3.1.1-latest
```

使用 docker 命令启动 Spark 3.1.1版 Sandbox 容器:

```shell
docker run -d \
-p 3306:3306 \
-p 9002:9002 \
-p 9003:9003 \
-e MYSQL_ROOT_HOST=% \
-e MYSQL_ROOT_PASSWORD=root \
--name sandbox-3.1.1-2.2.0-SNAPSHOT \
--restart=always \
byzer/byzer-sandbox:3.1.1-2.2.0-SNAPSHOT
```

使用 docker 命令启动 Spark 2.4.3版 Sandbox 容器:

```shell
docker run -d \
-p 3306:3306 \
-p 9002:9002 \
-p 9003:9003 \
--name sandbox-2.4.3-2.2.0-SNAPSHOT \
--restart=always \
-e MYSQL_ROOT_HOST=% \
-e MYSQL_ROOT_PASSWORD=root \
byzer/byzer-sandbox:2.4.3-2.2.0-SNAPSHOT
```

> 请注意，若启动容器时，拉取镜像超时，您只需启动一个 Sandbox 镜像。


### 体验 Byzer 功能

浏览器[登录](http://localhost:9002)，在用户注册界面，输入用户名和密码，点击 **Create a New Notebook - Create**，进入 Notebook 使用界面，贴入如下代码：

#### 使用 Python 和 Ray 处理 JSON 数据

请执行代码:

```sql
-- 构造测试数据
set mockData='''

{"title":"第一","body":"内容 1"}
{"title":"第二","body":"内容 2"}
{"title":"第三","body":"内容 3"}

''';

load jsonStr.`mockData` as data;

-- 设置 Python 环境 
!python env "PYTHON_ENV=:";
!python conf "runIn=driver";
!python conf "schema=st(field(title,string),field(body,string))";
!python conf "dataMode=data";

-- Python 代码在 Sandbox 内置的 Ray 上执行
!ray on data '''

import ray
from pyjava.api.mlsql import RayContext
import numpy as np;

ray_context = RayContext.connect(globals(),"localhost:10001")

def echo(row):
    row1 = {}
    row1["title"]="jackm"
    row1["body"]= row["body"]
    return row1

ray_context.foreach(echo)

''' named newdata;

select * from newdata as output;
```

结果如下:

  <img src="/byzer-lang/zh-cn/installation/containerized_deployment/images/python-ray-result.PNG" alt="Python-Ray 结果"/>

#### 处理MySQL 数据

 ```sql
-- 加载 mlsql_console.mlsql_job 表数据
load jdbc.`usage_template` where url="jdbc:mysql://localhost:3306/notebook?characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&tinyInt1isBit=false"
 and driver="com.mysql.jdbc.Driver"
 and user="root"
 and password="root"
 as table1;
 
-- 查询100条
select * from table1 limit 100 as table2;

-- 保存到DeltaLake
save append table2 as delta.`dt1`;

-- 查询 DeltaLake 
load delta.`dt1` as table3;
 
select * from table3 as table4;
 ```

结果如下:

  <img src="/byzer-lang/zh-cn/installation/containerized_deployment/images/mysql-deltalake.PNG" alt="MySQL-deltalake"/>