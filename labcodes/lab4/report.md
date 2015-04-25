#练习一：
所以进程的初始state为PROC_UNINIT
pid，run，kstack，need_resched，mm，context，flags及name都在创建后被重新定义，所以可以将他们都设置成0
cr3：base addr of Page Directroy Table(PDT)，ucore中定义了boot_cr3，所以直接赋值

context为上下文，在proc.h中可以看到其中存了8个寄存器，用于进程切换时保存现场和恢复现场
struct trapframe *tf是一个中断结构，记录了内核的代码段与数据段等参数


#练习二：
按照提示首先初始化子进程，将其parent设为当前进程，然后设置堆栈，拷贝内存，获取pid，将其添加到hash表和进程链表中，最后将其唤醒
kstack的初始化位于setup_kstack()函数: 调用alloc_page分配页面，然后调用page2kva得到页对应的内核虚拟地址，即为堆栈起始地址
trapframe的初始化位于copy_thread函数: 为trapframe分配地址空间，由这一句"proc->tf = (struct trapframe * )(proc->kstack+KSTACKSIZE)-1)"可知其终止地址为kstack的栈顶，利用传入的参数tf对其进行初始，然后对寄存器进行相应设置
context的初始化位于copy_thread函数: 为context的eip esp赋初值
然后便完成了资源的分配。

做到了pid唯一：
在get_pid函数中从last_pid = 1开始，遍历proc_list，若已有进程的pid与last_pid相同则last_pid+1，若超过MAX_PID，则
为1，重新遍历proc_list，以此保证pid不重复，所以做到了pid唯一


#练习三：
创建运行了两个内核线程，分别为：initproc和idleproc
local_intr_save(intr_flag);....local_intr_restore(intr_flag); 分别用于关闭中断和开启中断，以此保证进程切换的正确进行

