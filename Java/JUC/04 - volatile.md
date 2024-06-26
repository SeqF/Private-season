### 简介
volatile 关键字用于将变量标记为“存储在主内存中”。这意味着每次读取 volatile 变量都会从计算机的主内存中读取，而不是从 CPU 寄存器中读取，并且每次对volatile 变量的写入都会写入主内存，而不仅仅是 CPU 寄存器

### 可见性的保证
volatile 关键字旨在解决变量可见性问题，通过将 counter 变量声明为 volatile ，所以 counter 变量的写入都会立即写回到主内存。此外，counter 变量的读取都讲直接从主内存读取

代码如下：
```java
public class SharedObject {
    public volatile int counter = 0;
}
```
将变量声明为 volatile 可以保证其他线程该变量写入的可见性
#### 全可见性的保证
实际上，volatile 可见性的保证超出了 volatile 变量本身：
- 如果线程 A 写入了一个 volatile 变量，并且线程 B 随后读取同一个 votlatile 变量，则在写入 volatile 变量之前线程 A 可见的所有变量对线程 B 也可见
- 如果线程 A 读取了一个 volatile 变量，那么所有线程 A 在读取 volatile 变量时可见的所有变量也将从主内存中重新读取

### 有序性的保证
出于性能原因，JVM 和 CPU 可以对程序中的指令重新排序，只要指令的语义保持不变

volatile 可以保持指令的有序，volatile 的 Happen-Before 原则：
- 对于其他变量的读/写 保证在 volatile 变量写入之前
- 对于其他变量的读/写 保证在 volatile 变量读取之后
因为 volatile 变量写入的时候会把缓存里的共享变量全都写入主内存中，如果在 volatile 变量写入之后写入的话，那么其他变量无法及时写入主内存中，造成对其他变量还是不可见

volatile 变量读取的时候，会把缓存清空，再从主内存读取，如果在 volatile 变量读取之前读取，那么肯能造成其他变量读取的是旧值（因为没有及时写入主内存）

### 不保证原子性
如果写入变量的新值不依赖于其先前的值，多个线程甚至可以写入共享的 volatile 变量，并且将正确的值存储在主内存中。意思就是，如果线程向 volatile 变量写入时，不需要读取其旧值

一旦线程线程需要先读取 volatile 变量的值，在进行更新操作，那么 volatile 变量就不保证正确的可见性，读取 volatile 变量和 写入新值 之间的时间间隔会产生竞争条件，多线程进行操作时会发生覆盖的现象（因为对于 volatile 变量，各个线程都是从主内存进行读取）
如果线程 1 将值为 0 的共享计数器变量读到 CPU 寄存器，然后加 1，并且还没将值写回主内存。然后，线程 2 从主内存中读取的值还是为 0 ，加 1 之后仍然没写回到主内存，如下图所示：
![[Pasted image 20231223192701.png]]
线程 1 和 线程 2 现在实际上不同步。共享计数器变量的实际值应该是 2 ，但是每个线程在其 CPU 寄存器红的变量值都是 1， 而在主内存中的值仍为 0