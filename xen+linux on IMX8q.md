### Reference
https://www.nxp.com/document/guide/get-started-with-the-i.mx-8quadmax-mek:GS-iMX-8QM-MEK </br>
https://www.nxp.com/docs/en/quick-reference-guide/IMX8QUADMAXQSG.pdf
</br></br></br>
### 工具
minicom
下载imx8软件包</br>
https://www.nxp.com/design/i-mx-developer-resources/i-mx-software-and-development-tools:IMX-SW


### step0 开发板
1. 接电源
2. 接USB接口，用于minicom调试用，用于开发板输出输入信息。
3. sd卡，用于插入SD卡插槽，向SD卡写入镜像，将3号位和4号位拨片置1，即从sd卡启动开发板


### step1 准备sd卡
- 用读卡器将SD卡插入电脑，一般SD卡为/dev/sdb。
- 从官网下载软件包，将其中的fsl-image-validation-imx-imx8qmmek.sdcard镜像烧写。
https://www.nxp.com/webapp/Download?colCode=L_VIRT_4.11_0.10_ga_MX8QM&appType=license
```
sudo dd if=fsl-image-validation-imx-imx8qmmek.sdcard.sdcard of=/dev/sdx bs=1M && sync
```
- 官网下载android_p9.0.0_2.1.0-autoga_image_8qmek.tar.gz，解压，将spl-imx8qm-xen.bin拷贝到SD卡FAT分区
```
sudo cp spl-imx8qm-xen.bin /medie/chen/imx\ Boot
```
- 将u-boot-imx8qm-xen-dom0.imx烧写如SD卡
```
sudo dd if=u-boot-imx8qm-xen-dom0.imx of=/dev/sdb seek=32 bs=1k && sync
```

### step2 交叉编译Unixbench
- 修改Makefile，修改编译器
```
CC=aarch-linux-gnu-gcc
```
- 注释Run文件内main函数内的prechecks调用，该函数会重新编译Unixbench，而Dom0运行的linux并不具备编译Unixbench的环境配置
```
//prechecks()
```
- 修改Makefile文件内内架构及CPU配置
```
march=armv8-a mtune=Cortex-a53
```
- 编译
```
make clean
make all
```
- 将Unixbench拷贝进SD卡
```
sudo cp -r Unixbench /path/to/linux/partition/lib
```

### step3 移植perl
官网准备好的linux rootfs，其perl版本较新，但是perl module不齐全，比如部剧本cpan.pm即POSIX.pm，这些都是运行Unixbench所需要的模块。
所以需要自己手动移植编译好的perl模块。
我将之前在qemu上运行的linux rootfs的perl移植到SD卡的rootfs上
```
wget   https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-arm64-uefi1.img -O ubuntu.qcow2
```
- 挂载该rootfs
```
sudo modprobe nbd
sudo qemu-nbd -c /dev/nbd0 /path/to/ubuntu.qcow2
sudo mount /dev/nbd0p1 /mnt
```
- 将perl移植到SD卡的linux rootfs上，并删除其原本的perl


### step4 运行测试
测试会执行两次，一次是单进程，第二次是多进程，一般进程数为CPU数量
```
./Run
```
- 测试数据
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/%E6%B5%8B%E8%AF%951.png)
</br></br></br>
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/%E6%B5%8B%E8%AF%952.png)




#### 问题
.sdcard镜像，完整的.sdcard镜像已经分好区了</br>
默认登录用户：root，无密码</br>

解决perl缺少POSIX.pm模块问题</br>
sudo perl -MCPAN -e 'install POSIX'
