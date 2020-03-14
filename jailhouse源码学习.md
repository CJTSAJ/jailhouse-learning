### GIC结构
gic主要分为两部分
- distributor：中断路由设备；对整个中断控制设备使能操作；对每个中断优先级控制；设置中断触发方式(边缘触发和电平触发)；对中断目标发送CPU进行设置决定分发到哪个CPU；记录每个中断的状态；
- cpu interface：将中断发给CPU；对中断进行认可(acknowledging an interrupt)；设置中断优先级屏蔽；中断完成识别(inacting an interrupt)；定义中断抢占策略。

与gicv2不同的是gicv3将cpu interface从gic中抽离出来放在了core内部。core需要频繁访问cpu interface，这样做可以减少访问延迟。

### 中断
三类中断
- SPI(shared peripheral interrupt)：常见的外部设备中断，如手机触屏，1-15
- PPI(private peripheral interrupt)：私有中断，发给特定CPU的中断，16-31，
- SGI(software generated interrupt)：软件中断，用于给其它核发信号，相当于IPI，32-1019

CPU以CLUSTER为单位管理，如10个cpu，其中4个小核为CLUSTER_2，另外4个CPU为CLUSTER_1，两个大核为CLUSTER_0，这样有利于CPU的上电和频率管理

![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/cpu%E6%8B%93%E6%89%91%E7%BB%93%E6%9E%84.png)

clusterid保存aff3 aff2 aff1的信息，targetid保存aff0的信息
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/sgi_code.png)

#### 中断分组
gicv3将中断分为group0和group1，使用寄存器GICD_IGROUPn来对每个中断进行分组
- group0：安全中断，提供给EL3使用
- group1：又分为2组，分别为安全中断和非安全中断

#### 时钟
ARM时钟主要是：SOC上的system counter(多个processor共享)以及附着在processor上的Timer
- system counter:计算输入时钟已经过了多少个clock，从0开始，每个clock，counter加一；system counter的值需要分发到各个timer中
- Timer：就是定时器，可以指定一段时间，当时间到了就会assert一个外部输出信号(可以输出到GIC，作为一个interrupt)
arm的timer在core内部，以PPI中断形式发送
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/arm%E6%97%B6%E9%92%9F.png)
</br></br>
时钟寄存器部分代码
![](https://github.com/CJTSAJ/jailhouse-learning/blob/master/picture/%E6%9C%AA%E5%91%BD%E5%90%8D%E5%9B%BE%E7%89%87.png)
</br></br></br></br>
处理器通过CNTPCT寄存器获取system counter的值，即physical counter；通过CNTVCT寄存器访问virtual counter。
</br></br>
每个timer都会有多个timer：
- 对于不支持security extension的SOC（不支持security extension也就意味着 不支持virtualization extension），timer实际上有两个，一个是physical timer，另外一个是virtual timer。虽然有两个，不过从行为上看，virtual timer和physical timer行为一致
- 对于支持security extension但不支持virtualization extension的SOC，每个cpu有三个timer：Non-secure physical timer，Secure physical timer和virtual timer
- 对于支持virtualization extension的SOC，每个cpu有四个timer：Non-secure PL1 physical timer，Secure PL1 physical timer，Non-secure PL2 physical timer和virtual timer
</br></br>
每个timer都有三个寄存器
- 64-bit CompareValue register：一个64 bit unsigned upcounter；physical counter - CompareValue >= 0的话，触发中断，向GIC发中断
- 32-bit TimerValue register：随着system counter的值不断累加，TimerValue register的值在递减，当值<=0的时候，便会向GIC触发中断
- 32-bit控制寄存器。该寄存器主要对timer进行控制，具体包括：enable或是disable该timer，mask或者unmask该timer的output signal（timer interrupt）

### 中断流程(gicv3)
#### 通过distributor(比如SPI)
- 外设发起中断，发给distributor，distributor把收集来的中断先缓存起来(pending状态)，然后根据优先级先后发出去
- distributor分发给合适re-distributor
- re-distributor将中断发送给cpu interface
- 中断认可：cpu响应该中断，中断从pending变为active状态，通过访问GICC_IAR认可group0中断，访问GICC_AIAR认可group1中断。
- 中断完成：分为两步
    - 优先级重置(priority drop)：将当前中断优先级重置，以便能响应较低优先级中断(以防这个中断处理时间过长)。group0中断通过写GICC_EOIR寄存器，group1通过写GICC_AOIR寄存器。
    - 中断无效(interrupt deactivation)：cpu读取一个中断，即读interface的寄存器，cpu interface产生合适的中断异常给处理器

#### 不通过distribuor(如LPI)
LPI是gicv3新添加的中断类型，不经过distributor，没有pending状态，目前没有具体应用。
- 外设发起中断，发送给ITS
- ITS分析中断。分发给re-distributor
- re-distributor将中断信息发给cpu interface
- 中断认可：cpu响应该中断，中断从pending变为active状态，通过访问GICC_IAR认可group0中断，访问GICC_AIAR认可group1中断。
- 中断完成：分为两步
    - 优先级重置(priority drop)：将当前中断优先级重置，以便能响应较低优先级中断(以防这个中断处理时间过长)。group0中断通过写GICC_EOIR寄存器，group1通过写GICC_AOIR寄存器。
    - 中断无效(interrupt deactivation)：cpu读取一个中断，即读interface的寄存器，cpu interface产生合适的中断异常给处理器

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
