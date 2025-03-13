---
title: ysyx_report_20250302
date: 2025-03-02 17:33:29
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

2025.03.02

# 近期进展

1. 接入nvboard gpio
2. lightsss 改bug，assert失败或者ctrl c 终止时，子进程无法正常退出，造成内存泄漏；新增信号处理函数，支持assert失败时也保存波形。

# 卡了一段bug
1. nvboard uart 试了很多办法都无法正常打印（打印乱码）
    思路：
    -- 计算仿真频率， 8000 insts / s， 每条指令 36 时钟周期， 故仿真频率 288KHz，以此设置nvboard除数： 不正确
    -- 查波形，nvboard在每条指令执行后更新，查看tx信号引脚，发现14条指令后结束一字节的发送并发送停止位，到第21条指令后开始下一字节的发送，但是将nvboard的除数设置为14 or 21 也均不对， 目前卡在了这里，想问下大家思路。

# 接下来的任务

通过nvboard测试串口输入功能

# DONE

