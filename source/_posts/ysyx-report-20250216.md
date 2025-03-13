---
title: ysyx_report_20250216
date: 2025-02-16 17:52:49
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

2025.02.16

# 近期进展

1. sdram 实现
2. sdram 位扩展
3. sdran 字扩展
4. 移植lightsss

# 卡了一段bug
1. sdram的每个bank的active row各自独立，刚开始只保存了一个active row, 导致读写失败（控制器代码对于已保存的active row 不会先active 而是直接read/write）


# 接下来的任务

接入更多的外设，即接入nvboard

# DONE

