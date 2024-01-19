```
$ docker pull superng6/qbittorrentee
$ docker run -d \
  --name=qbittorrent \
  -e WEBUIPORT=8880 \ 
  -p 8880:8880 \
  -p 6881:6881 \
  -p 6881:6881/udp \
  -v /home/bazinga/raid5/docker_hub/qbittorrent/config:/config \
  -v /home/bazinga/raid5/movies:/downloads/movies \
  -v /home/bazinga/raid5/downloads:/downloads/outher \
  --restart unless-stopped \
  superng6/qbittorrentee
```

默认帐号密码 admin/adminadmin

改做种子时间，改自动trackers，高级里面上传策略改反吸血

> <https://github.com/ngosang/trackerslist>
>
> <https://raw.githubusercontent.com/ngosang/trackerslist/master/trackers_all.txt>