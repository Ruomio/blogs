---
title: FPGA随笔
date: 2024-02-09 16:46:50
tags: 
    - FPGA
    - Verilog
categories:
    - FPGA
    - gowin
keywords: 
    - FPGA
---

# FGPA

## 整体过程

```markdown
编写 Verilog (.v) 文件：

使用 Verilog 或其他支持的硬件描述语言编写您的 FPGA 设计代码。Verilog 文件描述了电路的行为和结构。
编写 Constraint (.cst) 文件：

编写约束文件，也称为约束条件文件。该文件描述了时序、引脚分配、时钟频率等约束条件，以确保设计在 FPGA 上正确运行。
综合：

使用综合工具，如 Yosys，将 Verilog 代码转换为逻辑网表。综合工具将分析和优化设计，并生成逻辑网表文件。
实现：

使用物理设计工具（例如 nextpnr, Vivado、Quartus Prime、iCEcube2 等），将逻辑网表映射到目标 FPGA 设备的物理资源上。该步骤包括布局（将逻辑元件放置在 FPGA 芯片上的位置）和布线（将逻辑元件之间的连线连接起来）。
生成 Bitstream 文件：

在实现步骤完成后，生成 FPGA 设备可识别的 Bitstream 文件。Bitstream 文件包含了将设计配置到 FPGA 芯片上所需的详细信息，例如逻辑元件的位置、连线信息和时钟设置。
配置 FPGA 开发板：

将生成的 Bitstream 文件加载到 FPGA 开发板上。这通常需要使用特定的软件工具或命令行工具，例如 Vivado Hardware Manager、Quartus Programmer 或 openFPGALoader。
```


# veirlog语言
input/output 参数默认是wire类型。
wire : 组合逻辑，不涉及时序，当电路变化时，跟着变化
reg : 用于时序逻辑，当电路变化时，要等到下一个时钟才变化。
所以 assign 赋值要比在always中赋值快一个周期。



## 工具

### yosys: 综合工具。
```markdown
  1. 启动 Yosys：
  - 在命令行中输入 `yosys` 命令来启动 Yosys 工具。

  2. 导入设计文件：
 
    - 在 Yosys 提示符下，使用
      read_verilog
      命令导入设计文件：
      复制
      yosys> read_verilog design.v

 3. 综合设计：

    - 使用
      synth
      命令对设计进行综合：

      复制

      yosys> synth <synthesis_command>
      <synthesis_command>
      表示您选择的综合命令。Yosys 支持不同的综合命令，如
      synth_ice40

      （针对 Lattice iCE40 FPGA）或
      synth_xilinx
      （针对 Xilinx FPGA）。您可以根据目标 FPGA 平台选择适当的综合命令。
 
 4. 查看综合结果：
    - 使用
      show
      命令查看综合后的设计：
      复制
      yosys> show
      这将显示综合生成的逻辑网表。

 5. 导出网表：
    - 使用
      write_verilog
      命令将综合后的逻辑网表导出为 Verilog 文件：
      复制
      yosys> write_verilog synthesized.v
      这将生成一个名为synthesized.v的文件，其中包含综合后的逻辑网表。
```

### nextpnr-gowin: 实现工具



### openFPGALoader: 烧录工具。



# 高云IDE启动过程

> ```shell
> LD_PRELOAD=/usr/lib/libfreetype.so /opt/Gowin_V1.9/IDE/bin/gw_ide
> ```



# 使用vscode开发环境

> OSS CAD Suite --> Lushay Code(vscode插件) -> 右下角 fpga toolchain
>
> Verilog-HDL 语法高亮



# iverilog 仿真

1. iverilog -o wave led_demo_tb.v led_demo.v
2. vvp -n wave -lxt2
3. gtkwave wave.vcdv



# IP核
## CMT clock manager tiles
1个CMT包含一个MMCM和1个PLL。

### MMCM 

### PPL锁相环
```mermaid
graph LR
	A[ ] --> |F_in| B(N)
	B --> |F_ref| C[PFD]
	C --> D[Charge Pump / Loop Filter &VCO]
	D --> J[F_vco]
	J --> |M| C
	J --> E[K]
	E --> F[F_out1]
	E --> G[F_out1]
	J --> H[V]
	H --> I[F_out2]

```
N：前置分频计数器  
PFD：相位频率检测器  
Charge Pump：电荷泵  
Loop Filter：环路滤波器  
VCO：压控振荡器  
M：反馈乘法器  
K,V：后置分频器

#### 电路原理介绍：

	首先Fin输入时钟，进入N预分频单元，出来的是FREF参考时钟，进入PFD频率相位检测电路，检测VCO反馈回来的时钟信号与参考时钟进行比较，然后将比较结果送入Charge，能得到一个电压信号，电压信号经过一个环路滤波器后，就是一个比较稳定的电压信号，这电压信号再来控制VCO压控振荡器，从来产生一个时钟信号Fvco，再次经过M的一个倍频，输入到PFD，这里就是一个环路反馈，不断调整达到一个平衡。当PFD=M就稳定下来，稳定下来之前需要一定时间。



## 单端口RAM
输入只有一组数据线和一组地址线，只有一个时钟，读写共用地址线。

输出只有一个端口

所以单端口RAM读写操作不能同时进行

## 伪双端口RAM
输入只有一组数据线，两组地址线，两个时钟。

两个输出端口共用输出端口

所以一个端口只读，另一个端口只写，但写入和读取时钟可以不同，且位宽比可以不是1:1，即同时写A与读B可以速率不同。

## 双端口RAM
输入有两组地址线和两组数据线，两个时钟。

输出有两个分别的数据线

所以双端口RAM两个端口都可以分别读写，彼此互不干扰


