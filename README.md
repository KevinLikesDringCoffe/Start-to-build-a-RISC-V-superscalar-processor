# Start-to-build-a-RISC-V-superscalar-processor
从零开始写一个基于RV32指令集的超标量处理器
## 立个flag先 2020.7.12
万事开头难，在开始这个项目之前，我花了将近一个星期写了一个 [五级流水线MIPS32处理器](https://github.com/KevinLikesDringCoffe/MIPS32-pipelined-processor), 在这个过程之中我对于流水线有了一个大概的认知，知道了一个流水线需要哪些的控制信号以及各种hazard的处理方法。在写完这个处理器之后我想继续写下去。对于超标量乱序执行处理器，我只是有一个非常笼统的认知，知道需要哪些额外的硬件单元；但是当自己要真正开始动手写的时候又发现无从下手，对于其中的保留站、reorder buffer，甚至cache，更是完全不知道如何用Verilog去描述。所以迷茫了两三天，我终于决定不能再想一个无头苍蝇一样到处搜索资料却迟迟不开始动手。我打算从一个最简单的双发射顺序执行流水线开始动手，最后一步步添加乱序执行的功能，并且处理其中的数据依赖问题。Just do it.
### 流水线基本架构
相比与传统的五级流水线处理器，超标量处理器由于同时需要执行多条指令，因此需要加一个issue stage.在issue stage流水线需要完成指令的发射以及部分的hazard处理。对于最初版的超标量流水线处理器，我打算先实现两个functional unit,一个专门处理跳转指令，另一个处理访存指令，两者同时又都可以执行逻辑和算术运算指令。
### 指令取指
取指阶段一次取两条指令，同时PC = PC + 8
### 指令解码
在解码阶段需要从寄存器组中取出alu运算的操作数以及指令类型，以便后续将指令分发到合适的功能单元。因为一次需要发射两条指令，所以需要有两个解码单元，每个单元解码一条指令。
### 指令发射
采用动态多发射，在发射阶段会处理冲突。在这个超标量流水线需要处理的Data hazard只有RAW。如果同时发射的两条指令的目的寄存器相同，则第二条指令在下一个时钟周期发射，防止出现同时有两条指令写入同一个寄存器而出现结构冲突。并且当两条指令全都发射时再取指下两条指令。如果前后指令存在RAW依赖，则第二条指令会在下一个时钟周期发射（如果前一条指令的结果需要访存得到，那么需要下一跳指令暂停两个周期）并且接受第一条指令forwarding的数据。
### 指令执行
指令执行需要两个时钟周期。指令执行时分为两个功能单元（两条流水线）。第一指令流水线处理算术逻辑运算指令以及分支指令，第二指令流水线处理算术逻辑运算以及访存指令。分支指令的跳转地址在执行的第一个周期给出。
### 指令写回
指令结果在这个周期写回到寄存器。
## 2020.7.13 进展
今天写完了指令解码单元，RV32I指令集和MIPS32大同小异，在RV32I中，源寄存器和目的寄存器地址在指令中的位置是固定的，为解码提供了方便，代价是立即数的产生方式会变得繁琐。具体指令格式可以参考[RISC-V指令集手册]()。接下来开始做指令发射单元...
## 2020.7.14 进展
动态指令发射实在是太令人头大了...我不打算用scoreboard而是用传统的判断方法去做信号旁路，然而把自己整蒙了，心态小崩。好消息是今天收到了《超标量处理器设计一书》，我打算先看书吧，毕竟知识实在是太匮乏了...
## 2020.7.20
大概把书翻了一遍...然而很多概念只是蜻蜓点水地掠过了...接下来开始着手设计了。其实核心还是处理超标量处理器中的发射环节，发射环节需要决定两条指令是否可以同时发射   
- 两条没有数据依赖的ALU指令必然可以同时发射
- 没有数据依赖的MEM类型指令和ALU指令
- 没有数据依赖的MEM类型指令和Branch指令
- 没有数据依赖的Branch和ALU指令
- 如果两条ALU指令存在RAW依赖，则第二条ALU指令延迟一周期执行，同时不再取新的指令
- 如果两条ALU指令，或者一条ALU与一条LOAD的目的寄存器相同，则第一条指令不会被执行（执行了也是白执行）
- 
