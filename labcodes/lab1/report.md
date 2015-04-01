# Lab 1 report

## [练习1]

[练习1.1] 操作系统镜像文件 ucore.img 是如何一步一步生成的?(需要比较详细地解释 Makefile 中
每一条相关命令和命令参数的含义,以及说明命令导致的结果)

```
首先我们切到lab1所在目录，然后make clean一下。
接着，我们用$ make V= 来查看一下make的具体过程。
可以看到，它首先调用gcc把一系列.c文件编译成.o文件
然后ld指令会把.o目标文件转换成执行程序，此处是bootblock.out
最后dd指令生成虚拟硬盘ucore.img,再把bootloader和kernel放到ucore.img中
```

[练习1.2] 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么?
```
根据视频的提示，我们查看tools/sign.c文件，找到对应的数组：char buf[512];
由此可知主引导扇区大小是512字节。
同时由如下两行：
	buf[510] = 0x55;
	buf[511] = 0xAA;
知道扇区最后两个字节必须是0x55和0xAA
```

## [练习2]

[练习2.1] 从 CPU 加电后执行的第一条指令开始,单步跟踪 BIOS 的执行。
```
使用si指令
```

[练习2.2] 在初始化位置0x7c00 设置实地址断点,测试断点正常。
```
在 tools/gdbinit文件中，添加b *0x7c00,则在0x7c00处设置了一个断点。
输入make debug进入gdb调试，输入x/8i $pc可以发现结果如下：
   0x7c00:      cli    
   0x7c01:      cld    
   0x7c02:      xor    %ax,%ax
   0x7c06:      mov    %ax,%es
   0x7c08:      mov    %ax,%ss
   0x7c0a:      in     $0x64,%al
   0x7c0c:      test   $0x2,%al
 因此确实是停在0x7c00处。
 并且cli和cld也与bootasm.S中的起始位置一致
 ```

[练习2.3] 从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。
```
使用desassemble 0x7c00, 0x7d00获得gdb的汇编信息如下（部分）：
   0x00007c16:  test   $0x2,%al
   0x00007c18:  jne    0x7c14
   0x00007c1a:  mov    $0xdf,%al
   0x00007c1c:  out    %al,$0x60

   0x00007c48:  add    %al,(%bx,%si)
   0x00007c4a:  call   0x7ccf
   0x00007c4d:  add    %al,(%bx,%si)
   0x00007c4f:  jmp    0x7c4f
   ......
   与bootasm.S对照后确认一致
```

[练习2.4] 自己找一个bootloader或内核中的代码位置，设置断点并进行测试。
```
在0x7c00停下来之后，输入b 56，然后continue，则停止在如下一行：
Breakpoint 3, grade_backtrace0 (arg0=0, arg1=1048576, arg2=-65536) at kern/init/init.c:58
```

## [练习3]
分析bootloader 进入保护模式的过程。
```
在0x7c00地址处开始，首先清理环境;
然后开启A20：通过将键盘A20线至于高电位，使32条地址线都可用;
接着载入gdt表： lgdt gdtdesc  ;
最后通过将CR0寄存器的PE位置1便开启保护模式。(在bootasm.M第50到第52行，代码如下：)
    movl %cr0, %eax
    orl $CR0_PE_ON, %eax
    movl %eax, %cr0
  其中CR0_PE_ON就表示CR0的PE位的使能
```

## [练习4]
分析bootloader加载ELF格式的OS的过程。

```
1、readsect通过inb和outb来读取磁盘扇区
2、bootmain函数里读取ELFHDR，通过其成员变量e_magic来判断是否合法。当判断为合法ELF格式时，从ELFHDR的描述来确定从什么地址开始读进内存(ELFHDR+ELFHDR->e_phoff)，并使用readseg函数将整个ELFHDR依次读进内存。最后把控制权交给ucore(跳转至ELFHDR->e_entry)
```

## [练习5] 
实现函数调用堆栈跟踪函数

```
ss:ebp表示上一层ebp
ss:(eip+4)表示上一层eip
根据注释依次向上递归返回，直至ebp == 0或者达到20层
```

## [练习6]
完善中断初始化和处理

[练习6.1] 中断向量表中一个表项占多少字节？其中哪几位代表中断处理代码的入口？

中断向量表一个表项占用8字节，其中2-3字节是段选择子，0-1字节和6-7字节拼成offset，两者联合便是中断处理程序的入口地址。

[练习6.2]
 见程序
[练习6.3]
 见程序