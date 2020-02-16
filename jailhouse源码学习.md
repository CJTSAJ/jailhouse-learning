## IOMMU
- 由于DMA不能像CPU一样通过MMU操作虚拟地址，所以DMA一般需要连续的物理地址。
- Guest OS不能直接操控物理地址，所以一般通过IOMMU(ARM中称为SMMU)进行映射翻译。
- PVU是一种IOMMU，进行2nd stage的翻译

### pvu(ti-pvu.c)
PVU是一种IOMMU，进行2nd stage的翻译，一次映射完所有内存，不支持运行时unmapping内存。

## 中断
三类中断
- SPI(shared peripheral interrupt)：常见的外部设备中断，如手机触屏，1-15
- PPI(private peripheral interrupt)：私有中断，发给特定CPU的中断，16-31
- SGI(software generated interrupt)：软件中断，相当于IPI，32-1019

### GIC寄存器
#### Distributor Registers
Distributor中断路由设备：对整个中断控制设备使能操作；对每个中断优先级控制；设置中断触发方式；对中断目标发送CPU进行设置决定分发到哪个CPU；
记录每个中断的状态；
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/Distributor%20Registers.png)
- GICD_IROUTER寄存器控制中断发给哪个CPU
- GICD_IPRIORITYR优先级寄存器，对于pending状态的irq，有不同的优先级
- GICD_TYPER，获取平台厂商提供的GIC信息，一般提供的信息是该GIC支持多少个IRQ硬件啊中断源，如ITLinesNumber:0xb，则有32*12=384个IRQ
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/GICD_TYPER.png)
- 

