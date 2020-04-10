
### 问题
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

