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
- 修改Makefile


.sdcard镜像，完整的.sdcard镜像已经分好区了

默认登录用户：root，无密码

解决perl缺少POSIX.pm模块问题
sudo perl -MCPAN -e 'install POSIX'
