- sudo -i sudo -s

- find查找文件
```
find / -name httpd.conf　　#在根目录下查找文件httpd.conf，表示在整个硬盘查找
```
- grep查找文件内容
```
grep -nr "name" /dir
```
- 添加用户
```
useradd username
passwd username
```
- rsync删除大量文件
```
rsync --delete-before -a -H -v /home/temp/ /home/file/

–delete-before 接收者在传输之前进行删除操作
–progress 在传输时显示传输过程
-a 归档模式，表示以递归方式传输文件，并保持所有文件属性
-H 保持硬连接的文件
-v 详细输出模式
–stats 给出某些文件的传输状态
```

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
