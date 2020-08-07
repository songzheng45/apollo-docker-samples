# Apollo Docker Samples

> 参考Apollo官方[分布式部署指南][1], 然后基于 docker-compose的部署指南.
>
> Portal 是独立部署, 官方是推荐在生产环境部署,然后统一管理DEV, FAT, UAT, PRO 4个环境的配置, 每个环境独立部署一套 ConfigService, 保存各自的配置内容.

## Portal UI

进入 portal 目录, 执行 `sh ./build.sh` 构建并启动 portal web服务和DB的容器.

登录网址:  http://{Server_IP}:8170/ 

默认登录账密:  apollo / admin

登录后, 进入**管理员工具-系统参数**可以更新参数:

| Key                        | Value                                                        | 备注                    |
| -------------------------- | ------------------------------------------------------------ | ----------------------- |
| apollo.portal.envs         | DEV,FAT,UAT,PRO                                              | 修改完需要重启生效。    |
| apollo.portal.meta.servers | {"DEV":"http://10.8.70.224:8180","FAT":"http://10.8.70.224:8180","UAT":"http://10.8.70.224:8180","PRO":"http://10.8.70.224:8180"} | 修改完需要重启生效。    |
| organizations              | [{"orgId":"IT","orgName":"IT"}]                              | 修改完需要重新登录成效. |
| 参考: [调整服务端配置][2]  |                                                              |                         |



## ConfigService



```sql
USE ApolloConfigDB;

UPDATE ServerConfig SET `Value` = 'http://192.168.8.194:8180/eureka/' WHERE `Key` = 'eureka.service.url';

```





[1]:https://github.com/ctripcorp/apollo/wiki/%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97
[2]:https://github.com/ctripcorp/apollo/wiki/%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97#213-%E8%B0%83%E6%95%B4%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%85%8D%E7%BD%AE