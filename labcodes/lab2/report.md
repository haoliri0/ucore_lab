LAB2

# [练习一]
```
在default_init_memmap函数中添加对property设置的判断
	p->property = 0;
        if (p == base)
        	p->property = n;
并将每一个page添加到free_list中list_add_before(&free_list, &(p->page_link));

default_alloc_pages函数中当找到可分配的page时，对n个page分别clear property，并将其从free_list中删除
当page的property>n时，第n个page变成了新的表头，所以其property要减n;

default_free_pages函数只需将list_add改成list_add_before即可

```


# [练习二]
```
找到页目录表对应項pgdir + PDX(la) (基址+index)
若不存在，则新建一个Page
得到线性地址 : uintptr_t pa = page2pa(page)
并将目录的表項更新：*pdep = pa | PTE_U | PTE_W | PTE_P;(地址+标记位)
返回二级页表对应表項：ptep + PTX(la)

页目录项和页表项的组成部分如下：
#define PTE_P           0x001                   // 是否使用中，已用于页机制中。
#define PTE_W           0x002                   // 是否可写，已用于页机制中。
#define PTE_U           0x004                   // 用户态使用，用于用户态管理。
#define PTE_PWT         0x008                   // 是否写直达，用于cache的写回。
#define PTE_PCD         0x010                   // 禁用cache，用于cache系统。
#define PTE_A           0x020                   // 是否被访问过
#define PTE_D           0x040                   // 脏位，被改写，用于write-back写回cache。
#define PTE_PS          0x080                   // 页大小
#define PTE_MBZ         0x180                   // 保留位，全部置0
#define PTE_AVAIL       0xE00                   // 软件是否可以使用

发生page fault时，寄存器cr0-cr3保存异常的信息和当前的程序运行状态，然后通过idt找到对应的中断处理例程的地址并跳转到那里去执行处理这个异常的程序。
```


# [练习三]
```
页表项存在时，得到对应页，将该页的reference - 1，当reference为0时释放该Page
清除二级页表只需将ptep指针清空

全局变量pages在page_init函数中可以看出即为页目录以及页表中对应的页

如果希望虚拟地址与物理地址相等，则需要将ucore的起始地址和内核虚地址设为相同的值

```
