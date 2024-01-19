## nextcloud server和客户端下载链接

> 官网链接，主要客户端下载
>
> <https://nextcloud.com/install/#install-clients>
>
> github，使用docker下载
>
> <https://github.com/nextcloud/all-in-one#how-to-use-this>

### 个人服务器实际使用命令

```
$ docker pull nextcloud 
$ docker run --name=nextcloud --privileged --restart=always --mount type=tmpfs,destination=/tmp -d -p 8000:80 --link mysql:db -v /home/bazinga/raid5/docker_hub/nextcloud:/var/www/html nextcloud:25.0.9
```

```
#老命令
$ docker run --name=nextcloud --restart=always --mount type=tmpfs,destination=/tmp -d -p 8000:80 -v /home/bazinga/nextcloud:/var/www/html --link mysql:db nextcloud  #--mount挂载的是为了识别的插件减轻磁盘负担-v后面是挂载的文件夹，需要本地创建，另外还有一个8080的可以使用ip访问的端口，有需要也可以挂载上去
```

### 系统升级

```
$ docker exec -u 33 nextcloud php occ upgrade 
#-u 33 表示用www-data这个用户执行
```

### 手动拷贝文件到nextcloud

```
#先将文件拷贝到nextcloud/data/user/files/,然后执行下面命令
$ docker exec -u 33 nextcloud php occ files:scan --all
#-u 33 表示用www-data这个用户执行，容器中无法切换这个用户，c82a1e71b5af表示容器id
```

**occ有三个用于管理Nextcloud中文件的命令：**

```
files
 files:cleanup              #清楚文件缓存
 files:scan                 #重新扫描文件系统
 files:transfer-ownership   #将所有文件和文件夹都移动到另一个文件夹
```

**我们需要使用 files:scan   来扫描新文件。**

```
  格式:
  files:scan [-p|--path="..."] [-q|--quiet] [-v|vv|vvv --verbose] [--all]
  [user_id1] ... [user_idN]

参数:
  user_id               #扫描所指定的用户（一个或多个，多个用户ID之间要使用空格分开）的所有文件

选项:
  --path                #限制扫描路径
  --all                 #扫描所有已知用户的所有文件
  --quiet               #不输出统计信息
  --verbose             #在扫描过程中显示正在处理的文件和目录
  --unscanned           #仅扫描以前未扫描过的文件
```

### 后台作业管理

> 官方文档
>
> <https://docs.nextcloud.com/server/25/admin_manual/configuration_server/background_jobs_configuration.html>
>
> 教程文档
>
> [https://hellogitlab.com/CI/docker/nextcloud_in_docker.html#\_13-手动添加文件到nextcloud用户目录-不显示处理](https://hellogitlab.com/CI/docker/nextcloud_in_docker.html#_13-%E6%89%8B%E5%8A%A8%E6%B7%BB%E5%8A%A0%E6%96%87%E4%BB%B6%E5%88%B0nextcloud%E7%94%A8%E6%88%B7%E7%9B%AE%E5%BD%95-%E4%B8%8D%E6%98%BE%E7%A4%BA%E5%A4%84%E7%90%86)

总结就是在宿主机使用crontab 每5分钟执行一次docker命令，命令主要是

```
$ sudo crontab -u bazinga -e

*/5  *  *  *  * docker exec -u 33 nextcloud php -f /var/www/html/cron.php #文本中添加这行

$ sudo crontab -u bazinga -l #可以用这行来查看作业
```

### 在线编辑office

> 部署教程
>
> <https://www.jianshu.com/p/f66f68de845c>

```
$ docker pull onlyoffice/documentserver
$ docker run -d --name=onlyoffice -p 8009:80 -v /home/bazinga/raid5/docker_hub/onlyoffice/logs:/var/log/onlyoffice -v /home/bazinga/raid5/docker_hub/onlyoffice/data:/var/www/onlyoffice/Data -v /home/bazinga/raid5/docker_hub/onlyoffice/lib:/var/lib/onlyoffice -v /home/bazinga/raid5/docker_hub/onlyoffice/db:/var/lib/postgresql --restart=always -e JWT_SECRET=secret onlyoffice/documentserver
```

**nextcloud上需要安装onlyoffice插件，并配置服务器等设置，密钥可以通过onlyoffice页面获取**

### MYSQL

当前服务器的数据库使用的是默认数据库，官方推荐使用mysql的数据库，准备后期搭建一个mysql的数据库docker，官方有迁移教程。教程如下：

> <https://www.cnblogs.com/steinven/p/11357295.html>

```
$ docker pull mysql
$ docker run --name=mysql -e MYSQL_ROOT_PASSWORD=password -v /home/bazinga/raid5/docker_hub/mysql/conf:/etc/mysql/conf.d -v /home/bazinga/raid5/docker_hub/mysql/data:/var/lib/mysql -d -p 33306:3306 --restart=always mysql:latest #新系统版本8.0.32

$ docker run -d \
--name=nextcloud \
--privileged \
--link mysql:db \ 
-v /home/bazinga/nextcloud:/var/www/html \
-p 8000:80 \
--restart=always \
nextcloud

## 需要先进入mysql的docker创建关联数据库
$ docker exec -it mysql /bin/bash
$ mysql -u root -p  #docker起来后需要等待很久，否则进不去
$ CREATE DATABASE nextcloud_db; #创建数据库最后一个是数据库name
$ drop database nextcloud_db; #出错删除数据库命令

## nextcloud迁移实际命令
$ php occ db:convert-type --port="33306" --password="password" mysql root 192.168.2.20 nextcloud_db
```

然后参考官方文档迁移数据库

> <https://docs.nextcloud.com/server/25/admin_manual/configuration_database/db_conversion.html>

#### MYSQL四字符

> 官方文档
>
> <https://docs.nextcloud.com/server/25/admin_manual/configuration_database/mysql_4byte_support.html>

#### 垃圾箱管理设置

> <https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/config_sample_php_parameters.html#deleted-items-trash-bin>

```
$ sudo vim nextcloud/config/config.php
## 添加下面的配置，表示删除超过30天的文件，如果空间不足，直接删除文件
'trashbin_retention_obligation' => 'auto, 30',
```