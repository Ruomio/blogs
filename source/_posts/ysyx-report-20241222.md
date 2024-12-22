---
title: ysyx_report_20241222
date: 2024-12-22 18:23:21
tags:
    - verilog
    - riscv32
    - C
    - makefile
categories:
    - ysyx
keywords:
    - riscv32e
---

# 一生一芯组会汇报/分享

内蒙古大学-彭培智

ysyx_24080020

B阶段

2024.12.22

# 近期进展
1. 仲裁器
2. xbar, uart, clint
3. 接入ysyx_soc, 将axi-lite 修改为axi-full
4. 修改nemu，添加mrom和sram, 为soc添加difftest
5. 折腾 —— 为npc实现与sram之间的突发传输（非内存对齐状态下多读写事物）
6. 全局变量写入，bootloader 将data段复制到sram，并将bss段初始化

# 接下来的进度

通过串口输出

# 卡进度
还以为 'mem-test测试内存访问' 就是要实现突发传输呢，结果才发现非对齐访问内存时，编译器优化只会生成lb,sb, 故npc与sram之间不会发生突发传输...

白实现了 :(

# DONE
