- ssh不断开连接
```
ssh -o ServerAliveInterval=30 IP地址
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
