### Reference
https://wiki.xenproject.org/wiki/Xen_ARM_with_Virtualization_Extensions/qemu-system-aarch64 </br>
https://blog.csdn.net/gong0791/article/details/86365718 </br>
https://gcc.gnu.org/onlinedocs/gcc-8.1.0/gcc/AArch64-Options.html </br>
https://www.cnblogs.com/idyllcheung/p/11299161.html

### Environtment
- 宿主机：windows 10 x86_64
- 虚拟机：VMWare Workstation 15.5
- 客户机：Ubuntu 16.04 x86_64 memmory-6G disk-70G
- 模拟环境：qemu 4.1

### step0 pre-request
准备交叉编译工具：aarch64-linux-gnu-gcc
系统跑分测试：unixbench
https://github.com/kdlucas/byte-unixbench </br>

### step1 交叉编译linux xen和qemu
- for linux
```
$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
```
修改.config CONFIG_SQUASHFS_XZ=y。后续启动rootfs需要这个功能。编译Linux，并拷贝到arm64目录
```
$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j8
$ cp arch/arm64/boot/Image.gz ~/workspace/arm64
```
- for xen
```
$ sudo git clone https://xenbits.xen.org/git-http/xen.git -b stable-4.12
$ make dist-xen XEN_TARGET_ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j8
$ cp xen/xen.efi ~/workspace/arm64
```
- for qemu
```
$ git clone https://gitee.com/mirrors/qemu.git -b stable-4.1
$ ./configure --target-list=aarch64-softmmu --enable-debug --prefix=/usr/local
$ make -j8
$ sudo make install
```

### step2 准备rootfs启动linux
- 下载rootfs
```
mkdir -pv ~/workspace/arm64 && cd /home/john/work/arm64
wget   https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-arm64-uefi1.img -O ubuntu.qcow2
```

### first boot
第一次boot主要设置root用户密码，并删除一些不需要的package。
一下命令最好放在exe.sh可执行文件内，方便修改，内存建议2G以下，内存过大运行benchmark时会耗尽客户机内存资源，导致qemu进程被强制杀死。
```
cd ~/workspace/arm64
qemu-system-aarch64 \
   -M virt,gic_version=3,virtualization=true,type=virt \
   -cpu cortex-a57 -nographic -smp 4 -m 2048 \
   -kernel Image.gz --append "console=ttyAMA0 root=/dev/vda1 init=/bin/sh" \
   -drive if=none,file=ubuntu.qcow2,format=qcow2,id=hd0 -device virtio-blk-device,drive=hd0 \
   -netdev user,id=hostnet0,hostfwd=tcp::2222-:22 -device virtio-net-device,netdev=hostnet0
```
在弹出的shell中，手动mount rootfs，然后使用passwd root命令设置root的密码，并删除部分不需要的包，否则下次启动linux会比较卡
```
mount -o remout,rw /dev/vda1 /
sudo apt-get purge cloud-init cloud-guest-utils snapd
passwd root
```
命令退出会卡住，用kill -9 pid指令关闭qemu进程

### 为xen启动准备需要的配置文件
- xen启动时可访问的目录为boot/efi
- 将ubuntu rootfs挂载到/mnt目录下，nbd0p1为ubuntu根目录，nbd0p15模块为xen可访问
```
$ sudo modprobe nbd
$ sudo qemu-nbd -c /dev/nbd0 /path/to/ubuntu.qcow2
$ sudo mount /dev/nbd0p15 /mnt
$ cd /mnt/EFI/XEN
$ sudo cp /path/to/xen.git/xen/xen/xen.efi ./
```
- 设置dom0的配置文件，vim /boot/efi/xen.cfg，kernel为编译好的image.gz
```
options=console=dtuart noreboot dom0_mem=1024M
kernel=kernel root=/dev/vda1 init=/bin/sh rw console=hvc0
dtb=virt-gicv3.dtb
```
- dtb=virt-gicv3.dtb通过以下命令自动生成，将virt-gicv3.dtc拷贝到/mnt/EFI/XEN目录下。
```
cd ~/workspace/arm64
qemu-system-aarch64 -M virt,gic_version=3,virtualization=true,type=virt -cpu cortex-a57 -smp 4 -m 2048 -display none -machine dumpdtb=virt-gicv3.dtb
```
### 交叉编译unixbench，并将其可执行文件放大rootfs目录下
- 修改Makefile交叉编译
```
#CC=gcc
CC = aarch64-linux-gnu-gcc
```
- 编译
```
make clean
make all
```
- 修改Run
将main函数中的 preChecks();注释掉，因为其中有 system("make all");这个函数会重新编译unixbench，而Dom0运行的ubuntu并没有配置gcc编译环境，所以会导致失败。
- 挂载/dev/nbd0p1 ubuntu根目录
```
$ sudo umoungt /dev/nbd0p15
$ sudo mount /dev/nbd0p1 /mnt
$ cd /mnt/EFI/XEN
$ sudo cp -r /path/to/UnixBench lib
```
- 取消根目录挂载
```
$ sudo umount /mnt
$ sudo qemu-nbd -d /dev/nbd0
```
### step3使用QEMU_EFI.fd启动arm64虚拟机
- 获取QEMU_EFI.fd
```
cd /home/john/work/arm64
wget  http://snapshots.linaro.org/components/kernel/leg-virt-tianocore-edk2-upstream/latest/QEMU-AARCH64/RELEASE_CLANG35/QEMU_EFI.fd 
```

- 启动qemu
```
qemu-system-aarch64 \
   -M virt,gic_version=3,virtualization=true,type=virt \
   -cpu cortex-a57 -nographic -smp 4 -m 2048 \
   -bios QEMU_EFI.fd \
   -drive if=none,file=ubuntu.qcow2,format=qcow2,id=hd0 -device virtio-blk-device,drive=hd0 \
   -netdev user,id=hostnet0,hostfwd=tcp::2222-:22 -device virtio-net-device,netdev=hostnet0
```
启动后连续敲击Esc键，进入uefi界面，进入Boot Manager，再进入EFI Internal Shell。 如果不敲击esc键的话，就会进入默认的ubuntu中。
进入fs0，执行xen.efi文件。执行xen.efi前可ls检查一下该目录下所需配置文件是否齐全，需要kernel镜像，xen.cfg配置文件，和virt-gicv3.dtb三个文件
```
Shell> fs0:
FS0:\> ls
FS0:\> xen.efi
```
几分钟启动后默认进入Dom0的ubuntu系统。

### 执行unixbench测试
大概需要半个小时
```
cd lib
cd UnixBnech
./Run
```
- 结果

![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/unixbench.png)
</br></br></br></br></br>

注：以下为一些参考知识
### 挂载镜像传输文件
- nbd(Network Block Device)挂载，让用户可以通过玩过访问某个块设备或者设备镜像，比如将xen镜像在nbd-server某端口运行起来，将nbd设备关联到/dev/nbdx设备上，然后将/dev/nbdpx挂载到mnt目录下，则可以通过mnt目录查看镜像内的内容。

```
$ sudo modprobe nbd
$ sudo qemu-nbd -c /dev/nbd0 /path/to/xenial-server-cloudimg-arm64-uefi1.img
$ sudo mount /dev/nbd0p15 /mnt
$ cd /mnt/EFI/XEN
```
- 取消挂载
```
$ sudo umount /mnt
$ sudo qemu-nbd -d /dev/nbd0
```


 ### 构建linux镜像
 - 获取linux源码
 ```
cd arm_vir/src 
git clone https://gitee.com/mirrors/linux.git
cd linux 
git checkout -b v5.5_dev v5.5
```
- 编译
```
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- O=arm_vir/build/linux V=1 
```

