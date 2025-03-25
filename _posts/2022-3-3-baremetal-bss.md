---
layout: post
title: Gcc issue in handling gp register in RISC-V
categories: [Geek]
tags: [riscv,gcc]
---

# RISC-V GCC在处理GP寄存器时的问题

## 背景

`BareMetal`下需要在`start.S`中初始化`bss`段，不然里面的数值不一定为`0`（所有未初始化或初始化为`0`的全局变量会在`bbs`段中）

### start.S

```armasm
# zero-out bss
  la t0, _bss_start
  la t1, _bss_end
_bss_zero:
  SREG zero, (t0)
  add t0, t0, REGBYTES
  blt t0, t1, _bss_zero
```

### link.ldS

```armasm
# bss segment
  _bss_start = .;
  .sbss : {
    *(.sbss .sbss.* .gnu.linkonce.sb.*)
    *(.scommon)
  }
  .bss : {
    *(.bss)
  }
  _bss_end = .;
```

## 踩坑

```armasm
  la t0, _bss_start  # 0x12c, 0x130 使用pc相对位置得到_bss_start
  la t1, _bss_end # 0x134 使用gp相对位置得到_bss_end
```

![](/images/baremetal-bss.png)

## 解决

`start.S`代码片段如下，可能此处有一个编译器的bug，如果`gp`在`t1`之后加载，而`0x134`行中的`t1`相对`gp`的偏移`-1944`是从`ldS`脚本中的`__global_pointer$`计算得到的，那么`t1`就会加载到错误的地址（此时`gp`中的值并非`__global_pointer$`）。

把`BSS`初始化放到`gp`初始化后面就一切正常，具体原因还要仔细再追。

```armasm
# zero-out bss
  la t0, _bss_start
  la t1, _bss_end
_bss_zero:
  SREG zero, (t0)
  add t0, t0, REGBYTES
  blt t0, t1, _bss_zero

# initialize global pointer
.option push
.option norelax
  la gp, __global_pointer$
.option pop
```
