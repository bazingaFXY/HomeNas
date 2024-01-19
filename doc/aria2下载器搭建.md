## 当前使用的方案

宿主机直接安装aira2+webUI

> <https://www.10bests.com/deploy-aria2-webui-on-ubuntu-and-debian/>

## 预计使用方案

docker

aira2+ariaNG 全用docker

```
$ docker pull p3terx/aria2-pro
$ docker pull p3terx/ariang
$ docker run -d \
    --name aria2-pro \
    --restart unless-stopped \
    --log-opt max-size=1m --network host -e PUID=0 -e PGID=0 \
    -e RPC_SECRET=password \
    -e RPC_PORT=6800 \
    -e LISTEN_PORT=6888 \
    -v /home/bazinga/raid5/docker_hub/aria2/config:/config \
    -v /home/bazinga/raid5/movies:/downloads/movies \
    -v /home/bazinga/raid5/downloads:/downloads/outher \
    p3terx/aria2-pro
$ docker run -d \
    --name ariang \
    --log-opt max-size=1m \
    --restart unless-stopped \
    -p 8888:6880 \
    p3terx/ariang
```
