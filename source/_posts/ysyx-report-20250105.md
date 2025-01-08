---
title: ysyx_report_20250105
date: 2025-01-05 17:39:47
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

# ysyx分享/汇报
内蒙古大学-彭培智

ysyx_24080025

B阶段

2025-01-05

# 近期进展

1. UART 正确初始化
2. xip flash读取
3. psram 读写
4. QPI模式

# 遇到的问题
1. 可以运行所有cpu-tests, 但是运行rtt时提示编译出的文件大小一异常，通过readelf 工具查看发现从data段开始偏移量错误。


# 接下来的进度
bootloader 二级加载过程

# DONE

