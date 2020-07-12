# Start-to-build-a-RISC-V-superscalar-processor
从零开始写一个基于RV32指令集的超标量处理器
## 立个flag先 2020.7.12
万事开头难，在开始这个项目之前，我花了将近一个星期写了一个 [五级流水线MIPS32处理器](), 在这个过程之中我对于流水线有了一个大概的认知，知道了一个流水线需要哪些的控制信号以及各种hazard的处理方法。在写完这个处理器之后我想继续写下去。对于超标量乱序执行处理器，我只是有一个非常笼统的认知，知道需要哪些额外的硬件单元；但是当自己要真正开始动手写的时候又发现无从下手，对于其中的保留站、reorder buffer，甚至cache，更是完全不知道如何用Verilog去描述。所以迷茫了两三天，我终于决定不能再想一个无头苍蝇一样到处搜索资料却迟迟不开始动手。我打算从一个最简单的双发射顺序执行流水线开始动手，最后一步步添加乱序执行的功能，并且处理其中的数据依赖问题。Just do it.
### 流水线基本架构
相比与传统的五级流水线处理器，超标量处理器由于同时需要执行多条指令，所欲需要加一个issue stage.在issue stage流水线需要完成指令的发射以及部分的hazard处理。对于最初版的超标量流水线处理器，我打算先实现两个functional unit,一个专门处理跳转指令，另一个处理访存指令，两者同时又都可以执行逻辑和算术运算指令。
