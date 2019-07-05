# G2 LLVM 代数优化

* 队员
  * PB16110386-丁峰： [df12138@mail.ustc.edu.cn](mailto:df12138@mail.ustc.edu.cn)
  * PB16110846-于颖奇：[yu971207@mai.ustc.edu.cn](mailto:yu971207@mai.ustc.edu.cn)
  * PB16111428-马凯：[ksqsf@mail.ustc.edu.cn](mailto:ksqsf@mail.ustc.edu.cn)
  * PB16060117-曾明亮： [zml2016@mail.ustc.edu.cn](mailto:zml2016@mail.ustc.edu.cn)

* 目录名：G2-AlgOpt

* 项目说明：

  通过调研，我们发现clang里面用for循环对一个多项式求和会直接优化成一个公式，但gcc做不到,只能向量化。icc甚至可以推断每个分支被执行的概率，从而进行进一步的优化。我们的想法是让llvm IR也做到对一个复杂的多项式进行代数上的优化。LLVM 的优化非常精妙且充分，大部分优化都可以做到，于是我们将目光转向了一个新的 LLVM 中尚未实现的优化，即将算术表达式直接化简。
  
  具体来说，我们发现，在-O2的编译条件下，clang会将循环次数较少的循环进行展开，但是对一个复杂的多项式求解，却没有进行优化，只能按部就班的照着式子运算。比如对于一个多项式`y=(x+1)(x-1)-x*x`，很明显，不论x的值为多少，在y,x都为`int`的情况下，y的值始终为-1。而clang在处理这边时比较保守，只会按部就班地运算。我们所做的工作就是试图将一个多项式进行化简（直接求出答案或者提取公因式等），将计算步骤尽量减少。

* Q：
  
** clang里面用for循环对一个多项式求和直接优化成公式的原理是什么？ 

clang 在 -O2 下会将循环次数不多的for循环展开，进行优化，但是不会优化其中的多项式计算。在 -O3 下，clang会将它们变成向量运算的形式。除此之外，我们发现，clang会把一个高次数的单项式进行优化。比如 `y=pow(x,100)` 或者写成 `for(int i=0;i<128;i++)  total *= x;`时，clang会使用反复平方再相乘相应的值来快速计算，而不是进行128次循环。

事实上，Clang 并不对 AST 进行优化，Clang 中的优化仅有少量在 Codegen 时发生，其他均在 LLVM IR 上进行。

** 在 llvm IR上做对一个复杂的多项式进行代数优化的依据是哪些？

LLVM 中的做法是：
1. 如果使用循环计算，且循环次数是常数且较小，LLVM 会直接将循环展开然后使用 SLSR。
2. 如果循环常数次但次数过多，则 LLVM 并不会将循环展开，但会考虑其他优化如 Load hoisting。

对于我们的项目，我们的理论依据有：
1. 与多项式有关的算法，如快速幂等。
2. 符号计算和多项式计算：本质上我们做的是一个符号多项式计算器，它通过遍历 IR 来构造并化简多项式。

** 计划处理哪些复杂的多项式的patterns？

由于时间仓促且任务较重，小组成员决定先在一个子函数中的一个基本块进行操作，并且假设优化的多项式仅仅含有加乘减等运算，又假设多项式内仅有一个变量。这样的多项式小组已在6号将其识别出来。

** 识别和优化的主要原理是否有考虑？

对多项式的化简，首先需要识别目标多项式是否是我们想优化的，若不是，那么我们不会改动该多项式，然后然后我们可以自下而上构建一棵树，树的每一个叶子节点都表示组成多项式的变量或常量，一个节点的父节点表示一次运算的结果，同时还需要获得多项式的系数以及所对应的次数，并把该多项式拆分为若干个因式的积（如果可能的话）。
  
** llvm 中类似的处理是哪些？

与我们工作最接近的是 InstCombine。但有很多标量优化都有类似的操作。

1. LLVM 中的 InstCombine 有类似的工作。InstCombine 实现了一个综合性的指令简化系统，可以做到常量折叠和因式分解等工作。它会利用交换律将二元运算的操作数以一定顺序存放，并在编译时检查两个指令操作数是否相同，若相同则直接提取公因子。
2. 此外，Straight Line Strength Reduce 中也有类似工作。它的原理是，若有 S1 和 S2 两个指令，S1 支配 S2，那么就可以在 S1 结果的基础上计算 S2。通过评估计算代价，可以在一定程度上优化计算效率。

** 打算利用LLVM的哪些开展你们的工作 ？

LLVM Pass。我们现在只考虑了一个简单的情形，即单参数、单输出、只有一个基本块、只有加减乘的函数做多项式计算。由于情形比较简单，我们没有使用 LLVM 自带的分析工具，而是手动通过遍历 IR 来收集信息。


## 设计思想以及处理流程
1. 首先进行模式识别。遍历整个函数的基本块中所有的语句，如果该基本块中所有的指令均有简单的`Add`,`Sub`,`Mul`,`Xor`组成，那么就可以对其进行AlgOpt的优化。
2. 提取多项式系数，转化成一个DAG，然后自下而上进行运算，不断地进行合并同类项，得到一个最终的结果，如果这个这结果是个常数，那么就直接令该函数返回该常数。


## 课堂问答
  > Q: 如何进行模式识别，能对哪些模式的多项式进行处理？
    
    A: 由于时间仓促且任务较重，小组成员决定先在一个子函数中的一个基本块进行操作，
    并且假设优化的多项式仅仅含有加乘减等运算，又假设多项式内仅有一个变量。
    比如多项式`y=x*x*x+3*x*x+3*x+1-x*x*x`该多项式仅含一个变量x，
    如果y的运算被实现在一个函数中，那么就可以被我们的程序识别。
  > Q: 这个优化在实际的工程开发中，能被匹配的模式占了多少比重，建议瞄准工程的热点处进行优化。
    
    A: 很不错的建议，我们会在之后的开发中着重考虑这点
  > Q: 在识别并优化多个变量的多项式时，可能会遇到哪些问题？
    
    A: 编译过程中所需要的空间时间资源指数级上升，可能维护信息的数据结构会比较复杂。

## 编写程序中遇到的问题
1. 配置环境时遇到的问题。一开始使用LLVM 6.0.1，没有办法生成`LLVMAlgebraic.so`动态链接库，必须换用LLVM 7.0.0。其次，在参考[LLVM: Writing an LLVM Pass文档时](http://llvm.org/docs/WritingAnLLVMPass.html)，由于文档有些地方并没有写的十分详细，对不熟悉LLVM的我们添了不少麻烦。
2. API查询困难。由于我们对LLVM工程并不是很熟悉，并且LLVM的文档有些地方不是很清楚，注释过于简洁，很多情况下我们需要直接阅读源代码来确定该使用什么API。
3. 在使用`opt -load ./lib/LLVMAlgebraic.so -algebraic < xxx.bc`时，生成出来的优化过的LLVM IR代码，在其末段出现了很多乱码。后来我们查了很多资料，最后我们发现只需要在`opt -load`后面加入`-S`选项，就可以只生成LLVM IR代码。

# LLVM 调研情况简述

在本次实验进行前，我们进行了不少前期调研。本节简述我们的调研成果。我们的调研均是基于 LLVM 7.0.1 和 Clang 7.0.1。

## LLVM/Clang 编译器架构

Clang 并非一个完整的 C/C++/Objective-C 编译器和连接器整体，而是仅作为 LLVM 的前端。

Clang 所做的工作是：

1. 预处理；
2. 解析和语义分析：将预处理过的源代码处理为抽象语法树；
3. 代码生成：将抽象语法树转换成 LLVM IR 并递送给 LLVM。

值得注意的是，Clang 中几乎没有对 AST 和 IR 进行任何优化。可以使用命令 `clang -Xclang -ast-dump <FILE>` 输出 AST 进行验证。这可能是因为以下因素：

1. LLVM 本身即作为 Clang 的后端设计，所以 LLVM 自带的优化本来就适合 C/C++。
2. 对 AST 的优化过于高级，由于缺失了硬件信息，这可能导致在某些机器上产生负优化。

这里我们知道优化工作的重心应该放在 IR 上，所以我们就没有继续深入调研 Clang 了。

## LLVM Pass

LLVM Pass 是 LLVM 对 IR 处理的主要方式，有三种表现形式：

1. 分析：使用相关算法对 IR 进行扫描，收集相关信息，供之后的 Pass 进行使用。
2. 转换：使用分析结果或其他信息，对 IR 进行变换。
3. 调试：可以在编译时产生一些 LLVM 内部信息，方便开发者调试。

LLVM 提供了丰富的预置分析器和转换器。

LLVM 给编写 Pass 提供了极大方便。它内置了许多 Pass 类型，比较重要的有：

1. `FunctionPass`: 在每个函数上都会执行一次；
2. `BasicBlockPass`: 在每个基本块上都会执行一次；
3. `LoopPass`: 在每个循环上都会执行一次；
4. `ModulePass`: 在每个模块上都会执行一次；
5. `RegionPass`, `MachineFunctionPass` 等其他许多类型。

这些类型均为 `Pass` 的子类。`Pass` 的调度执行受 `PassManager` 管理。`PassManager` 的主要工作是：

1. 负责在合适的时候调用一个 Pass；
2. 负责保证 Pass 执行前其所依赖的 Pass（一般是分析）被执行；
3. 如果一个 Pass 修改了 IR，则无效化之前的分析结果，这保证了编译结果的正确性；
4. 最优化执行顺序，提高编译性能。

编写 Pass 的通常流程如下：

1. 选择一个合适的 Pass 类型；
2. 声明与其他 Pass 的依赖关系；
3. 使用 `RegisterPass` 注册到 LLVM。

可见，编写 LLVM Pass 是十分方便的。

此外，LLVM 不仅仅支持静态链接的 Pass，也支持动态链接的 Pass，即 LLVM 可加载模块（LLVM loadable module）。可加载模块可以看作一类编译器插件。它的缺陷是 Clang 无法将动态加载参数传递给 LLVM，所以无法直接让 Clang 调用此类 Pass，如有此类需求，仍然必须静态链接。

## LLVM Pass 的测试和使用

测试和使用一个 Pass 是相当方便的。LLVM 提供了 `opt` 命令，它可以在一个 LLVM IR 模块上执行指定的 pass 并输出变换后的结果。

例如，

```
## 查看数据依赖分析结果。
## -debug-pass=Structure 可以看到 PassManager 是如何安排 Pass 执行顺序的。
opt -debug-pass=Structure -debug=1 -da -analyze <BC-FILE>

## 在 XXX.so 中定义了 xxx Pass，使用它处理 <BC-FILE>。
opt -load XXX.so -xxx FILE1.bc > FILE2.bc
```

若 Pass 是动态加载的，则调试前必须先在 `llvm::PassManager::run` 处加断点。这是因为 `opt` 刚启动时并未加载插件，会找不到相应的符号；而在 `run` 之前，所有插件和相应的符号均已加载。

## LLVM 优化器案例研究

我们研究了和我们项目目标相近的一些 LLVM 优化器，以学习对 IR 的处理方法和典型的优化技巧。

### 死代码消除

相关代码文件：`/lib/Transforms/Scalar/DCE.cpp`

死代码消除（dead code elimination）是一个相对简单的函数 Pass，它包含两个部分：死指令消除和死代码消除。

死指令消除极为简单，它遍历函数所有指令，并删去所有 trivially dead 的指令。trivially dead 的确切定义可见 `/lib/Transforms/Utils/Local.cpp`，主要思想可以解释为：指令结果没有使用者且没有副作用。

死代码消除则稍微复杂一些。它遍历所有指令，如果一个指令是 trivially dead，就把它的 operand 设置成空，再检查是否有指令变得不再有用，最后把它们全部删除。这里有一个小技巧是把这些新产生的死指令收集到一个 WorkList 中。这一想法看似简单，但在 LLVM Pass 中得到了广泛应用。

### 指令组合

相关代码文件：`/lib/Transforms/InstCombine/*`

指令组合是一个庞大而综合的转换 Pass，它可以识别并处理 LLVM 中的大部分指令（算术、逻辑、控制、存取、资源分配和释放等），并将一些模式优化成更高效的等价形式。LLVM 中的大部分代数优化均发生在此处。但是值得注意的是，它的优化由 WorkList 驱动，整体近似于一种贪心策略，这意味着它的优化较为盲目，并不能有一种全局观。

它的核心部分定义在 `/lib/Transforms/InstCombine/InstructionCombining.cpp` 中。它的基本思想是：
1. 将一个运算转换成某种范式，从而方便之后的检查和判断（如将常数放在算符右边）；
2. 灵活使用交换律、结合律、分配律，找出代数等价形式；
3. 识别一些低效的模式，并利用上述代数运算律转换为等价形式。

虽然它的基本想法并不复杂，但不得不承认将这些简单的想法合并起来，最终效果极好。

### 直线强度削弱

相关代码文件：`/lib/Transforms/Scalar/StraightLineStrengthReduce.cpp`

直线强度削弱类似于循环强度削弱，但它是在顺序执行的语句上执行的。SLSR 的意义在于方便循环展开后的优化。它的基本思想是，若先后有 S1 和 S2 两个表达式，且 S2 可以以较低的代价从 S1 的结果中计算得到，就用这样的计算替代原本的高代价计算。且若有多个可以用来替代计算的结果，则选择支配树上最近的祖先。

## 参考文献

1. [LLVM for Grad Students](https://www.cs.cornell.edu/~asampson/blog/llvm.html)
2. [Writing an LLVM Pass](http://llvm.org/docs/WritingAnLLVMPass.html)
3. [LLVM's Analysis and Transform Passes](https://llvm.org/docs/Passes.html)

除此之外，LLVM 和 Clang 的源代码也是重要的参考资料。
