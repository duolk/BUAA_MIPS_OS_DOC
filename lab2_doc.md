##lab2实验报告


> ####Thinking 3.1
我们注意到我们把宏函数的函数体写成了 do { // ... } while(0)的形式，而不是仅仅写成形如 {// ... } 的语句块，这样的写法好处是什么？

* 首先，最大的好处就是，当我们在使用的这个宏的时候，必须在宏后面直接添加一个‘；’，  
这样，我们可以不必考虑宏与函数的差别，无差别的使用。这是区别于{}形式的最大好处。
* 其次，和{}一样，都具有括住宏定义的用途，防止二义性。
* 消除了goto语句的使用，可用break代替

> ####Thinking 3.2
了解了二级页表页目录自映射的原理之后，我们知道， Win2k 内核的
虚存管理也是采用了二级页表的形式，其页表所占的 4M 空间对应的虚存起始地址
为 0xC0000000，那么，它的页目录的起始地址是多少呢？

* 起始地址是0xC0000000，该地址对应的页表项为第C0000000 >> 12，则页目录的起始地址是  
0xC0000000+（0xC0000000>>12）*4 = 0xC0300000

>####Thinking 3.3
思考一下 tlb_out 汇编函数，结合代码阐述一下跳转到 NOFOUND
的流程？

> LEAF(tlb_out)  
    6 nop  
    7 mfc0 k1,CP0_ENTRYHI  
    8 mtc0 a0,CP0_ENTRYHI  
    9 nop  
    10 tlbp  
    11 nop    
    12 nop    
    13 nop    
    14 nop  
    15 mfc0 k0,CP0_INDEX  
    16 bltz k0,NOFOUND  
    17 nop  
    18 mtc0 zero,CP0_ENTRYHI  
    19 mtc0 zero,CP0_ENTRYLO0  
    20 nop  
    21 tlbwi  
    22 NOFOUND:  
    23  
    24 mtc0 k1,CP0_ENTRYHI  
    25  
    26 j ra  
    27 nop  
    28 END(tlb_out) 
    
* 该汇编函数跳转到NOFOUND的流程：  
       tlbp指令检测到no TLB entry matches,便将CP0_INDEX寄存器最高位变成1，  
       然后后边如果检测到CP0_INDEX为负的（首位为1），则表示未找到，跳转到NOFOUND