### Reference
http://patches.dpdk.org/patch/41262/ </br>
https://blog.csdn.net/aa304037208/article/details/90400351 </br>

```
https://git.kernel.org/pub/scm/utils/rt-tests/rt-tests.git/snapshot/rt-tests-1.8.tar.gz
tar -zxvf rt-tests-1.8.tar.gz
cd rt-tests
export CROSS_COMPILE=aarch64-linux-gnu-
make
```

### 问题
- 编译出错
```
src/cyclictest/rt_numa.h:18:10: fatal error: numa.h: No such file or directory
 #include <numa.h>
```
解决
```
> +   git clone https://github.com/numactl/numactl.git
> +   cd numactl
> +   git checkout v2.0.11 -b v2.0.11
> +   ./autogen.sh
> +   autoconf -i
> +   ./configure --host=x86_64 CC=aarch64-linux-gnu-gcc prefix=<numa install dir>
> +   make install


> +   cp <numa_install_dir>/include/numa*.h <cross_install_dir>/gcc-linaro-7.2.1-2017.11
> +   -x86_64_aarch64-linux-gnu/bin/../aarch64-linux-gnu/libc/usr/include/
```
