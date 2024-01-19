## 搭建教程

```
$ sudo su #都要root权限

$ fdisk -l #or parted -l #查看硬盘
$ parted /dev/sda #硬盘分区，大于2T用GPT分区
  (parted) mklabel gpt
  (parted) mkpart primary 0% 100%
  (parted) print
  quit
$ parted -l

$ apt install mdadm #安装mdadm
$ mdadm -Cv /dev/md0 -l5 -n3 /dev/sda /dev/sdb /dev/sdc #raid5，-l是raid等级，-n是盘数，-Cv创建一个阵列并打印详细信息
$ mkfs.ext4 -F /dev/md0 #格式化

#挂载
$ mkdir ~/raid5
$ mount /dev/md0 ~/raid5
$ df -h

$ cat /proc/mdstat #擦看raid

$ mdadm --detail --scan|tee -a /etc/mdadm/mdadm.conf #raid信息写入

$ update-initramfs -u #更新配置

$ vim /etc/fstab #配置开机自动挂载，nofail表示无设备时跳过
/dev/md0 /home/bazinga/raid5 ext4 defaults,nofail 0,0
```

> <https://blog.csdn.net/Mico18/article/details/127314938>

## 重新生成磁盘阵列（ps来自gpt未测试）

如果你的电脑损坏了，但是RAID 5阵列的磁盘驱动器还是完好无损的，你可以在新的计算机上重新生成这个RAID阵列，并且访问你的数据。请注意，在重新生成RAID阵列之前，确保你使用相同数量和大小的磁盘驱动器。

下面是在新的计算机上重新生成RAID 5阵列的步骤：

1. **安装**`mdadm`工具： 首先，在新的计算机上确保`mdadm`工具已经安装。如果尚未安装，可以使用以下命令进行安装：

```
sudo apt update
sudo apt install mdadm
```

1. **插入磁盘驱动器**： 将之前用于RAID 5阵列的磁盘驱动器插入新的计算机中，确保它们被识别并显示在系统中。
2. **查看磁盘驱动器列表**： 使用以下命令查看系统上已连接的磁盘驱动器列表：

```
sudo fdisk -l
```

通常，你会在输出中看到类似`/dev/sdX`（例如：`/dev/sdb`、`/dev/sdc`、`/dev/sdd`）的设备名称。

1. **激活RAID 5阵列**： 使用`mdadm`命令激活之前创建的RAID 5阵列：

```
sudo mdadm --assemble --scan
```

这个命令会自动扫描所有可用的RAID设备并激活RAID 5阵列。

1. **查看RAID 5阵列状态**： 可以使用以下命令来查看RAID 5阵列的状态：

```
cat /proc/mdstat
```

这将显示有关RAID设备状态和同步进度的信息。

1. **挂载RAID 5阵列**： 如果RAID 5阵列被成功激活，你可以将其挂载到文件系统以访问你的数据：

```
sudo mkdir /mnt/raid
sudo mount /dev/md0 /mnt/raid
```

现在，你应该可以通过`/mnt/raid`目录访问你之前创建的RAID 5阵列中的数据。

请注意，RAID 5阵列的重新生成只适用于当RAID磁盘驱动器仍然完好无损时。如果磁盘驱动器有损坏，你可能需要进行数据恢复操作。为了更好地保护你的数据，请定期备份重要文件。

## 更换磁盘（ps来自gpt未测试）

如果一个磁盘损坏了，你需要及时更换损坏的磁盘以保持RAID 5阵列的冗余性和数据完整性。在更换损坏的磁盘时，你需要遵循以下步骤：

1. **检查RAID 5阵列状态**： 在更换磁盘之前，首先检查RAID 5阵列的状态，确保你知道哪个磁盘损坏了。使用以下命令查看RAID 5阵列的状态：

```
cat /proc/mdstat
```

这将显示有关RAID设备状态和同步进度的信息。你将看到类似`/dev/mdX`（例如：`/dev/md0`）和相应的RAID设备状态。

1. **删除损坏的磁盘**： 在更换损坏的磁盘之前，你需要从RAID 5阵列中删除损坏的磁盘。使用以下命令将损坏的磁盘从RAID阵列中删除：

```
sudo mdadm /dev/mdX --fail /dev/sdXY
sudo mdadm /dev/mdX --remove /dev/sdXY
```

将`/dev/mdX`替换为你的RAID设备名称（例如：`/dev/md0`），将`/dev/sdXY`替换为你要删除的损坏磁盘的设备名称（例如：`/dev/sdc1`）。

1. **插入新的磁盘**： 将新的磁盘驱动器插入到系统中，确保系统可以识别并显示该磁盘。
2. **添加新的磁盘到RAID阵列**： 使用以下命令将新的磁盘添加到RAID 5阵列：

```
sudo mdadm /dev/mdX --add /dev/sdXY
```

将`/dev/mdX`替换为你的RAID设备名称（例如：`/dev/md0`），将`/dev/sdXY`替换为你新插入的磁盘设备名称（例如：`/dev/sdd1`）。

1. **等待重建**： RAID 5阵列将开始使用新的磁盘进行重建，这个过程可能需要一些时间，具体时间取决于磁盘的大小和系统负载。你可以使用以下命令来监视重建进度：

```
watch cat /proc/mdstat
```

重建完成后，你的RAID 5阵列将重新恢复冗余性，并且你的数据将保持完整。

请注意，在重建期间，RAID 5阵列处于易损状态，如果在这个过程中另一个磁盘出现问题，可能会导致数据丢失。因此，在更换损坏的磁盘后，请确保备份重要数据。为了保护数据，请定期备份数据并定期检查RAID 5阵列的状态。