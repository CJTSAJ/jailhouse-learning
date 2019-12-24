# ACRN: A Big Little Hypervisor for IoT Development

## 主要工作
- 介绍acrn ，一款面向IoT开发的，轻量级、可扩展、灵活的嵌入式hypervisor
- 提出一种安全有效实时虚拟化方案，如何把实时虚拟化和非实时虚拟化相结合
- 工业级全面评估

#### section2 Related work
Xen的事件通道对嵌入式来说太重量级了、API复杂、难以优化


与xvisor等Hypervisor不同的是，ACRN将主要的外设模块都放在VM中，而不是放在hypervisor中，类似于Xen的Dom0.

#### section3 System Architecture
- 支持三种VM：service VM, user VM, real-time VM；在上面运行的OS分别称Service OS, User OS和Real time OS
- service VM类似于Xen的Dom0；real-time VM是一种特殊的user VM，提供实时服务。
- service VM和ACRN DM管理外设；hypervisor和VMs通过hypercall通信
- RTVM为实时性提供"VM-Exit-less" solution；

#### section4 Design and Implementation
- CPU partitoning：每创建一个vCPU，为其选择一个物理CPU，加入vCPU进入该物理CPU的runqueue，顺序schedule。
- Memory partitioning：除了hypervisor区域，SOS管理所有内存，并且静态分配给其他VMs。hyervisor区域在SOS低地址区域，并且对SOS不可见
- I/O Flow：对Service VM的interrupt devices的访问会trap，其它IO直接访问；对user VM除了pass-through IO，其它全部trap，ACRN维护一个全局IO优先级list；对RTVM不会trap任何IO访问，都是直接访问物理硬件。
- Interrupt Delivery Flow：

#### Real-Time
RTVM为了保证实时直接访问硬件资源，但也限制了资源利用率，限制了灵活性。如何权衡实时性能和设备共享能力是一个挑战

