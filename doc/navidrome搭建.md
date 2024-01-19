## navidrome搭建

> github：
>
> <https://github.com/navidrome/navidrome>
>
> 文档：
>
> <https://www.navidrome.org/docs/usage/configuration-options/#configuration-file>

```
$ vim /data/navidrome.toml #创建配置文件

LogLevel = 'DEBUG'
ScanSchedule = '0'
TranscodingCacheSize = '150MiB'
MusicFolder = '/music'
DEFAULTLANGUAGE = 'zh-Hans'
ENABLETRANSCODINGCONFIG = true
```

```
$ docker run -d \
   --name navidrome \
   --restart=unless-stopped \
   -v /home/bazinga/raid5/movies/Music:/music \
   -v /home/bazinga/raid5/docker_hub/navidrome/data:/data \
   -p 4533:4533 \
   -e ND_CONFIGFILE=/data/navidrome.toml \
   deluan/navidrome:latest
```

## tag搭建

> github：
>
> <https://github.com/xhongc/music-tag-web>

```
docker run -d -p 8001:8001 -v /home/bazinga/raid5/movies/Music:/app/media -v /home/bazinga/raid5/docker_hub/music_tag:/app/data --name music_tag xhongc/music_tag_web:latest
```