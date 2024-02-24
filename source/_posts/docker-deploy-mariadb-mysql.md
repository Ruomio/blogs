---
title: docker_deploy_mariadb_mysql
date: 2024-02-24 09:43:52
tags:
    - mysql
    - mariadb 
categories:
    - docker
---

# docker 配置MariaDB
1. 拉取mariadb/mysql docker 镜像
```shell
docker pull mariadb
```

2. 启动容器
```shell
docker run -d --name mariadb -p 3306:3306 -e TZ="Asia/Shanghai"  -e MYSQL_ROOT_PASSWORD='your_password' -e MARIADB_PASSWORD='your_password'  mariadb
```

3. 设置远程连接权限
```shell
# a. 进入容器
docker exec -it mariadb /bin/bash 

# b. 授权
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'your_password';

# c. 刷新权限
flush privileges;

```

4. 重启容器
```shell 
docker restart mariadb
```

5. 可以测试连接刚才设置的数据库
```shell 
# 远程连接 x.x.x.x为公网ip地址
mariadb -h x.x.x.x -u root -p 'your_password'

# 本地连接
mariadb -u root -p 'your_password'
```

# 注意
要在云服务器厂商的控制台开启对应的端口规则，这里使用的默认端口3306
