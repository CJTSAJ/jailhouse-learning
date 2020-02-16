### GIC结构
gic主要分为两部分
- distributor：中断路由设备；对整个中断控制设备使能操作；对每个中断优先级控制；设置中断触发方式(边缘触发和电平触发)；对中断目标发送CPU进行设置决定分发到哪个CPU；记录每个中断的状态；
- cpu interface：将中断发给CPU；对中断进行认可(acknowledging an interrupt)；设置中断优先级屏蔽；中断完成识别(inacting an interrupt)；定义中断抢占策略。

### 中断
三类中断
- SPI(shared peripheral interrupt)：常见的外部设备中断，如手机触屏，1-15
- PPI(private peripheral interrupt)：私有中断，发给特定CPU的中断，16-31，
- SGI(software generated interrupt)：软件中断，用于给其它核发信号，相当于IPI，32-1019

CPU以CLUSTER为单位管理，如10个cpu，其中4个小核为CLUSTER_2，另外4个CPU为CLUSTER_1，两个大核为CLUSTER_0，这样有利于CPU的上电和频率管理

![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/cpu%E6%8B%93%E6%89%91%E7%BB%93%E6%9E%84.png)

#### 时钟
arm的timer在core内部，以PPI中断形式发送
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/arm%E6%97%B6%E9%92%9F.png)

### 中断流程(gicv3)
#### 通过distributor(比如SPI)
- 外设发起中断，发给distributor
- distributor分发给合适re-distributor
- re-distributor将中断发送给cpu interface
- cpu interface产生合适的中断异常给处理器

#### 不通过distribuor(如LPI)
LPI是gicv3新添加的中断类型，不经过distributor，没有pending状态，目前没有具体应用。
- 外设发起中断，发送给ITS
- ITS分析中断。分发给re-distributor
- re-distributor将中断信息发给cpu interface
- cpu interface产生合适的中断异常给处理器

![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/%E5%A4%96%E8%AE%BE%E4%B8%AD%E6%96%AD%E6%B5%81%E7%A8%8B.png)


#### Distributor Registers(GICD)
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/distributor.png)
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/Distributor%20Registers.png)
- GICD_IROUTER寄存器控制中断发给哪个CPU
- GICD_IPRIORITYR优先级寄存器，对于pending状态的irq，有不同的优先级
- GICD_TYPER，获取平台厂商提供的GIC信息，一般提供的信息是该GIC支持多少个IRQ硬件啊中断源，如ITLinesNumber:0xb，则有32*12=384个IRQ
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/GICD_TYPER.png)
- GICD_SGIR，通过写该寄存器来产生软中断
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/GICD_SGIR.png)
- 


![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/cpu%20interface.png)


## jaihouse中断源码解读
struc irqchip管理中断操作，在初始化的时候irqchip.c文件的irqchip_cpu_init函数内根据gic的版本(gicv2 gicv3)进行初始化。
其各个中断操作函数分别实现在gic-v2.c和gic-v3.c文件中。
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/irqchip.png)

- send_sgi函数(对应gic-v3.c中gicv3_send_sgi函数)写GICD_SGIR寄存器发送软中断，
- inject_irq函数(gic-v3.c中gicv3_inject_irq实现)写list registers寄存器，向pending队列注入中断。


pending irq被放在list registers中

## IOMMU
- 由于DMA不能像CPU一样通过MMU操作虚拟地址，所以DMA一般需要连续的物理地址。
- Guest OS不能直接操控物理地址，所以一般通过IOMMU(ARM中称为SMMU)进行映射翻译。
- PVU是一种IOMMU，进行2nd stage的翻译

### pvu(ti-pvu.c)
PVU是一种IOMMU，进行2nd stage的翻译，一次映射完所有内存，不支持运行时unmapping内存。

### gic寄存器
MMIO寄存器
- GICC: cpu interface寄存器
- GICD: distributor寄存器
- GICH: virtual interface控制寄存器，在hypervisor模式访问
- GICR: redistributor寄存器
- GICV: virtual cpu interface寄存器
- GITS: ITS寄存器
系统寄存器访问的寄存器：
- ICC: 物理 cpu interface 系统寄存器
- ICV: 虚拟 cpu interface 系统寄存器
- ICH： 虚拟 cpu interface 控制系统寄存器

#### 


redist_base：Redistributor Registers映射的起始地址

每个核都有一个MPIDR寄存器，每个核的MPIDR寄存器值都不同，用来表示该核
