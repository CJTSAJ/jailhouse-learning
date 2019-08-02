## intro
  Linux Torvalds在2011.3.17的ARM linux邮件中吐槽了ARM的硬件设备繁琐"the whole ARM thing is a fucking pain in the ass"。
随后ARM linux社区开始进行一系列的修正，以前的ARM linux代码大量代码在描述板级细节，这些细节对内核来说是垃圾。ARM社区开始引入其它体系架构的Device Tree，这是一种描述硬件的数据结构。dts文件编译为dtb文件后被bootloader传递给内核。

- **interrupt-parent**
标识此设备节点属于哪一个中断控制器，如果没有，会自动依附于父节点。
- **compatible**
用来匹配驱动，格式为"[manufacturer], [model]"
- **address-cells**
表示address长度，即用于表示address的数字的数量
- **size-cells**
size的长度，同上

#### reg(region)
reg = <address1 length1 [address2 length2] [address3 length3]>;
```
 1 external-bus {-----------------------------------------------------/* 外部总线片选 */
 2     #address-cells = <2>
 3     #size-cells = <1>;
 4 
 5     ethernet@0,0 {-------------------------------------------------/* 一个片选号，一个片选号上偏移量*/
 6         compatible = "smc,smc91c111";
 7         reg = <0 0 0x1000>;----------------------------------------/* 子设备：片选号/偏移量/地址长度 */
 8     };
 9 
10     i2c@1,0 {
11         compatible = "acme,a1234-i2c-bus";
12         #address-cells = <1>;
13         #size-cells = <0>;
14         reg = <1 0 0x1000>;
15         rtc@58 {
16             compatible = "maxim,ds1338";
17             reg = <58>;
18         };
19     };
20 
21     flash@2,0 {
22         compatible = "samsung,k8f1315ebm", "cfi-flash";
23         reg = <2 0 0x4000000>;
24     };
25 };
```
####  range
属性用于地址映射用，比如把PCI地址映射到内存上。
- <子节点address 父节点address 子节点size>，address长度由对应的cells决定。
```
 1 #address-cells = <1>;
 2 #size-cells = <1>;
 3 ...
 4 external-bus {
 5     #address-cells = <2>
 6     #size-cells = <1>;
 7     ranges = <0 0  0x10100000   0x10000     // Chipselect 1, Ethernet
 8               1 0  0x10160000   0x10000     // Chipselect 2, i2c controller
 9               2 0  0x30000000   0x1000000>; // Chipselect 3, NOR Flash
10 };
```
