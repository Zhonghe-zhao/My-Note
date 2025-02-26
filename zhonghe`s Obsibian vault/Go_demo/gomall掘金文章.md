
# 指令

杀死进程，查询进程

```shell
taskkill /PID 24352 /F  
成功: 已终止 PID 为 24352 的进程。
PS E:\go-zero-server\gonivinck\mall\service\user\rpc> netstat -ano | findstr :9000
  TCP    [::1]:9000             [::]:0                 LISTENING       31592
PS E:\go-zero-server\gonivinck\mall\service\user\rpc> taskkill /PID 31592 /F      
成功: 已终止 PID 为 31592 的进程。
```


`2379:2379` 表明容器内 2379 端口映射到主机 2379 端口。



---



https://juejin.cn/post/7036010157075071006

目前的问题是项目无法被构建！需要一个一个拉取镜像



各个容器的启动指令！

你可以使用 `docker run` 命令来启动你所需要的 Docker 镜像。以下是启动各个镜像的基本命令格式：

# 启动 `simplebank` 项目所需的 Docker 镜像

1. **Postgres**:
    
    ```bash
    docker run -d --name simplebank-postgres -e POSTGRES_PASSWORD=your_password -p 5432:5432 postgres:12-alpine
    ```
    
    这个命令会启动 Postgres 容器并设置 `POSTGRES_PASSWORD`，同时将 5432 端口映射到宿主机的 5432。
    
2. **MySQL**:
    
    ```bash
    docker run -d --name simplebank-mysql -e MYSQL_ROOT_PASSWORD=your_password -p 3306:3306 mysql:5.7
    ```
    
    启动 MySQL 容器，设置 `MYSQL_ROOT_PASSWORD` 环境变量，映射 3306 端口。
    
3. **Redis**:
    
    ```bash
    docker run -d --name simplebank-redis -p 6379:6379 redis
    ```
    
    启动 Redis 容器并将端口 6379 映射到宿主机。
    
4. **Grafana**:
    
    ```bash
    docker run -d --name simplebank-grafana -p 3000:3000 grafana/grafana
    ```
    
    启动 Grafana 容器并将端口 3000 映射到宿主机。
    
5. **Jaeger**:
    
    ```bash
    docker run -d --name simplebank-jaeger -p 5775:5775 -p 6831:6831/udp -p 6832:6832/udp -p 5778:5778 -p 16686:16686 -p 14268:14268 -p 14250:14250 -p 9411:9411 jaegertracing/all-in-one:1.28
    ```
    
    启动 Jaeger 容器并将多个端口映射到宿主机，确保 Jaeger 相关的所有端口都可访问。
    
6. **Prometheus**:
    
    ```bash
    docker run -d --name simplebank-prometheus -p 9090:9090 bitnami/prometheus
    ```
    
    启动 Prometheus 容器并将端口 9090 映射到宿主机。
    
7. **ETCD**:
    
    ```bash
    docker run -d --name simplebank-etcd -p 2379:2379 -p 2380:2380 bitnami/etcd
    ```
    
    启动 ETCD 容器并将端口 2379 和 2380 映射到宿主机。
    
8. **Dtm**:
    
    ```bash
    docker run -d --name simplebank-dtm -p 36789:36789 yedf/dtm
    ```
    
    启动 Dtm 容器并将端口 36789 映射到宿主机。

### 启动 `etcdkeeper` 容器

假设你已经有一个 `etcd` 服务运行在你的机器上（或者你也可以使用 Docker 启动 `etcd` 服务），你可以使用以下命令启动 `etcdkeeper` 容器：

bash

复制编辑

`docker run -d --name etcdkeeper \   -e ETCD_ENDPOINTS=http://<etcd_host>:<etcd_port> \   -p 4001:4001 \   evildecay/etcdkeeper`


### 如果没有 `etcd` 服务，你可以同时启动 `etcd` 容器和 `etcdkeeper` 容器：

1. 启动 `etcd` 容器：
    
    bash
    
    复制编辑
    
    `docker run -d --name etcd \   -p 2379:2379 \   -e ALLOW_NONE_AUTHENTICATION=yes \   quay.io/coreos/etcd:v3.4.0`
    
2. 启动 `etcdkeeper` 容器：
    
    bash
    
    复制编辑
    
    `docker run -d --name etcdkeeper \   -e ETCD_ENDPOINTS=http://etcd:2379 \   -p 4001:4001 \   evildecay/etcdkeeper`


### 访问 `etcdkeeper` Web 界面

- 打开浏览器并访问 `http://localhost:4001`，你应该能看到 `etcdkeeper` 的 Web 界面，从中可以浏览和管理 `etcd` 中的数据。

###  启动 phpMyAdmin 容器

执行以下命令，启动一个 `phpmyadmin` 容器并与 MySQL 数据库进行连接：

bash

复制编辑

`docker run -d --name phpmyadmin \   -e PMA_HOST=mysql_host \   -e PMA_PORT=3306 \   -p 8080:80 \   phpmyadmin/phpmyadmin`


提供的 Dockerfile 示例是可以正常工作的，但它比较简洁。根据你的需求，Dockerfile 可以根据不同的目的进行扩展。以下是一个基础的 `Dockerfile` 示例，适用于构建基于 Redis 镜像的容器：

    

### 启动项目镜像

如果你有 `gonivinck` 项目相关的镜像，可以通过如下命令启动：

```bash
docker run -d --name gonivinck-simplebank -p 8080:8080 simplebank:latest
```

这个命令会启动 `simplebank` 镜像并将其端口 8080 映射到宿主机的 8080 端口。

redis 启动
`docker run -d --name redis-container -p 6379:6379 redis:latest`


查看运行中的容器

你可以通过以下命令查看已启动的容器：

```bash
docker ps
```

 停止和删除容器

如果需要停止并删除某个容器，可以使用以下命令：

```bash
docker stop container_name
docker rm container_name
```


# 简单的 Dockerfile 示例：

```Dockerfile
**FROM redis:latest

LABEL maintainer="Ving <ving@nivin.cn>"

# 设置 Redis 密码 (如果需要)
RUN echo "requirepass yourpassword" >> /usr/local/etc/redis/redis.conf

# 通过修改配置文件来做更多定制化配置
# RUN sed -i 's/appendonly no/appendonly yes/' /usr/local/etc/redis/redis.conf

# 暴露端口
EXPOSE 6379

# 启动 Redis 服务并指定自定义配置文件
CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
**
```



镜像全部启动之后 

`docker-compose up -d `成功


![[gomall.png]]


### 容器报错应该学会看容器的日志报错

示例：
`docker logs redis-container`


### rpc项目启动指令：

`go run user.go -f etc/user.yaml`

# 学会在程序中排查问题

  

func main() {

    flag.Parse()

  

    // 加载配置文件

    var c config.Config

    //排查问题

    fmt.Println("Loading config file...")

    conf.MustLoad(*configFile, &c)

    fmt.Printf("Config loaded: %+v\n", c) // 打印出配置，检查是否正确加载

    ctx := svc.NewServiceContext(c)


```shell
 go run user.go -f etc/user.yaml
Loading config file...
Config loaded: {RestConf:{ServiceConf:{Name: Log:{ServiceName: Mode: Encoding: TimeFormat: Path: Level: MaxContentLength:0 Compress:false Stat:false KeepDays:0 StackCooldownMillis:0 MaxBackups:0 MaxSize:0 Rotation: FileTimeFormat:} Mode: MetricsUrl: Prometheus:{Host: Port:0 Path:} Telemetry:{Name: Endpoint: Sampler:0 Batcher: OtlpHeaders:map[] OtlpHttpPath: OtlpHttpSecure:false Disabled:false} DevServer:{Enabled:false Host: Port:0 MetricsPath: HealthPath: EnableMetrics:false EnablePprof:false HealthResponse:} Shutdown:{}} Host: Port:0 CertFile: KeyFile: Verbose:false MaxConns:0 MaxBytes:0 Timeout:0 CpuThreshold:0 Signature:{Strict:false Expiry:0s PrivateKeys:[]} Middlewares:{Trace:false Log:false Prometheus:false MaxConns:false Breaker:false Shedding:false Timeout:false Recover:false Metrics:false MaxBytes:false Gunzip:false} TraceIgnorePaths:[]} RpcConf:{ServiceConf:{Name: Log:{ServiceName: Mode: Encoding: TimeFormat: Path: Level: MaxContentLength:0 Compress:false Stat:false KeepDays:0 StackCooldownMillis:0 MaxBackups:0 MaxSize:0 Rotation: FileTimeFormat:} Mode: MetricsUrl: Prometheus:{Host: Port:0 Path:} Telemetry:{Name: Endpoint: Sampler:0 Batcher: OtlpHeaders:map[] OtlpHttpPath: OtlpHttpSecure:false Disabled:false} DevServer:{Enabled:false Host: Port:0 MetricsPath: HealthPath: EnableMetrics:false EnablePprof:false HealthResponse:} Shutdown:{}} ListenOn: Etcd:{Hosts:[] Key: ID:0 User: Pass: CertFile: CertKeyFile: CACertFile: InsecureSkipVerify:false} Auth:false Redis:{RedisConf:{Host: Type: Pass: Tls:false NonBlock:false PingTimeout:0s} Key:} StrictControl:false Timeout:0 CpuThreshold:0 Health:false Middlewares:{Trace:false Recover:false Stat:false StatConf:{SlowThreshold:0s IgnoreContentMethods:[]} Prometheus:false Breaker:false} MethodTimeouts:[]} Mysql:{DataSource:} CacheRedis:[] Salt: Auth:{AccessSecret: AccessExpire:0} UserRpc:{Etcd:{Hosts:[] Key: ID:0 User: Pass: CertFile: CertKeyFile: CACertFile: InsecureSkipVerify:false} Endpoints:[] Target: App: Token: NonBlock:false Timeout:0 KeepaliveTime:0s Middlewares:{Trace:false Duration:false Prometheus:false Breaker:false Timeout:false}} Mode: ListenOn:}
2025/01/26 22:20:38 no cache nodes
exit status 1
```


更改配置之后 

```go
package config

  

import (

    "github.com/zeromicro/go-zero/core/stores/cache"

    "github.com/zeromicro/go-zero/zrpc"

)

  

type Config struct {

    RpcServerConf zrpc.RpcServerConf `yaml:",inline"` // 使用 inline 标签来避免与配置文件中的字段冲突

  

    Mysql struct {

        DataSource string

    }

  

    CacheRedis cache.CacheConf

  

    Salt string

  

    Mode string `yaml:"Mode,default=dev"` // 指定默认值

  

    ListenOn string `yaml:"ListenOn"` // 指定默认值

} 
```

启动成功了

可以监听 

http://localhost:9090/query


启动api时遇到的问题

etcd需要设置环境变量 比如说密码：

但是我再docker-compose文件中看见了 etcd的相关配置有密码啊

```D

  etcd:                                  # 自定义容器名称

    build:

      context: ./etcd                    # 指定构建使用的 Dockerfile 文件

    environment:

      - TZ=${TZ}

      - ALLOW_NONE_AUTHENTICATION=yes

      - ETCD_ADVERTISE_CLIENT_URLS=http://etcd:2379

    ports:                               # 设置端口映射

      - "${ETCD_PORT}:2379"

    networks:

      - backend

    restart: always
```


重新拉取docker etcd镜像
`docker run -d -e ALLOW_NONE_AUTHENTICATION="yes" --name gomall-etcd quay.io/coreos/etcd:v3.5.15   `


# 问题

E:/go-zero-server/gonivinck/mall/service/user/api/user.go:27 +0x21f  
{"@timestamp":"2025-01-27T21:35:04.607+08:00","content":"rpc dial: etcd://localhost:2379/user.rpc, error: failed to exit idle mode: context deadline exceeded, make sure rpc service "user.rpc" is already started\n\ngoroutine 1 [running]:\nruntime/debug.Stack()\n\tD:/Go/src/runtime/debug/stack.go:24 +0x5e\ngithub.com/zeromicro/go-zero/core/logx.Must({0x1d71500?, 0xc000115ee0?})\n\tD:/gocode/pkg/mod/github.com/zeromicro/go-zero@v1.7.6/core/logx/logs.go:245 +0x4e\ngithub.com/zeromicro/go-zero/zrpc.MustNewClient({{{0xc000684bc0, 0x1, 0x1}, {0xc00070a9e0, 0x8}, 0x0, {0x0, 0x0}, {0x0, 0x0}, ...}, ...}, ...)\n\tD:/gocode/pkg/mod/github.com/zeromicro/go-zero@v1.7.6/zrpc/client.go:45 +0x65\nmall/service/user/api/internal/svc.NewServiceContext(...)\n\tE:/go-zero-server/gonivinck/mall/service/user/api/internal/svc/servicecontext.go:19\nmain.main()\n\tE:/go-zero-server/gonivinck/mall/service/user/api/user.go:27 +0x21f\n","level":"fatal"}  
exit status 1 这个错误我真的无法解决了，我使用docker-compose 启动了一些服务，但是后来我看应该时etcd的错误，我重新拉取了etcd镜像使用指令docker run -d -e ALLOW_NONE_AUTHENTICATION="yes" --name gomall-etcd quay.io/coreos/etcd:v3.5.15 但是api启动的时候就是报上面的错误，这是docker-compose文件你看看哪里有错误吗？version: '3.5'

networks:  
backend:  
driver: ${NETWORKS_DRIVER}
服务容器配置

services:  
golang: # 自定义容器名称  
build:  
context: ./golang # 指定构建使用的 Dockerfile 文件  
environment: # 设置环境变量  
- TZ={TZ} privileged: true volumes: # 设置挂载目录 - {CODE_PATH_HOST}:/usr/src/code # 引用 .env 配置中 CODE_PATH_HOST 变量，将宿主机上代码存放的目录挂载到容器中 /usr/src/code 目录  
ports: # 设置端口映射  
- "8000:8000"  
- "8001:8001"  
- "8002:8002"  
- "8003:8003"  
- "9000:9000"  
- "9001:9001"  
- "9002:9002"  
- "9003:9003"  
stdin_open: true # 打开标准输入，可以接受外部输入  
tty: true  
networks:  
- backend  
restart: always # 指定容器退出后的重启策略为始终重启

etcd: # 自定义容器名称  
build:  
context: ./etcd # 指定构建使用的 Dockerfile 文件  
environment:  
- TZ={TZ} - ALLOW_NONE_AUTHENTICATION=yes - ETCD_ADVERTISE_CLIENT_URLS=http://etcd:2379 ports: # 设置端口映射 - "{ETCD_PORT}:2379"  
networks:  
- backend  
restart: always

mysql:  
build:  
context: ./mysql  
environment:  
- TZ=TZ−MYSQLUSER=TZ−MYSQLU​SER={MYSQL_USERNAME} # 设置 Mysql 用户名称  
- MYSQL_PASSWORD={MYSQL_PASSWORD} # 设置 Mysql 用户密码 - MYSQL_ROOT_PASSWORD={MYSQL_ROOT_PASSWORD} # 设置 Mysql root 用户密码  
privileged: true  
volumes:  
- {DATA_PATH_HOST}/mysql:/var/lib/mysql # 引用 .env 配置中 DATA_PATH_HOST 变量，将宿主机上存放 Mysql 数据的目录挂载到容器中 /var/lib/mysql 目录 ports: - "{MYSQL_PORT}:3306" # 设置容器3306端口映射指定宿主机端口  
networks:  
- backend  
restart: always

redis:  
build:  
context: ./redis  
environment:  
- TZ=TZprivileged:truevolumes:−TZprivileged:truevolumes:−{DATA_PATH_HOST}/redis:/data # 引用 .env 配置中 DATA_PATH_HOST 变量，将宿主机上存放 Redis 数据的目录挂载到容器中 /data 目录  
ports:  
- "${REDIS_PORT}:6379" # 设置容器6379端口映射指定宿主机端口  
networks:  
- backend  
restart: always

mysql-manage:  
build:  
context: ./mysql-manage  
environment:  
- TZ=TZ−PMAARBITRARY=1−MYSQLUSER=TZ−PMAA​RBITRARY=1−MYSQLU​SER={MYSQL_MANAGE_USERNAME} # 设置连接的 Mysql 服务用户名称  
- MYSQL_PASSWORD={MYSQL_MANAGE_PASSWORD} # 设置连接的 Mysql 服务用户密码 - MYSQL_ROOT_PASSWORD={MYSQL_MANAGE_ROOT_PASSWORD} # 设置连接的 Mysql 服务 root 用户密码  
- PMA_HOST={MYSQL_MANAGE_CONNECT_HOST} # 设置连接的 Mysql 服务 host，可以是 Mysql 服务容器的名称，也可以是 Mysql 服务容器的 ip 地址 - PMA_PORT={MYSQL_MANAGE_CONNECT_PORT} # 设置连接的 Mysql 服务端口号  
ports:  
- "${MYSQL_MANAGE_PORT}:80" # 设置容器80端口映射指定宿主机端口，用于宿主机访问可视化web  
depends_on: # 依赖容器  
- mysql # 在 Mysql 服务容器启动后启动  
networks:  
- backend  
restart: always

redis-manage:  
build:  
context: ./redis-manage  
environment:  
- TZ=TZ−ADMINUSER=TZ−ADMINU​SER={REDIS_MANAGE_USERNAME} # 设置 Redis 可视化管理的用户名称  
- ADMIN_PASS={REDIS_MANAGE_PASSWORD} # 设置 Redis 可视化管理的用户密码 - REDIS_1_HOST={REDIS_MANAGE_CONNECT_HOST} # 设置连接的 Redis 服务 host，可以是 Redis 服务容器的名称，也可以是 Redis 服务容器的 ip 地址  
- REDIS_1_PORT={REDIS_MANAGE_CONNECT_PORT} # 设置连接的 Redis 服务端口号 ports: - "{REDIS_MANAGE_PORT}:80" # 设置容器80端口映射指定宿主机端口，用于宿主机访问可视化web  
depends_on: # 依赖容器  
- redis # 在 Redis 服务容器启动后启动  
networks:  
- backend  
restart: always

etcd-manage:  
build:  
context: ./etcd-manage  
environment:  
- TZ=TZports:−"TZports:−"{ETCD_MANAGE_PORT}:8080" # 设置容器8080端口映射指定宿主机端口，用于宿主机访问可视化web  
depends_on: # 依赖容器  
- etcd # 在 etcd 服务容器启动后启动  
networks:  
- backend  
restart: always

prometheus:  
build:  
context: ./prometheus  
environment:  
- TZ={TZ} privileged: true volumes: - ./prometheus/prometheus.yml:/opt/bitnami/prometheus/conf/prometheus.yml # 将 prometheus 配置文件挂载到容器里 ports: - "{PROMETHEUS_PORT}:9090" # 设置容器9090端口映射指定宿主机端口，用于宿主机访问可视化web  
networks:  
- backend  
restart: always

grafana:  
build:  
context: ./grafana  
environment:  
- TZ=TZports:−"TZports:−"{GRAFANA_PORT}:3000" # 设置容器3000端口映射指定宿主机端口，用于宿主机访问可视化web  
networks:  
- backend  
restart: always

jaeger:  
build:  
context: ./jaeger  
environment:  
- TZ=TZports:−"TZports:−"{JAEGER_PORT}:16686" # 设置容器16686端口映射指定宿主机端口，用于宿主机访问可视化web  
networks:  
- backend  
restart: always

dtm:  
build:  
context: ./dtm  
environment:  
- TZ={TZ} entrypoint: - "/app/dtm/dtm" - "-c=/app/dtm/configs/config.yaml" privileged: true volumes: - ./dtm/config.yml:/app/dtm/configs/config.yaml # 将 dtm 配置文件挂载到容器里 ports: - "{DTM_HTTP_PORT}:36789"  
- "${DTM_GRPC_PORT}:36790"  
networks:  
- backend  
restart: always  
就是我在使用docker-compose启动服务后，我在docker desktop中并没有看到容器启动，而是需要手动启动？我想不明白，docker run一个容器之后，这个容器和docker desktop是互通的吗？还是我的dokcer使用出了一些问题，





你的 Docker 容器很多，且许多容器都处于 "Exited" 状态，表示它们已经停止运行。你可以使用以下命令清理不再需要的容器和镜像：

1. **查看所有停止的容器：**
    
    ```bash
    docker ps -a
    ```
    
2. **删除停止的容器：** 如果你想删除所有已停止的容器，可以使用：
    
    ```bash
    docker container prune
    ```
    
    它会提示确认删除所有已停止的容器。
    
3. **删除特定容器：** 如果你只想删除某个特定容器，可以使用：
    
    ```bash
    docker rm <container_id>
    ```
    
4. **删除未使用的镜像：** 删除没有依赖的镜像：
    
    ```bash
    docker image prune
    ```
    
5. **删除所有未使用的容器、网络、镜像和构建缓存：**
    
    ```bash
    docker system prune -a
    ```
    

这些命令会帮助你清理不需要的容器和镜像，释放磁盘空间。如果你确认某些容器不再需要，可以使用相应的命令来清理。



#   指令


`docker run -d --name etcd -p 2379:2379 -e ETCD_ROOT_PASSWORD=secret bitnami/etcd
fa7a2bc6d42e7f151a075dbc287490b69854ae5ce9be2979b194cca1ca995925`


你拉取的镜像已经要和文件夹中定义的版本一样才行啊!!!!!!不能自己下拽，否则服务根本无法关联到一起

！！！！


为etcd创建用户密码！

你遇到的错误 `etcdserver: user name is empty, key is user.rpc` 说明在执行操作时，etcd 试图使用一个空的用户名，这通常是由于身份验证或用户配置的问题。

### 解决步骤：

1. **确认 `etcd` 用户是否已正确创建：**
    
    根据你提供的错误信息，etcd 无法识别有效的用户名。首先需要确保 `etcd` 已正确配置用户，并且正在使用有效的用户名。
    
    如果你还没有创建用户，请尝试以下步骤：
    
    - **进入 `etcd` 容器**：
        
        ```bash
        docker exec -it etcd /bin/bash
        ```
        
    - **使用正确的用户名和密码添加用户**：
        
        ```bash
        ETCDCTL_API=3 etcdctl --endpoints=http://localhost:2379 --user=root:secret user add root:secret
        ```
        
    
    这里使用了你启动容器时定义的 `root:secret` 用户名和密码。
    
2. **使用有效的身份验证连接 `etcd`：**
    
    在执行任何命令时，确保你正确地指定了 `--user=root:secret`，否则 `etcdctl` 会认为用户名为空。
    
    - 例如，列出用户时：
        
        ```bash
        ETCDCTL_API=3 etcdctl --endpoints=http://localhost:2379 --user=root:secret user list
        ```
        
3. **检查 `etcd` 服务状态：**
    
    确保 `etcd` 服务正在运行并且没有其他错误。你可以查看容器日志：
    
    ```bash
    docker logs etcd
    ```
    
4. **配置和启用安全连接（如果需要）：**
    
    如果你使用的是 `https` 连接而不是 `http`，你需要确保容器中包含正确的证书文件，或者禁用 TLS 验证（仅在非生产环境中使用）：
    
    ```bash
    ETCDCTL_API=3 etcdctl --endpoints=https://localhost:2379 --user=root:secret --insecure-transport --insecure-skip-tls-verify user list
    ```
    
5. **重启 `etcd` 容器：**
    
    如果问题仍然存在，可以尝试重新启动容器，以确保所有配置生效：
    
    ```bash
    docker restart etcd
    ```
    
6. **检查 `etcd` 配置文件：**
    
    如果上面的步骤没有解决问题，检查 `etcd` 的配置文件是否正确，确保没有遗漏任何用户权限设置或配置。
    

通过确保正确设置用户名、密码和身份验证信息，你应该能够解决当前的 `user name is empty` 错误。


错误：

```shell
etcdctl user list
{"level":"warn","ts":"2025-01-27T14:13:38.751736Z","logger":"etcd-client","caller":"v3@v3.5.18/retry_interceptor.go:63","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc0003f2000/127.0.0.1:2379","attempt":0,"error":"rpc error: code = InvalidArgument desc = etcdserver: user name is empty"}
Error: etcdserver: user name is empty
```



# vscode打断点调试

```json
{

    "version": "0.2.0",

    "configurations": [

        {

            "name": "Launch",

            "type": "go",

            "request": "launch",

            "mode": "auto",

            "program": "E:\\go-zero-server\\gonivinck\\mall\\service\\user\\api"  // 调试程序的路径（绝对路径）

        }

    ]

}
```




# 成功启动rpc

![[rpc启动.png]]