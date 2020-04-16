### jailhouse
- jailhouse作为一个firmware image加载，resides in dedicated memory region which you should reserve at Linux boot time
- jailhouse.ko 加载这个firmware image，然后创建/dev/jailhouse 设备
- 挂载linux rootfs，然后将jailhouse make install到该目录下

```
make ARCH=arm64 CROSS_COMPILE=aarch64-poky-linux- KDIR=/home/public2/jailhouse/my_linux/linux-imx CC="aarch64-poky-linux-gcc --sysroot=$SDKTARGETSYSROOT" clean
```
### 问题
bitbake编译中下载软件包遇到的问题: export BB_NO_NETWORK=1
```
bitbake fsl-image-validation-imx --runall fetch
```
```
ERROR:  OE-core's config sanity checker detected a potential misconfiguration.
    Either fix the cause of this error or at your own risk disable the checker (see sanity.conf).
    Following is the list of potential problems / advisories:

    Fetcher failure for URL: 'https://www.example.com/'. URL https://www.example.com/ doesn't work.
    Please ensure your host's network is configured correctly,
    or set BB_NO_NETWORK = "1" to disable network access if
    all required sources are on local disk.
```

编译yocto遇到的问题：升级git版本即可解决
```
bitbake fsl-image-validation-imx
```
```
ERROR: gstreamer1.0-plugins-good-1.14.4.imx-r0 do_unpack: Fetcher failure: Fetch command export PSEUDO_DISABLED=1; export https_proxy="https://202.120.32.250:5678"; export http_proxy="http://202.120.32.250:5678"; export PATH="/home/public2/imx-yocto-bsp/imx8qmmek_wayland/tmp/work/aarch64-mx8-poky-linux/gstreamer1.0-plugins-good/1.14.4.imx-r0/recipe-sysroot-native/usr/bin/python3-native:/home/public2/imx-yocto-bsp/sources/poky/scripts:/home/public2/imx-yocto-bsp/imx8qmmek_wayland/tmp/work/aarch64-mx8-poky-linux/gstreamer1.0-plugins-good/1.14.4.imx-r0/recipe-sysroot-native/usr/bin/aarch64-poky-linux:/home/public2/imx-yocto-bsp/imx8qmmek_wayland/tmp/work/aarch64-mx8-poky-linux/gstreamer1.0-plugins-good/1.14.4.imx-r0/recipe-sysroot/usr/bin/crossscripts:/home/public2/imx-yocto-bsp/imx8qmmek_wayland/tmp/work/aarch64-mx8-poky-linux/gstreamer1.0-plugins-good/1.14.4.imx-r0/recipe-sysroot-native/usr/sbin:/home/public2/imx-yocto-bsp/imx8qmmek_wayland/tmp/work/aarch64-mx8-poky-linux/gstreamer1.0-plugins-good/1.14.4.imx-r0/recipe-sysroot-native/usr/bin:/home/public2/imx-yocto-bsp/imx8qmmek_wayland/tmp/work/aarch64-mx8-poky-linux/gstreamer1.0-plugins-good/1.14.4.imx-r0/recipe-sysroot-native/sbin:/home/public2/imx-yocto-bsp/imx8qmmek_wayland/tmp/work/aarch64-mx8-poky-linux/gstreamer1.0-plugins-good/1.14.4.imx-r0/recipe-sysroot-native/bin:/home/public2/imx-yocto-bsp/sources/poky/bitbake/bin:/home/public2/imx-yocto-bsp/imx8qmmek_wayland/tmp/hosttools"; export HOME="/home/ahv"; git -c core.fsyncobjectfiles=0 submodule update --init --recursive failed with exit code 1, output:
Submodule 'common' (https://anongit.freedesktop.org/git/gstreamer/common.git) registered for path 'common'
Cloning into 'common'...

fatal: unable to access 'https://anongit.freedesktop.org/git/gstreamer/common.git/': Received HTTP code 407 from proxy after CONNECT
Clone of 'https://anongit.freedesktop.org/git/gstreamer/common.git' into submodule path 'common' failed
```

