## root-cell配置
```
struct {
	struct jailhouse_system header;
	__u64 cpus[1];
	struct jailhouse_memory mem_regions[4];
	struct jailhouse_irqchip irqchips[1];
	struct jailhouse_pci_device pci_devices[1];
	struct jailhouse_pci_capability pci_caps[11];
}
```

#### struct jailhouse_system
```
struct jailhouse_system {
	char signature[6];
	__u16 revision;
	__u32 flags;

	/** Jailhouse's location in memory */
	struct jailhouse_memory hypervisor_memory;
	struct jailhouse_console debug_console;
	struct {
		__u64 pci_mmconfig_base;
		__u8 pci_mmconfig_end_bus;
		__u8 pci_is_virtual;
		__u16 pci_domain;
		union {
			struct {
				__u16 pm_timer_address;
				__u32 vtd_interrupt_limit;
				__u8 apic_mode;
				__u8 padding[3];
				__u32 tsc_khz;
				__u32 apic_khz;
				struct jailhouse_iommu
					iommu_units[JAILHOUSE_MAX_IOMMU_UNITS];
			} __attribute__((packed)) x86;
			struct {
				u8 maintenance_irq;
				u8 gic_version;
				u64 gicd_base;
				u64 gicc_base;
				u64 gich_base;
				u64 gicv_base;
				u64 gicr_base;
			} __attribute__((packed)) arm;
		} __attribute__((packed));
	} __attribute__((packed)) platform_info;
	struct jailhouse_cell_desc root_cell;
} __attribute__((packed));
```

- .signature: 可取值JAILHOUSE_SYSTEM_SIGNATURE、JAILHOUSE_CELL_DESC_SIGNATURE
- .revision: 一般取JAILHOUSE_CONFIG_REVISION，jailhouse_cmd_enable和jailhouse_cmd_cell_create做检查。
- .flags
  - JAILHOUSE_SYS_VIRTUAL_DEBUG_CONSOLE: 允许root cell从虚拟控制台读取
  - JAILHOUSE_CELL_VIRTUAL_CONSOLE_PERMITTED: 允许non-root cell调用dbg_putc hypercall写hypervisor的控制台，用于debug，root cell据此可以读inmate的输出
  - JAILHOUSE_CELL_VIRTUAL_CONSOLE_ACTIVE: If JAILHOUSE_CELL_VIRTUAL_CONSOLE_ACTIVE is set, inmates should use the virtual console. This flag implies JAILHOUSE_CELL_VIRTUAL_CONSOLE_PERMITTED.
- .hypervisor_memory: 
  - .phys_start: 物理起始地址，根据不同板子设置不同的地址
  - .size: 为jailhouse预留的内存大小，根据不同的板子一般取4MB和64MB
  - .virt_start
  - .flags
- debug_console:
  - .address
  - .size
  - .type: 根据平台选择不同的类型，类型定义于include/jailhouse/console.h，例：JAILHOUSE_CON_TYPE_IMX
  - .flags: 同见include/jailhouse/console.h
 ```
struct jailhouse_console {
	__u64 address;
	__u32 size;
	__u16 type;
	__u16 flags;
	__u32 divider;
	__u32 gate_nr;
	__u64 clock_reg;
} __attribute__((packed));
```
- .platform_info: 若arm需要配置pci，需要虚拟pci，pci_is_virtual置为1。driver/pci.c中jailhouse_pci_virtual_root_devices_add函数会检查pci_is_virtual并调用函数创建虚拟pci(暂时未实现)。
```
.platform_info = {
			.arm = {
				.gic_version = 3,
				.gicd_base = 0x38800000,
				.gicr_base = 0x38880000,
				.maintenance_irq = 25,
			},
		}
```
- .root_cell: 
```
.root_cell = {
			.name = "RootCell",
			.cpu_set_size = sizeof(config.cpus),
			.num_memory_regions = ARRAY_SIZE(config.mem_regions),
			.num_irqchips = ARRAY_SIZE(config.irqchips),
			.num_pio_regions = ARRAY_SIZE(config.pio_regions),
			.num_pci_devices = ARRAY_SIZE(config.pci_devices),
			.num_pci_caps = ARRAY_SIZE(config.pci_caps),
		},
```
```
struct jailhouse_cell_desc {
	char signature[6];
	__u16 revision;

	char name[JAILHOUSE_CELL_NAME_MAXLEN+1];
	__u32 id; /* set by the driver */
	__u32 flags;

	__u32 cpu_set_size;
	__u32 num_memory_regions;
	__u32 num_cache_regions;
	__u32 num_irqchips;
	__u32 num_pio_regions;
	__u32 num_pci_devices;
	__u32 num_pci_caps;

	__u32 vpci_irq_base;

	__u64 cpu_reset_address;
	__u64 msg_reply_timeout;

	struct jailhouse_console console;
} __attribute__((packed));
```

- .cpus: 4个cpu: {0xf}; 128个cpu: {0xffffffffffffffff, 0xffffffffffffffff}
```
.cpus = {
		% for n in range(int(cpucount / 64)):
		0xffffffffffffffff,
		% endfor
		% if (cpucount % 64):
		${'0x%016x,' % ((1 << (cpucount % 64)) - 1)}
		% endif
	},
```
		

- .mem_regions: 内存分配，数组形式，每个表示一个区域，比如：MMIO IVSHMEM RAM；根据falgs设置内存可读、可写、可执行、DMA、等特性
```
struct jailhouse_memory {
	__u64 phys_start;
	__u64 virt_start;
	__u64 size;
	__u64 flags;
} __attribute__((packed));
```

- .irqchips: 中断处理配置，x86对应IOAPIC, arm对应gic配置。
```
struct jailhouse_irqchip {
	__u64 address;
	__u32 id;
	__u32 pin_base;
	__u32 pin_bitmap[4];
} __attribute__((packed));
```
```
.irqchips = {
		/* GIC */ {
			.address = 0x38800000,
			.pin_base = 32,
			.pin_bitmap = {
				0xffffffff, 0xffffffff, 0xffffffff, 0xffffffff,
			}
	},
```

- .pci_devices: arm中的pci_devices type都是IVSHMEM，用于cell间通信使用。pci设备配置，bdf,type,iommu;
```
struct jailhouse_pci_device {
	__u8 type;
	__u8 iommu;
	__u16 domain;
	__u16 bdf;
	__u32 bar_mask[6];
	__u16 caps_start;
	__u16 num_caps;
	__u8 num_msi_vectors;
	__u8 msi_64bits:1;
	__u8 msi_maskable:1;
	__u16 num_msix_vectors;
	__u16 msix_region_size;
	__u64 msix_address;
	/** Memory region index of virtual shared memory device. */
	__u32 shmem_region;
	/** PCI subclass and interface ID of virtual shared memory device. */
	__u16 shmem_protocol;
	__u8 padding[2];
} __attribute__((packed));
```

- .pci_caps: arm通常不配置该项，用于x86

## non-root cell 配置
```
struct {
	struct jailhouse_cell_desc cell;
	__u64 cpus[1];
	struct jailhouse_memory mem_regions[3];
	struct jailhouse_irqchip irqchips[1];
}
```
```
.cell = {
		.signature = JAILHOUSE_CELL_DESC_SIGNATURE,
		.revision = JAILHOUSE_CONFIG_REVISION,
		.name = "gic-demo",
		.flags = JAILHOUSE_CELL_PASSIVE_COMMREG,

		.cpu_set_size = sizeof(config.cpus),
		.num_memory_regions = ARRAY_SIZE(config.mem_regions),
		.num_irqchips = 0,
		.num_pci_devices = 0,

		.console = {
			.address = 0x30860000,
			.type = JAILHOUSE_CON_TYPE_IMX,
			.flags = JAILHOUSE_CON_ACCESS_MMIO |
				 JAILHOUSE_CON_REGDIST_4,
		},
	},
```
