---
layout: post
title: DMA coherence of macb driver on RISC-V systems
categories: [Geek]
tags: [linux, dma]
---

## RISC-V系统上的linux macb驱动的DMA一致性问题

macb驱动在DMA内存一致性上使用了两种致性操作，即

1. DMA coherent -> dma_alloc_coherent/dma_free_coherent

riscv下的实现是使用svpbmt，在页表中配置NO_CACHE来实现读写一致性。此框架目前有问题，在开启svpbmt后，NO_CACHE并不能正确传递flag。

macb中的tx/rx的控制内存tx_ring/rx_ring使用dma coherent。

2. 流式DMA -> dma_map_single/dma_unmap_single

riscv下的实现是使用dma_engine的cache_op来单向操作流式dma内存，即对DMA_TO_DEVICE/DMA_FROM_DEVICE进行clean/flush操作。最早由平头哥的cacheflush.c实现，目前在最新6.1的代码中进行重构，但还存在bug。

macb中的tx_buffer/rx_buffer使用流式DMA

---------------------------------------------

Ref:
