# ARM Virtualization performance and hyrachy implications

从VM到hypervisor切换时，保存寄存器的开销比恢复快，这是由于读取vGIC寄存器导致的。
访问GiC寄存器，Xen-ARM比KVM-ARM快，因为type1 Hypervisor模拟GIC中断控制器，而el1的KVM仅做了部分模拟，会切入EL2状态。
所以访问虚拟GIC寄存器，发送IPIs和接受虚拟中断，Xen-ARM都比KVM-ARM快


- Xen-ARM VM间切换：trap到EL2，做一个EL1的contex switch
- KVM-ARM VM间切换：trap到EL2使能trap和stage2，切换到HostOS一次contex switch，回到EL2使能trap和stage，然后从hostOS切换到新的VM contex switch。

对于I/O延迟，Xen-ARM在两个方向上都比KVM-ARM差。因为Xen需要trap到EL2，然后再进入EL1的Dom0进行IO操作。
IO out操作 Xen-ARM 比KVM-ARM慢很多；
- KVM ARM 发送网路包，trap到hypervisor，context switch到EL1的host，然后host直接发包
- Xen ARM 发包，先trap到hypervisor，然后给Dom0发消息，Dom0进行发包。
  Dom0在空闲和等待IO时会进入idle domain，当xen向dom0发信号时，需要从idle domain切换到Dom0。
  
  
