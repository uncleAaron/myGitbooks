# 第二章 Java并发编程机制的底层实现原理

## 一、volatile

volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的“可见性”。（当一个程序修改一个共享变量时，另外一个线程能读到这个修改的值）

Java规范定义：允许线程访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。

volatile使用恰当的话，它比synchronized的使用和执行成本更低，因为它**不会引起线程上下文切换和调度。**

### volatile原理

```java
/* 对volatile进行写操作 */
instance = new Singleton();		//instance是volatile变量
```
转化成汇编代码：
```assembly
0x01a3de1d: movb $0x0,0x1004800(%esi)
0x01a3de24: lock addl $0x0,(%esp)
```
有volatile变量修饰的共享变量进行写操作的额时候会多出第二行汇编代码。
Lock前缀的指令在多核处理器下会发生两件事：
1. Lock前缀指令会引起处理器缓存回写到内存。
2. 一个处理器的缓存回写到内存会使在其他CPU里的该缓存数据无效（个人理解：可以保证数据一致性）

#### volatile使用优化

例：JDK7的并发包新增的队列集合类LinkedTransferQueue在使用volatile变量时，采用追加字节的方式来优化队列出入队的性能。他将共享变量追加到64字节，原因是现在许多CPU缓存行的大小是64字节宽，不支持部分填充缓存行， 当一个处理器试图修改缓存行的时候会将整个缓存行锁定，若字节数小于64，则头尾节点可能会缓存到同一个缓存行，这样会互相锁定降低修改效率。



## 二、synchronized

利用 synchronized