###Lab4实验报告

> Thinking 3.1
> 为什么我们在构造空闲进程链表时使用了逆序插入的方式？

答：因为我们的插入操作是插入到空闲进程链表的头部，而取出一个进程  
也是在头部操作，这样，如果我们逆序插入的话，在env_alloc的时候  
就会正序地取出env[0],env[1]……方便简单。

> Thinking 3.2
> 思考 env_ setup_ vm 函数：

>- 第三点注释中的问题: 为什么我们要执行pgdir[i]=boot\_pgdir[i]这个  
赋值操作？换种说法，我们为什么要使用boot\_pgdir作为一部分模板？ (提  
示:mips 虚拟空间布局)  
- UTOP 和 ULIM 的含义分别是什么，在 UTOP 到 ULIM 的区域与其他用户区  
相比有什么最大的区别？  
- (选做) 我们为什么要让pgdir[PDX(UVPT)]=env_cr3?(提示: 结合系统自映射  
机制)

答：  
这部分我觉得只要解释的通了，前两个问题就一并解决了。  
首先看UTOP和ULIM  
首先根据指导书上说的，2Gkuseg是用户区，2G剩余的kuseg0-2就是内核区了，  
而且这个ULIM就是内核区和用户区的分界线了，很明显，ULIM往上就是内核区了  
所以把ULIM往上所对应的页目录项拷贝过来就毫无疑问（前提是因为每个进程有  
“成为临时班长”的资格，也即拥有成为临时内核的潜质，这个潜质是什么呢？  
这就在每个进程中ULIM往上的区域中体现出来了，有了内核的内容，就有了潜质，  
但潜质不一定能激发出来，只有当他通过某种方式，比如系统调用，陷入内核态的  
时候，潜质就激发出来了。）  
那么现在问题来了，UTOP-ULIM这一区域到底是个什么玩意儿呢？看到这个函数中  
把boot\_pgdir从UTOP往上的区域完全拷贝给进程的pgdir了，也即进程在UTOP往  
上拥有了和内核进程（到底有没有绝对意义上的内核进程，这里我搞不清楚，既然  
是拷贝，那么源文件所在的地方是不是内核进程？）一样的地址布局，UTOP这个东  
西，我通过查找发现它是用户有写权限的地址上限。也就是说UTOP-ULIM这一用户区  
和内核区不同点在于可读！，而且，boot\_pdgir是存在ULIM往上的，pgdir是存在  
UTOP-ULIM中的，通过拷贝，你可以发现其实可以通过访问pgdir处的用户页表来得  
到物理地址（但又有一个地方有些疑惑，拷贝的页表不是只与UTOP向上的地址有映射  
吗？那么用户想要访问用户区可写的地址时怎么办呢？这个我也比较迷糊，但处理的  
方法有两个？一是：可写区还有一个用户页表，UTOP-ULIM的只是为了查询方便？二是:  
进入内核态之后再去修改这个UTOP-ULIM？没搞懂）  
第三问我觉得是把pgdir的物理地址给映射到UVPT这一块，是想把UVPT这一地址强行做成  
页目录入口吗？如果这样的话，UVPT-ULIM这一段是做什么的呢……蜜汁又蜜。。。

>Thinking 3.3
> 思考 user_data 这个参数的作用。没有这个参数可不可以？为什么？（如果你能说  
> 明哪些应用场景中可能会应用这种设计就更好了。可以举一个实际的库中的例子）

答：user_data这个参数我一开始是觉得乾神强行秀了一波。。。它在传入的函数里边为  
void* 在load\_elf中也未void *，仔细观察kernel\_elfloader.c，发现里边完全没有
env这个结构出现，说明这个elf\_loader可以具有普遍性，是不是可以认为如果我要修改  
只需要在env.c文件中修改，不用麻烦的跑到kernel...中去修改了（感觉不仅仅是这么简单  
但我想不出来了>_<）



>Thinking 3.4
思考上面这一段话，并根据自己在 lab2 中的理解，回答：
- 我们这里出现的” 指令位置” 的概念，你认为该概念是针对虚拟空间，还是物
理内存所定义的呢？
- 你觉得entry\_point其值对于每个进程是否一样？该如何理解这种统一或不
同？
- 从布局图中找到你认为最有可能是entry\_point的值。

答：我觉得是虚拟地址，因为首先程序运行的时候是连续运行（一条接着一条）  
从计组课上也知道地址是连续的，而一般情况下，程序的主代码部分只在虚拟  
空间上是连续的，如果pc存储了物理地址，就无法通过pc+4这一操作执行下一  
指令，虚拟地址很容易通过页表就转化为物理地址，而物理地址转为虚拟地址  
我觉得不是那么容易，也不符合页表设计的初衷。  
应该是统一的，不统一的话这对进程的运行就带来了麻烦，我觉得计算机的设计  
肯定是尽可能的简单。
UTEXT吧，不是太明白