- kpartx命令挂载镜像: kpartx -a指定去添加哪个映像文件(add)，-v是指挂到loop设备(verbose)，-d就是delete的意思了
```
kpartx -av imx-rootfs.sdcard
kpartx -dv imx-rootfs.sdcard
```

- ssh不断开连接
```
ssh -o ServerAliveInterval=30 IP地址
```

- scp从远程拷贝文件
```
scp -P port user@server:/path/to/name/ /path/to/local/dir/
```
- 查看文件大小
```
du -h filename
du -h --max-depth=1 filename
```
- 查看磁盘空间
```
df -lh
fdisk -l
```

- 杀死多个进程
```
ps -ef | grep qemu | grep -v grep | cut –c 9-15 | xargs kill –s 9
ps aux | grep 进程名| grep -v grep | awk '{print $2}' | xargs kill -9
```
