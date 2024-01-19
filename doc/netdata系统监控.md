```
$ docker run -d --name=netdata \
  --pid=host \
  --network=host \
  -v /home/bazinga/raid5/docker_hub/netdata/config:/etc/netdata \
  -v /home/bazinga/raid5/docker_hub/netdata/lib:/var/lib/netdata \
  -v /home/bazinga/raid5/docker_hub/netdata/cache:/var/cache/netdata \
  -v /etc/passwd:/host/etc/passwd:ro \
  -v /etc/group:/host/etc/group:ro \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /etc/os-release:/host/etc/os-release:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --restart unless-stopped \
  --cap-add SYS_PTRACE \
  --cap-add SYS_ADMIN \
  --security-opt apparmor=unconfined \
  netdata/netdata
#要记得在路由器做端口转发
```

## 邮件预警功能

> <https://learn.netdata.cloud/docs/alerting/notifications/agent-dispatched-notifications/email>
>
> <https://www.emperinter.info/2020/12/11/config-email-notifications-of-netdata/>

```
$ vi /root/.msmtprc

defaults
port 465
tls on
tls_starttls off
logfile /var/log/msmtp.log
account 163work
host smtp.163.com
from admin@admin.com
auth login
user admin@admin.com
password password
account default : 163work
```

```
$ docker exec -it netdata /bin/bash
$ ./edit-config health_alarm_notify.conf
 #修改参数
sendmail="/usr/bin/msmtp"
SEND_EMAIL="YES"
DEFAULT_RECIPIENT_EMAIL="admin@admin.com"

$ chmod 600 ~/.msmtprc
$ cp ~/.msmtprc /etc/msmtprc
$ /usr/libexec/netdata/plugins.d/alarm-notify.sh test #测试
```