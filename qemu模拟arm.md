### 编译qemu
- 获取qemu源码
```
mkdir -pv arm_vir/src
git clone https://gitee.com/mirrors/qemu.git -b stable-4.1
```
- 配置qemu编译选项
```
./configure --target-list=aarch64-softmmu --prefix=/usr/local
make
sudo make install
```

### 编译u-boot
```
cd arm_vir/src
git clone https://gitlab.denx.de/u-boot/u-boot.git
cd u-boot
git checkout -b v2019.10_origin v2019.10
```
- 配置
```
 mkdir -pv ${XEN_ROOT}/build/u-boot
 make CROSS_COMPILE=aarch64-nonelinux-gnu- O=arm_vir/build/u-boot V=1 qemu_arm64_defconfig
 vi arm_vir/build/u-boot/.config
 CONFIG_ARCH_QEMU=y
 CONFIG_TARGET_QEMU_ARM_64BIT=y
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

### 构建busybox镜像
```
cd arm_vir/src
git clone https://github.com/mirror/busybox.git
cd busybox
git checkout -b 1_31_stable remotes/origin/1_31_stable
```

### 构建xen镜像
- 获取xen源码
```
 sudo git clone https://xenbits.xen.org/git-http/xen.git -b stable-4.121
```
