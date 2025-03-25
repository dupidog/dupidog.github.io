---
layout: post
title: DMA coherence of macb driver on RISC-V systems
categories: [Geek]
tags: [linux, dma]
---

## RISC-V系统上的linux macb驱动的DMA一致性问题

### 1. Linux 5.19及以前版本中RISC-V架构下的DMA一致性bug

#### 1.1 pgprot_wirtecombine / pgprot_noncached
	
_PAGE_MTMASK/_PAGE_IO /_PAGE_NOCACHE没有正确传递宏定义的svpbmt page table flag，均为0，需要暂时作如下修改

#### 1.2 cache_op
	
由于dma engine需要实现cache_op对流式dma内存进行cache_op操作。这边dma相关mapping接口给出的是物理地址，而实际进行cache_op操作的CMO指令需要虚拟地址，但riscv架构没有定义phys_to_virt函数。这边需要使用__va(pa)来进行转换

~~~C
#define phys_to_virt(x) __va(x)
~~~

### 2. T-head的实现与svpbmt的区别

DMA coherent和流式内存分配，由架构的特性来保证一致性，因架构而异。通常页表属性中会有buffer和cache属性来标记当前内存页的缓存同步行为。

页表属性参考pgtable.h
缓存同步行为参考cacheflush.c

dma相关代码为通用代码，dma_alloc_coherent和dma_map_single分配的内存会在页表上做noncache或writecombine。页表上的权限位各架构不同，由架构实现。具体可以参考arch/xxx/include/asm目录中的pgtable.h和pgtable-bits.h

#### 2.1 T-head C910

![](/images/macb-driver-on-riscv-2.png)

这一段SEC,SHARE,BUF,CACHE,SO由平头哥扩展

![](/images/macb-driver-on-riscv-3.png)

下面是平头哥对noncache和writecombine内存属性的页表配置

![](/images/macb-driver-on-riscv-4.png)

在cacheflush.c中 (arch/riscv/mm/cacheflush.c) 对cache invald/clean实际操作的实现

![](/images/macb-driver-on-riscv-5.png)

#### 2.2 RISC-V svpbmt

Linux 5.19及之前的版本没有对RISC-V svpbmt做实现，如果使用RISC-V svpbmt，需要注意作以下几点：
* pgtable-bit.h中的页表各bit定义是否有自己的修改
* pgprot_noncached / pgprot_writecombine的实际操作
* 在arch/riscv/mm/cacheflush.c中使用CPU支持的CMO指令实现cache invald/clean操作

### 3. 结论

macb驱动在DMA内存一致性上使用了两种致性操作，即

#### 3.1 DMA coherent -> dma_alloc_coherent/dma_free_coherent

riscv下的实现是使用svpbmt，在页表中配置NO_CACHE来实现读写一致性。此框架目前有问题，在开启svpbmt后，NO_CACHE并不能正确传递flag。

macb中的tx/rx的控制内存tx_ring/rx_ring使用dma_coherent。

#### 3.2 流式DMA -> dma_map_single/dma_unmap_single

riscv下的实现是使用dma_engine的cache_op来单向操作流式dma内存，即对DMA_TO_DEVICE/DMA_FROM_DEVICE进行clean/flush操作。最早由平头哥的cacheflush.c实现，目前在最新6.1的代码>中进行重构，但还存在bug。

macb中的tx_buffer/rx_buffer使用流式DMA。

---------------------------------------------

Ref:

[http://www.wowotech.net/kernel_synchronization/Why-Memory-Barriers.html](http://www.wowotech.net/kernel_synchronization/Why-Memory-Barriers.html )

