# Apollo Docker Samples

> 参考Apollo官方[分布式部署指南][1], 然后基于 docker-compose的部署指南.
>
> Portal 是独立部署, 官方是推荐在生产环境部署,然后统一管理DEV, FAT, UAT, PRO 4个环境的配置, 每个环境独立部署一套 ConfigService, 保存各自的配置内容.
>
> 顺序是：1. 部署 configService， 2. 部署 portal



## ConfigService

### 准备

- `.env`

`.env`定义 MySQL root密码和当前服务器 IP。

必须将 `THIS_SERVER_IP`指定为当前机器的IP，否则会导致portal 无法连接。这里采用的是[分布式部署指南#14-网络策略][3]最后一种方式，即通过系统环境变量直接指定要注册到 Meta Server 的IP+端口。

> 由于 configService 和 adminService 都是跑在容器里，自动注册到 eureka 的ip是容器内ip，比如` 172.19.0.3`，这就导致 portal 无法访问，因此需要修改 `THIS_SERVER_IP`，当然该IP不要暴露在公网中。



### 启动

命令行下进入`config`目录，执行 `build.bat`(windows) 或`sh ./build.sh`(linux)，启动后要稍等1分钟左右，等待 configService 启动。

然后连接 MySQL （端口号 13306），执行以下 SQL 脚本：

```sql
USE ApolloConfigDB;

UPDATE ServerConfig SET `Value` = 'http://{THIS_SERVER_IP}:8180/eureka/' WHERE `Key` = 'eureka.service.url';

```

> 需要修改 `THIS_SERVER_IP`，作用是为了让 adminService 知道实际的 eureka 服务注册地址。

然后重启 adminService 容器。

最后访问`http://{THIS_SERVER_IP}:8180`来查看服务注册情况。

或者  `http://{THIS_SERVER_IP}:8180/services/config`和`http://{THIS_SERVER_IP}:8180/services/admin` 分别检查 configService 和 adminService 状态。



## Portal

进入 portal 目录, 执行 `sh ./build.sh` 构建并启动 portal web服务和DB的容器.

登录网址:  `http://{Server_IP}:8170/` 

默认登录账密:  `apollo / admin`

登录后, 进入**管理员工具-系统参数**更新参数:

| Key                        | Value                                    | 备注                    |
| -------------------------- | ---------------------------------------- | ----------------------- |
| apollo.portal.envs         | DEV,FAT,UAT,PRO                          | 修改完需要重启生效。    |
| apollo.portal.meta.servers | {"DEV":"http://{ConfigService_IP}:8180"} | 修改完需要重启生效。    |
| organizations              | [{"orgId":"IT","orgName":"IT"}]          | 修改完需要重新登录成效. |

参考: [调整服务端配置][2]

> `apollo.portal.meta.servers` 配置各个环境的 Meta Server 地址，也即上面启动的 ConfigService 地址。

修改参数并生效后，观察输出日志，确保和对应环境的 configService 成功建立连接，否则新建项目会出错，因为项目的配置必须保存在ConfigService 的DB中。





[1]:https://github.com/ctripcorp/apollo/wiki/%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97
[2]:https://github.com/ctripcorp/apollo/wiki/%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97#213-%E8%B0%83%E6%95%B4%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%85%8D%E7%BD%AE
[3]:https://github.com/ctripcorp/apollo/wiki/%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97#14-%E7%BD%91%E7%BB%9C%E7%AD%96%E7%95%A5(https://github.com/ctripcorp/apollo/wiki/分布式部署指南#14-网络策略)