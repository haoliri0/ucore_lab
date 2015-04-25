练习一：
> 区分page_fault的触发类型，页表項中的PTE_W，PTE_U，PTE_P，可以触发不同类型的page_fault，如修改只读页，访问权限不够等  
  
> 其会触发page_fault，但因为实在处理page_fault中出现的同一种类型的异常，所以会进入double_fault处理例程

练习二：
> FIFO只需要一个链表即可实现，而extended clock需要涉及到访问位和修改位的值。不过其本质还是一个链表结构，只需在链表項访问时访问PTE_A与PTE_D这两个标记位，当不能替换时将该项移到链表的最后即可  
需要被换出的页的特征是：链表中第一个PTE_A与PTE_D标记都为0  
ucore中只需直接访问PTE_A，PTE_D这两个标记为即可  
与FIFO相同，在出现缺页时实现换入换出

