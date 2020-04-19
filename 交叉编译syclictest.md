```
https://git.kernel.org/pub/scm/utils/rt-tests/rt-tests.git/snapshot/rt-tests-1.8.tar.gz
tar -zxvf rt-tests-1.8.tar.gz
cd rt-tests
export CROSS_COMPILE=aarch64-linux-gnu-
```

### 问题
- 编译出错
```
src/cyclictest/rt_numa.h:18:10: fatal error: numa.h: No such file or directory
 #include <numa.h>
```
