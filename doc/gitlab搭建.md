```
$ docker pull gitlab/gitlab-ce:latest #15.4.6
## 8008和8022是为了保证宿主机和docker内一致，这样内部的url不会显示错误，4567是都docker仓库上传镜像用的端口
$ docker run --detach \
--publish 8843:443 --publish 8008:8008 --publish 8022:8022 --publish 4567:4567 \
--name gitlab \
--restart always \
--volume /home/bazinga/raid5/docker_hub/gitlab/config:/etc/gitlab \
--volume /home/bazinga/raid5/docker_hub/gitlab/logs:/var/log/gitlab \
--volume /home/bazinga/raid5/docker_hub/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```

> <https://docs.gitlab.com/ee/install/docker.html>

### 容器内22端口需要修改

```
## docker内部ssh文件需要更改22端口,操作都在docker里
$ docker exec -it gitlab /bin/bash
$ vi /assets/sshd_config #进入文件，修改port
$ service ssh restart #容器内执行
```

### ps

**常用的docker镜像，我都存在了gitlab的项目里**

基本docker都拉官方的升级即可

### gitlab升级

> 需要按照版本升级，不能直接升级最新docker
>
> <https://docs.gitlab.com/ee/update/index.html#upgrade-paths>
>
> docker 仓库
>
> <https://hub.docker.com/r/gitlab/gitlab-ce/tags?page=1&name=15.4>