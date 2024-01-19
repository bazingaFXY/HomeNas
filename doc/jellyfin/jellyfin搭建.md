> > jellyfin官方文档
> >
> > <https://jellyfin.org/docs/general/installation/container>

### 服务器实际命令

```
$ docker pull linuxserver/jellyfin
$ docker run -d \
 --name jellyfin \
 --net=host \
 --volume /home/bazinga/raid5/docker_hub/jellyfin/config:/config \
 --volume /home/bazinga/raid5/docker_hub/jellyfin/cache:/cache \
 --mount type=bind,source=/home/bazinga/raid5/movies,target=/movies \
 --restart=always --device=/dev/dri:/dev/dri \ #硬解驱动
linuxserver/jellyfin
```

# ps

1. ubuntu22的默认内核是5.15，n5105做硬解需要开启低功耗模式，但是这个版本的内核会影响，需要升级

```
$ sudo apt install --install-recommends linux-generic-hwe-22.04 #当前是升级到5.19
```

ps.尝试升级到了5.17的内核，但是对系统负载影响很大，主要体现在web访问很卡顿上，所以教程放在最后，不做推荐

1. 5\.18-6.1.3的版本在使用**opencl based HDR/DV 会对 i915有影响**，但是目前应该不准备使用这个功能，所以内核还是使用5.19

> <https://jellyfin.org/docs/general/administration/hardware-acceleration/intel>

 1. 下面是硬解教程，都在**docker中执行**
 2. 假设您已将 jellyfin 存储库添加到 apt 源列表中并安装了`jellyfin-server`和`jellyfin-web`.
 3. 安装`jellyfin-ffmpeg5`软件包。`jellyfin`如果已弃用的元包破坏了依赖关系，请删除它：

    ```
    sudo apt update && sudo apt install -y jellyfin-ffmpeg5
    ```
 4. 确保`renderD*`中至少存在一台设备`/dev/dri`。否则，请升级内核或在 BIOS 中启用 iGPU。

    ps:请注意可写入的权限和组，在本例中为`render`：

    ```
    $ ls -l /dev/dri
    
    total 0
    drwxr-xr-x  2 root root        120 Mar  5 05:15 by-pathcrw-rw----+ 1 root video  226,   0 Mar  5 05:15 card0crw-rw----+ 1 root video  226,   1 Mar  5 05:15 card1crw-rw----+ 1 root render 226, 128 Mar  5 05:15 renderD128crw-rw----+ 1 root render 226, 129 Mar  5 05:15 renderD129
    ```
 5. 将用户添加`jellyfin`到`render`组中，然后重新启动`jellyfin`服务：

    ps:在某些版本中，该组可能是`video`或`input`代替`render`.

    ```
    sudo usermod -aG render jellyfinsudo systemctl restart jellyfin
    ```
 6. `intel-opencl-icd`检查Linux 发行版提供的版本：

    ```
    $ apt policy intel-opencl-icd
    
    intel-opencl-icd:  Installed: (none)  Candidate: 22.14.22890-1...
    ```
 7. 如果版本比`22.xx.xxxxx`直接安装它新。[否则从Intel 计算运行时存储库](https://github.com/intel/compute-runtime/releases)安装。

    ```
    sudo apt install -y intel-opencl-icd
    ```
 8. 检查支持的 QSV / VA-API 编解码器：

    笔记
    * `iHD driver`表示支持 QSV 和 VA-API 接口。
    * `i965 driver`表示仅支持 VA-API 接口，该接口只能在 Broadwell 之前的平台上使用。

    ```
    $ sudo /usr/lib/jellyfin-ffmpeg/vainfo --display drm --device /dev/dri/renderD128
    
    libva info: VA-API version 1.17.0libva info: Trying to open /usr/lib/jellyfin-ffmpeg/lib/dri/iHD_drv_video.solibva info: Found init function __vaDriverInit_1_17libva info: va_openDriver() returns 0Trying display: drmvainfo: VA-API version: 1.17 (libva 2.17.0)vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 23.1.2 (xxxxxxx)vainfo: Supported profile and entrypoints...
    ```
 9. 检查 OpenCL 运行时状态：

    ```
    $ sudo /usr/lib/jellyfin-ffmpeg/ffmpeg -v verbose -init_hw_device vaapi=va:/dev/dri/renderD128 -init_hw_device opencl@va
    
    [AVHWDeviceContext @ 0x55cc8ac21a80] 0.0: Intel(R) OpenCL HD Graphics / Intel(R) Iris(R) Xe Graphics [0x9a49][AVHWDeviceContext @ 0x55cc8ac21a80] Intel QSV to OpenCL mapping function found (clCreateFromVA_APIMediaSurfaceINTEL).[AVHWDeviceContext @ 0x55cc8ac21a80] Intel QSV in OpenCL acquire function found (clEnqueueAcquireVA_APIMediaSurfacesINTEL).[AVHWDeviceContext @ 0x55cc8ac21a80] Intel QSV in OpenCL release function found (clEnqueueReleaseVA_APIMediaSurfacesINTEL)....
    ```
10. 如果您想使用第二个 GPU，请在 Jellyfin 仪表板中更改`renderD128`为。`renderD129`

在 Jellyfin 中启用 QSV 或 VA-API 并取消选中不支持的编解码器。

```
# docker中安装中文字体，解决封面方块问题
$ apt update
$ apt install fonts-noto-cjk-extra
```