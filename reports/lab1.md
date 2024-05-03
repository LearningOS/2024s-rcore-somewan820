# ch3-实验报告
## 编程作业
ch3的编程任务是实现一个 **sys_task_info** 的系统调用，它的主要功能是：
- 任务控制块相关信息（任务状态）
- 任务使用的系统调用次数
- 系统调用时刻距离任务第一次被调度时刻的时长
即struct TaskInfo中给我们写好的status、syscall_times和time
### status
因为是查询当前任务，那任务状态肯定是Running
### syscall_times
这里syscall_times采用桶计数的方法，定义一个[u32; MAX_SYSCALL_NUM]的数组，以SYSCALL_ID 为索引去记录每个syscall的调用次数。首先在 **os/src/task/task.rs** 的 **pub struct TaskControlBlock** 中定义 **pub sys_call_times: [u32; MAX_SYSCALL_NUM]** ,再在 **os/src/task/mod.rs** 的 **TaskManager** 中定义函数 **set_syscall_records()** ，在外部对其进行封装 **_set_syscall_records()** 。最后在 **os/src/syscall/mod.rs** 的 **syscall** 函数中调用 **_set_syscall_records()** ，到达记录每次系统调用的作用
### time
使用 **os/src/timer.rs** 中给好的 **get_time_ms()** 即可
## 简答作业
### 一
Rustsbi 版本为: 0.2.0-alpha.2
有报错
```
[kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003c4, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.
```
ch2b_bad_address.rs 由于除0错误触发异常退出
ch2b_bad_instructions.rs 在用户态非法使用指令sretch2b_bad_register.rs 在用户态非法使用指令csrr
### 二. 
#### Q1
a0代表系统调用的第一个参数
- 从系统调用和异常返回时, 恢复要返回的用户态的上下文信息
- 任务切换时, 恢复要切换的任务的上下文信息
#### Q2
这几行汇编代码特殊处理了以下三个寄存器：t0、t1 和 t2：
- ld t0, 32*8(sp): 这行代码将从内存中加载位于栈指针 sp 偏移 32*8 字节处的值到 t0 寄存器中。这里 t0 寄存器用于保存之前异常处理程序中的 sstatus 寄存器的值。
- ld t1, 33*8(sp): 这行代码将从内存中加载位于栈指针 sp 偏移 33*8 字节处的值到 t1 寄存器中。这里 t1 寄存器用于保存之前异常处理程序中的 sepc 寄存器的值。
- ld t2, 2*8(sp): 这行代码将从内存中加载位于栈指针 sp 偏移 2*8 字节处的值到 t2 寄存器中。这里 t2 寄存器用于保存之前异常处理程序中的 sscratch 寄存器的值。

意义：
- sstatus（保存在 t0 寄存器中）：sstatus 是一个特权状态寄存器，用于控制和监视 CPU 的运行状态。在进入用户态前，需要将先前保存的 sstatus 寄存器的值恢复回来。这是为了确保用户态的代码和数据以正确的权限运行，并且访问受限资源时受到适当的限制。
- sepc（保存在 t1 寄存器中）：sepc 是一个特权程序计数器，用于保存下一条指令的地址。在进入用户态前，需要将先前保存的 sepc 寄存器的值恢复回来。这是为了确保从正确的位置继续执行用户态代码，以防止异常处理程序返回到错误的位置。
- sscratch（保存在 t2 寄存器中）：sscratch 是一个特权寄存器，用于保存临时数据。在进入用户态前，需要将先前保存的 sscratch 寄存器的值恢复回来。这是为了确保在用户态执行期间可以使用先前保存的 sscratch 数据。
#### Q3
是由于循环体内的 LOAD_GP 宏的迭代逻辑，以及循环之前通过 .set 指令设置的初始值。
#### Q4
sp 和 sscratch 的值发生了交换。
#### Q5
csrrw sp, sscratch, sp
使用 csrrw 指令将 sp 寄存器与 sscratch 寄存器进行交换，以切换栈指针的目标。这会将栈指针切换到用户栈（保存在 sscratch 中），使得之后的指令在用户栈上执行。
#### Q6
sp 和 sscratch 的值发生了交换。
#### Q7
ecall
## 荣誉准则
1. 在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：
    无
2. 此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：
    无
3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。
4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。