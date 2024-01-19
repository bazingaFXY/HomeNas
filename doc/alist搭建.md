> <https://github.com/alist-org/alist>

### 实际使用命令

```
$ docker pull xhofe/alist:latest
$ docker run -d --restart=always -v /home/bazinga/docker_hub/alist:/opt/alist/data -p 5244:5244  --name="alist" xhofe/alist:latest
```