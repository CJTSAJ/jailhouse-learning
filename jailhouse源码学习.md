## IOMMU
- 由于DMA不能像CPU一样通过MMU操作虚拟地址，所以DMA一般需要连续的物理地址。
- Guest OS不能直接操控物理地址，所以一般通过IOMMU(ARM中称为SMMU)进行映射翻译。
- PVU是一种IOMMU，进行2nd stage的翻译
