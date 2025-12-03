---
title: Knowledge Fusion of Large Language Models
published: 2025-01-25T00:00:00.000Z
description: ''
image: ''
tags:
  - notes
category: 嗑嗑论文
draft: false
lang: ''
---


<!-- toc -->

- [1. Tasks实现思路](#1-tasks%E5%AE%9E%E7%8E%B0%E6%80%9D%E8%B7%AF)
  * [1.1 Task1 大页表双重映射和重定位](#11-task1-%E5%A4%A7%E9%A1%B5%E8%A1%A8%E5%8F%8C%E9%87%8D%E6%98%A0%E5%B0%84%E5%92%8C%E9%87%8D%E5%AE%9A%E4%BD%8D)
    + [1.1.1 `vm.c` 页表映射原理与实现](#111--vmc-%E9%A1%B5%E8%A1%A8%E6%98%A0%E5%B0%84%E5%8E%9F%E7%90%86%E4%B8%8E%E5%AE%9E%E7%8E%B0)
      - [1.1.1 setup_vm 函数代码详解](#111-setup_vm-%E5%87%BD%E6%95%B0%E4%BB%A3%E7%A0%81%E8%AF%A6%E8%A7%A3)
    + [1.1.2 `head.S` 启动分页与重定位](#112-heads-%E5%90%AF%E5%8A%A8%E5%88%86%E9%A1%B5%E4%B8%8E%E9%87%8D%E5%AE%9A%E4%BD%8D)
  * [1.2 Task 2 多级页表](#12-task-2-%E5%A4%9A%E7%BA%A7%E9%A1%B5%E8%A1%A8)
  * [1.3 Task 1 实现结果](#13-task-1-%E5%AE%9E%E7%8E%B0%E7%BB%93%E6%9E%9C)
- [2. 动手做](#2-%E5%8A%A8%E6%89%8B%E5%81%9A)
  * [2.1 验证内核加载的仍是物理地址](#21-%E9%AA%8C%E8%AF%81%E5%86%85%E6%A0%B8%E5%8A%A0%E8%BD%BD%E7%9A%84%E4%BB%8D%E6%98%AF%E7%89%A9%E7%90%86%E5%9C%B0%E5%9D%80)
    + [2.1.1 打断点](#211-%E6%89%93%E6%96%AD%E7%82%B9)
    + [2.1.2 反汇编结果对比](#212-%E5%8F%8D%E6%B1%87%E7%BC%96%E7%BB%93%E6%9E%9C%E5%AF%B9%E6%AF%94)
  * [2.2 解释同一条指令的不同翻译结果](#22--%E8%A7%A3%E9%87%8A%E5%90%8C%E4%B8%80%E6%9D%A1%E6%8C%87%E4%BB%A4%E7%9A%84%E4%B8%8D%E5%90%8C%E7%BF%BB%E8%AF%91%E7%BB%93%E6%9E%9C)
    + [2.2.1](#221)
    + [2.2.2](#222)
    + [2.2.3](#223)
  * [2.3 中断实现重定位](#23-%E4%B8%AD%E6%96%AD%E5%AE%9E%E7%8E%B0%E9%87%8D%E5%AE%9A%E4%BD%8D)
    + [2.3.1 Linux 内核 `relocate_enable_mmu()` 的实现原理](#231-linux-%E5%86%85%E6%A0%B8-relocate_enable_mmu-%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86)
    + [2.3.2 两种重定位方式是否需要恒等映射？](#232-%E4%B8%A4%E7%A7%8D%E9%87%8D%E5%AE%9A%E4%BD%8D%E6%96%B9%E5%BC%8F%E6%98%AF%E5%90%A6%E9%9C%80%E8%A6%81%E6%81%92%E7%AD%89%E6%98%A0%E5%B0%84)
- [3. 心得&吐槽](#3-%E5%BF%83%E5%BE%97%E5%90%90%E6%A7%BD)

<!-- tocstop -->

# 1. Tasks实现思路

## 1.1 Task1 大页表双重映射和重定位
### 1.1.1  `vm.c` 页表映射原理与实现

实现RISC-V Sv39分页机制。

在Task1中，我们无需实现完整的三级页表，而是利用Sv39支持1GB巨页（Gigapage）的特性，仅使用一级页表（即根页表）就完成对整个物理内存的映射。

![[Sv39_Translation.png]]

上图展示了Sv39的三级页表转换过程。当一个页表项（`PTE`）的`R/W/X`标志位不为0时，它就是一个叶子节点，指向一个物理页。当`R/W/X`为0时，它指向下一级页表。对于1GB的巨页，我们在根页表（L2 Table）中的一个条目就直接指向了1GB的物理内存区域，从而大大简化了映射过程。

核心是在 `vm.c` 中实现 `uint64_t setup_vm(void)` 函数，它负责创建这个初始页表，并为后续的地址重定位做好准备。
#### 1.1.1 setup_vm 函数代码详解

下面我们结合代码，逐步分析 `setup_vm` 的实现流程。

```c
// vm.c
uint64_t setup_vm(void)
{
	// 1. 初始化页目录
	memset(early_pg_dir, 0, PGSIZE);

	/* Lab3 Task1 */

	// 2. 计算页目录索引
	uint64_t gigapage_ppn = VA2VPN2(PHY_START);
	uint64_t gigapage_vpn = VA2VPN2(VM_DIRECT_START);

	// 3. 计算物理页号
	uint64_t ppn = PTE_PA2PPN2(0x80000000ULL);

	// 4. 设置页表项权限
	uint64_t pte_flags = PTE_V | PTE_R | PTE_W | PTE_X | PTE_A | PTE_D;

	// 5. 建立双重映射
	early_pg_dir[gigapage_ppn] = ppn | pte_flags;
	early_pg_dir[gigapage_vpn] = ppn | pte_flags;

	// 6. 准备并返回 satp 值
	uint64_t early_pg_dir_pa = (uint64_t)early_pg_dir;
    return SATP_MODE_SV39 | SATP_PA2PPN(early_pg_dir_pa);
}
```

**1. 初始化页目录**
```c
memset(early_pg_dir, 0, PGSIZE);
```

`early_pg_dir `是我们在BSS段中静态分配的一个4KB对齐的数组，用作根页表。首先将其清零，确保所有页表项（`PTE`）的有效位（`PTE_V`）都为0，避免任何无效的地址转换。

**2. 计算页目录索引**
```c
uint64_t gigapage_ppn = VA2VPN2(PHY_START);
uint64_t gigapage_vpn = VA2VPN2(VM_DIRECT_START);
```

这里我们为两个关键地址计算它们在根页表中的索引。`VA2VPN2` 宏（定义于 `vm.h`）的作用是提取一个64位虚拟地址的最高9位（bits 38-30），这正是Sv39根页表的索引。
- `PHY_START (0x80000000)`：物理内存的起始地址。我们为它创建一个**恒等映射**（identity mapping），即虚拟地址等于物理地址。
- `VM_DIRECT_START (0xffffffd600000000)`：内核的高虚拟地址。我们将用它来创建内核的**高地址映射**。

**3. 计算物理页号（`PPN`）**
```c
uint64_t ppn = PTE_PA2PPN2(0x80000000ULL);
```

`PTE_PA2PPN2` 宏将一个物理地址转换为适用于巨页PTE的物理页号（`PPN`）格式。这两个映射都指向同一个物理内存起点 `0x80000000`。

**4. 设置页表项权限**
```c
uint64_t pte_flags = PTE_V | PTE_R | PTE_W | PTE_X | PTE_A | PTE_D;
```

我们将页表项设置为：有效（V）、可读（R）、可写（W）、可执行（X）、已访问（A）、已修改（D）。这为内核提供了对这块内存区域的完全访问权限。

**5. 建立双重映射**
```c
early_pg_dir[gigapage_ppn] = ppn | pte_flags;
early_pg_dir[gigapage_vpn] = ppn | pte_flags;
```

这是Task1的核心。我们在 `early_pg_dir` 中填入两个条目：  
1. **恒等映射**：`va(0x80000000) -> pa(0x80000000)`。这个映射至关重要，因为在开启分页后，CPU的程序计数器（PC）仍然是物理地址。我们需要一个有效的映射来保证CPU能继续执行下一条指令。  
2. **高地址映射**：`va(0xffffffd600000000) -> pa(0x80000000)`。这是为内核准备的最终运行地址空间。后续的重定位操作会把执行流切换到这个高虚拟地址空间。

**6. 生成satp寄存器值**
```c
uint64_t early_pg_dir_pa = (uint64_t)early_pg_dir;
return SATP_MODE_SV39 | SATP_PA2PPN(early_pg_dir_pa);
```
satp 寄存器需要两个信息：分页模式（Sv39）和根页表的物理地址。
- `SATP_MODE_SV39` 是一个常量，用于设置模式
- `SATP_PA2PPN` 宏将 `early_pg_dir` 的物理地址转换为 `satp` 所需的PPN格式。  
函数将这两个值通过或运算组合后返回

### 1.1.2 `head.S` 启动分页与重定位
`setup_vm` 仅仅是创建了页表，真正启用分页和完成地址空间切换是在汇编代码 `head.S` 中完成的。
```Assembly
# head.S
_start:
    ...
    /* Lab3 Task1 */
    call setup_vm        # 1. 设置页表，返回 SATP 值到 a0
    csrw satp, a0       # 2. 写入 SATP 寄存器，启用 Sv39 分页
    sfence.vma          # 3. 刷新 TLB 和缓存
    call relocate       # 4. 调用重定位函数

    ...

relocate:
    /* Lab3 Task1 */
    li t0, PA2VA_OFFSET # 5. 加载地址偏移量
    add ra, ra, t0      # 6. 重定位返回地址 (ra)
    add sp, sp, t0      # 7. 重定位栈指针 (sp)
    ret                 # 8. 跳转到高虚拟地址继续执行
```

**1. 调用 `setup_vm`**：执行C函数，其返回值（配置好的satp值）存储在 a0 寄存器中。

**2. 启用分页**：`csrw satp, a0` 指令将 a0 的值写入 satp 寄存器。**从这条指令执行完毕的下一刻起，CPU的所有内存访问都将通过我们刚刚建立的页表进行地址转换。**

**3. 刷新TLB**：`sfence.vma` (Supervisor Fence Virtual Memory Address) 是一条关键指令。它用于清空TLB（Translation Lookaside Buffer），这是一个用于缓存地址转换结果的高速缓存。因为我们刚刚更改了页表，必须刷新TLB以确保后续的地址转换使用新的映射关系，而不是陈旧的缓存。

**4. 调用 relocate**：此时分页已开启，但PC、sp（栈指针）和ra（返回地址）等寄存器中仍然是物理地址。得益于我们创建的**恒等映射**，`call relocate` 指令能够正确执行。

**5. relocate 函数**：实现重定位的精髓。

- `li t0, PA2VA_OFFSET`：从 `private_kdefs.h` 中可知 `PA2VA_OFFSET = VM_DIRECT_START - PHY_START`。这个常量就是物理地址和虚拟地址之间的固定偏移量。
- `add ra, ra, t0` 和 `add sp, sp, t0`：将返回地址和栈指针分别加上偏移量，将它们从物理地址转换为内核的高虚拟地址。
- ret：该指令会跳转到 ra 寄存器中的地址。由于 ra 已经被重定位，这条指令执行后，程序将无缝地跳转到 `0xffffffd6`... 开头的高虚拟地址空间继续执行。

至此，我们成功地建立了初始页表，启用了分页，并将内核的执行流从物理地址空间平稳地切换到了高虚拟地址空间，完成了Task1的全部要求。


## 1.2 Task 2 多级页表
本部分由另一个队友完成，按下不表。

## 1.3 Task 1 实现结果
本地测评全部通过：
![[截屏2025-11-11 下午3.46.31.png]]

GDB 调试：
![[截屏2025-11-11 下午3.53.04.png]]
成功映射虚拟地址。

# 2. 动手做
## 2.1 验证内核加载的仍是物理地址
>  动手调试一下刚编译的内核：
>  - 在 `0x80200000` 处打断点，让 GDB 运行到断点处停下来；
>  - 你会发现 GDB 现在没法解析源码位置（比如断点处停下来的时候显示 `0x80200000 in ??()`）。这是因为 GDB 是根据符号的位置来推算当前位于哪个函数的，现在所有符号都变成了虚拟地址，它现在找不到 `0x80200000` 对应哪个函数；
>  - 使用 GDB 或 QEMU Monitor，将从 `0x80200000` 开始的内存打印为指令，与 `vmlinux.asm` 中的反汇编结果对比，它们是否一致？

### 2.1.1 打断点
![[截屏2025-11-11 下午3.18.18.png]]
**观察结果：**
- GDB 在 `0x80200000` 处成功停下
- 但 GDB 无法解析源码位置，显示为 `0x80200000 in ??()`
- **原因分析：** GDB 根据符号表来推算当前函数位置。由于链接器脚本将符号地址设置为虚拟地址（`0xffffffd600200000+`），而当前 PC 是物理地址（`0x80200000`），GDB 无法找到对应的符号，因此无法显示函数名。
### 2.1.2 反汇编结果对比
![[截屏2025-11-11 下午3.24.37.png]]

![[截屏2025-11-11 下午3.24.50.png]]

![[截屏2025-11-11 下午3.25.13.png]]
对比 `vmlinux.asm` 发现：
1. **机器码完全一致：** 所有指令的机器码在 GDB 和 `vmlinux.asm` 中完全相同，证明内核镜像确实被加载到物理地址 `0x80200000`。

2. **地址显示不同：**
- GDB 显示物理地址：`0x80200000`
- vmlinux.asm 显示虚拟地址：`0xffffffd600200000`
- 地址差：`0xffffffd600200000 - 0x80200000 = 0xffffffd5ff800000 = PA2VA_OFFSET`

3. **跳转目标地址不同：**
- GDB: `jal 0x80201f44`（物理地址）
- vmlinux.asm: `jal ffffffd600201f44 <setup_vm>`（虚拟地址）
- 但机器码相同：`0x72d010ef`
## 2.2  解释同一条指令的不同翻译结果
> 提示：
> - 把机器码（用你自己的机器码，和文档不一定相同）输入到 [rvcodec.js · RISC-V Instruction Encoder/Decoder](https://luplab.gitlab.io/rvcodecjs/) 中，看看解码出来是什么。
> - `jal` 指令的寻址方式是什么？
> - 在 `vmlinux.asm` 和 GDB 中，**当前指令地址（PC)** 分别是什么？你能算算跳转目标地址是否和翻译结果一致吗？
### 2.2.1 
![[截屏2025-11-11 下午3.32.24.png]]
**解码结果：**
```
Assembly: jal x1, 0x1f44
Format: J-type
Instruction set: RV64I
```
**指令格式：**
```
jal rd, offset
- rd: 目标寄存器（x1 = ra）
- offset: 有符号立即数偏移量（20 位，编码在指令中）
```

### 2.2.2
`jal` 指令使用 **PC 相对寻址（PC-relative addressing）**：

`跳转目标地址 = PC + offset`

其中：
- `PC`：当前指令地址（Program Counter）
- `offset`：指令中编码的偏移量（有符号 20 位，范围 ±1 MiB）
### 2.2.3
问题 1：当前指令地址（PC）分别是什么？

在 GDB 中：
- 当前指令地址（PC）：`0x80200018`（物理地址）
在 `vmlinux.asm` 中：
- 当前指令地址（PC）：`0xffffffd60020001`8（虚拟地址）
说明： 两者是同一指令在不同地址空间的表示，地址差为 `PA2VA_OFFSET`。
---

问题 2：跳转目标地址是否和翻译结果一致？

计算验证：

GDB 中（物理地址空间）：
- 当前 PC: `0x80200018`
- 目标地址: `0x80201f44`
- 偏移量: `0x80201f44 - 0x80200018 = 0x1f2c`

`vmlinux.asm` 中（虚拟地址空间）：
- 当前 PC: `0xffffffd600200018`
- 目标地址: `0xffffffd600201f44`
- 偏移量: `0xffffffd600201f44 - 0xffffffd600200018 = 0x1f2c`

结论：
- 一致。相对偏移量相同（`0x1f2c`），说明 `jal` 使用 PC 相对寻址，偏移量固定，不依赖绝对地址
## 2.3 中断实现重定位
> 阅读 Linux 内核源码 [`arch/riscv/kernel/head.S`](https://github.com/torvalds/linux/blob/master/arch/riscv/kernel/head.S) 的 `relocate_enable_mmu()` 函数，解释它是怎么实现重定位的。
> 
> 完成 Task 1 后，请你分析这两种重定位方式是否都需要恒等映射？为什么？
### 2.3.1 Linux 内核 `relocate_enable_mmu()` 的实现原理
Linux 内核使用 Trap 方式实现重定位，通过异常处理完成切换：
1. 重定位返回地址： 计算 `PA2VA_OFFSET`，将 ra 加上偏移量
2. 设置异常向量为虚拟地址： 将 `stvec` 指向虚拟地址空间的异常处理代码
3. 加载临时页表（trampoline）： 临时页表只映射第一个超级页（通常是恒等映射）
4. 触发页错误： 执行下一条指令时，由于临时页表映射不完整，触发页错误
5. 异常处理中切换： CPU 跳转到 stvec（虚拟地址），在异常处理中切换到最终页表
关键机制：
- 通过异常自动将 PC 切换到虚拟地址空间
- 临时页表作为过渡，确保异常处理代码可以执行
---
### 2.3.2 两种重定位方式是否需要恒等映射？
跳转方式（Lab3 Task1）：需要恒等映射

原因：
- ret 执行时，PC 仍在物理地址空间
- MMU 需要翻译所有内存访问（包括指令获取）
- 恒等映射确保 ret 指令本身和重定位过程中的内存访问能正常工作
- 如果没有恒等映射，ret 执行时可能无法正确获取指令或访问内存

Trap 方式（Linux 内核）：理论上不需要，但实际可能保留

原因：
- 通过异常机制切换，异常处理代码在虚拟地址空间（`stvec` 指向虚拟地址）
- 不需要通过 ret 从物理地址跳转到虚拟地址
- 临时页表可能包含恒等映射，主要用于简化实现和确保异常处理代码可执行

# 3. 心得&吐槽
Debug 过程中主要卡壳的问题：启用分页机制后遇到的 GDB 连接超时和 Cannot access memory 错误。

**遇到的困难与本质原因分析：**

起初我觉得是页表项（PTE）的内容计算出了问题。通过在 GDB 中设置断点，使用 `x/gx` 命令逐一检查 `early_pg_dir` 在物理内存中的内容，发现 PPN 字段和权限位都符合预期。

进一步分析发现本质原因是 **satp 指针错误**：
问题的根源在于计算 satp 寄存器值时，在错误的时间点（relocate 之前）调用了 `VA2PA` 宏。此时内核运行在 `VA=PA` 的低地址，而 `VA2PA` 是为高地址内核设计的。将一个低地址传入该宏会导致整数下溢，从而计算出一个完全错误的、指向随机物理地址的页表基址。MMU 拿着这个错误的“地址”自然找不到正确的页表，导致地址翻译失败。

**解决：** 在 relocate 之前，获取 early_pg_dir 的物理地址不需要 VA2PA 转换，因为此时它的虚拟地址就是物理地址。

关键修复代码：
```c
uint64_t early_pg_dir_pa = (uint64_t)early_pg_dir;
// 使用正确的物理地址去构建 satp 的值
return SATP_MODE_SV39 | SATP_PA2PPN(early_pg_dir_pa);
```
