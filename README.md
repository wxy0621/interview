## 面经 2022 for Linux 后端 服务端 Cpp 云原生

### 一、操作系统



#### 1. 进程 & 线程

```
区别：
1. 进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动，线程是进程的一个实体；
2. 进程是系统进行资源分配和调度的一个独立单位。线程是进程的一个实体，是CPU调度和分派的基本单位；
3. 线程自己基本上不拥有系统资源，只拥有一点在运行中必不可少的资源（如程序计数器,一组寄存器和栈），但它可与同属一个进程的其他的线程共享进程所拥有的全部资源；
4. 线程的切换开销小，进程的切换开销大；
5. 线程崩溃会使得整个进程都崩溃，进程崩溃则不会影响到别的进程；
6. ...

何时使用线程，何时使用进程：
1. 需要频繁创建销毁的优先使用线程，因为创建和销毁一个进程代价是很大的。 
2. 线程的切换速度快，所以在需要大量计算，切换频繁时用线程，还有耗时的操作使用线程可提高应用程序的响应。 
3. 多进程可以使用在多机分布式系统，需要扩展到其他机器上，使用多进程，多线程适用于多核处理机。 
4、需要更稳定安全时，适合选择进程；需要速度时，选择线程更好。

线程是否创建的越多越好？
不是，线程创建越多，对共享资源同步要求多，设计同步容易出错。另外多线程并发容易导致资源分配问题。而且创建线程本身也会消耗大量的资源，如果一有任务就创建线程，那么容易导致系统的负载过大。一般解决方案是线程池。 
```



#### 2. 进程间通信方式（IPC）

```
1. 管道 pipe：半双工，只能在具有亲缘关系的进程间使用（父子）；
2. 先入先出队列 FIFO：全双工，允许不相关的进程之间相互通信；
3. 消息队列：消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。
4. 共享存储：共享内存就是映射一段能被其他进程所访问的内存，是最快的 IPC 方式，和信号量配合使用；
5. 信号量：一个计数器，可以用来控制多个进程对共享资源的访问，常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源，因而主要作为进程间以及同一进程内不同线程之间的同步手段；
6. 套接字：IP + 端口号，分为三种： 流套接字（SOCK_STREAM），数据报套接字（SOCK_DGRAM），原始套接字（SOCK_RAW），例如 Ping 或者 ICMP 等非传输层数据包或者操作系统无法处理的数据包。
```



#### 3. 僵尸进程和孤儿进程

```
1. 孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程（进程号为1）所收养，并由init进程对它们完成状态收集工作。孤儿进程并不会有什么危害。

2. 僵尸进程：一个进程使用 fork() 创建子进程，如果子进程退出，而父进程并没有调用wait() 或 waitpid() 获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵尸进程。但僵尸进程有挺大的危害，其状态为Z
可以通过kill -SIGKILL 父进程ID　来解决僵尸进程
```



#### 4. wait()或者waitpid()

```
wait() 函数一般用在父进程中等待回收子进程的资源，而防止僵尸进程的产生。
```



#### 5. 乐观锁 & 悲观锁 & 自旋锁 & 读写锁 & 互斥锁

```
乐观锁和悲观锁是一种概念
1）乐观锁：每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据。
2）悲观锁：每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会 block 直到它拿到锁。采用“先加锁再访问”的保守策略，会降低效率、增加数据库额外开销、增加死锁几率、降低并行性。之所以叫做悲观锁，是因为这是一种对数据的修改持有悲观态度的并发控制方式。总是假设最坏的情况，每次读取数据的时候都默认其他线程会更改数据，因此需要进行加锁操作。
乐观锁比较适用于读多写少的情况（多读场景），悲观锁比较适用于写多读少的情况（多写场景）

3）自旋锁：当一个线程在获取锁的时候，如果锁已经被其他线程获取了，那么该线程将循环等待，不断地判断锁是否可以被成功获取，直到成功获取才能退出循环。
优点：自旋锁无上下文切换，一直处于用户态，线程一直是 active 的，不会进入阻塞，减少上下文切换，因此执行速度快。
缺点：如果有个线程持有锁的时间过长，就会导致其他等待的线程一直处于循环等待，消耗 CPU，使用不当会使得 CPU 使用率高。

4）读写锁：允许多个线程同时读，但只允许一个线程写，在线程获取到写锁的时候，其他写操作和读都会处于阻塞状态，读锁和写锁是互斥的。

5）互斥锁
```



#### 6. 虚拟内存

```
一种存储模式，通过该模式让我们感觉内存本身能够处理远比内存大的多的数据或者文件。基本原理即按需加载，缺少哪页就加载哪一页（发送缺页信号，读取新数据进内存，释放内存上处理过的旧数据）。

优点：1）可以使用有限的内存资源，处理比实际内存更大的文件或者数据；
     2）更加高效的内存利用；
     3）在有限的内存资源内，让系统运行更多的程序实例，因为每个程序都是按需取。

缺点：1）如果内存严重不足，而处理超级大的文件时，会频繁引起内存和磁盘进行swap，从而降低系统性能；
     2）在多个应用程序之间切换会花费更多的时间；
     3）虚拟内存本质上是充分了磁盘空间，但同时变相的提供用户使用的实际磁盘空间也会变小。
     
内存页替换算法：1）FIFO 先入先出；
						 2）LFU 最不常用；
						 3）LRU 最近最少使用；
```



#### 7. 进程间同步

```
进程同步方式 
1、临界区（Critical Section）:通过对多线程的串行化来访问公共资源或一段代码，速度快，适合控制数据访问。 
优点：保证在某一时刻只有一个线程能访问数据的简便办法 
缺点：虽然临界区同步速度很快，但却只能用来同步本进程内的线程，而不可用来同步多个进程中的线程。 
2、互斥量（Mutex）:为协调共同对一个共享资源的单独访问而设计的。 
互斥量跟临界区很相似，比临界区复杂，互斥对象只有一个，只有拥有互斥对象的线程才具有访问资源的权限。 
优点：使用互斥不仅仅能够在同一应用程序不同线程中实现资源的安全共享，而且可以在不同应用程序的线程之间实现对资源的安全共享。 
缺点：①互斥量是可以命名的，也就是说它可以跨越进程使用，所以创建互斥量需要的资源更多，所以如果只为了在进程内部是用的话使用临界区会带来速度上的优势并能够减少资源占用量。因为互斥量是跨进程的互斥量一旦被创建，就可以通过名字打开它。 
②通过互斥量可以指定资源被独占的方式使用，但如果有下面一种情况通过互斥量就无法处理，比如现在一位用户购买了一份三个并发访问许可的数据库系统，可以根据用户购买的访问许可数量来决定有多少个线程/进程能同时进行数据库操作，这时候如果利用互斥量就没有办法完成这个要求，信号量对象可以说是一种资源计数器。 
3、信号量（Semaphore）:为控制一个具有有限数量用户资源而设计。它允许多个线程在同一时刻访问同一资源，但是需要限制在同一时刻访问此资源的最大线程数目。互斥量是信号量的一种特殊情况，当信号量的最大资源数=1就是互斥量了。 
优点：适用于对Socket（套接字）程序中线程的同步。（例如，网络上的HTTP服务器要对同一时间内访问同一页面的用户数加以限制，只有不大于设定的最大用户数目的线程能够进行访问，而其他的访问企图则被挂起，只有在有用户退出对此页面的访问后才有可能进入。） 
缺点：①信号量机制必须有公共内存，不能用于分布式操作系统，这是它最大的弱点； 
②信号量机制功能强大，但使用时对信号量的操作分散，而且难以控制，读写和维护都很困难，加重了程序员的编码负担； 
③核心操作P-V分散在各用户程序的代码中，不易控制和管理，一旦错误，后果严重，且不易发现和纠正。 
4、事件（Event）: 用来通知线程有一些事件已发生，从而启动后继任务的开始。 
优点：事件对象通过通知操作的方式来保持线程的同步，并且可以实现不同进程中的线程同步操作。 
```



#### 8. 信号量 & PV操作 & 锁

```
S：信号量
P 操作表示通过：1）S减1；
						  2）若S减1后仍大于或等于0，则进程继续执行；
						  3）若S减1后小于0，则该进程被阻塞后放入等待该信号量的等待队列中，然后转进程调度；

V 操作表示释放：1）S加1；
						  2）若相加后结果大于等于0，则进程继续执行；
						  3）若相加后结果小于0，则从该信号的等待队列中释放一个等待进程，然后再返回原进程继续执行或转进程调度；
						  
						  
int sem_init(sem_t *sem, int pshared, unsigned int value);其中sem是要初始化的信号量，pshared表示此信号量是在进程间共享还是线程间共享，value是信号量的初始值。
int sem_destroy(sem_t *sem);其中sem是要销毁的信号量。只有用sem_init初始化的信号量才能用sem_destroy销毁。
int sem_wait(sem_t *sem);等待信号量，如果信号量的值大于0,将信号量的值减1,立即返回。如果信号量的值为0,则线程阻塞。相当于P操作。成功返回0,失败返回-1。
int sem_post(sem_t *sem);释放信号量，让信号量的值加1。相当于V操作。
```



#### 9. 死锁 & 如何解决死锁

```
当线程A持有独占锁a，并尝试去获取独占锁b的同时，线程B持有独占锁b，并尝试获取独占锁a的情况下，就会发生AB两个线程由于互相持有对方需要的锁，而发生的阻塞现象，我们称为死锁。造成死锁的4个条件：1）互斥条件：一个资源每次只能被一个线程使用；2）请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放；3）不剥夺条件：线程已获得的资源，在未使用完之前，不能强行剥夺；4）循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系。破坏造成死锁的任意1个达成条件即可。
```



#### 10. 同步异步、原子操作、互斥等

```
原子操作：一个函数（原语）或动作的指令序列不可分割，要么作为一个整体执行（不可中断），要么都不执行。互斥：当一个进程正在临界区中访问临界资源时，其他进程不能进入临界区。临界资源：一次仅允许一个进程独自占有使用的不可剥夺资源。临界区：进程访问临界资源的那一段代码。饥饿：一个就绪进程被调度程序长期忽视、不被调度执行。即使系统没有发生死锁，某些进程也可能会长时间等待．当等待时间给进程推进和响应带来明显影响时，称发生了进程饥饿，当饥饿到一定程度的进程所赋予的任务即使完成也不再具有实际意义时称该进程被饿死。
```



#### 11. 进程调度策略

```
1. 先来先服务调度算法（FCFS）：最简单的调度算法，每次调度都是从后备作业队列中选择一个或多个最先进入该队列的作业，将它们调入内存，为它们分配资源、创建进程，然后放入就绪队列；2. 短作业(进程)优先调度算法（SJF）：指对短作业或短进程优先调度的算法，从后备队列中选择一个或若干个估计运行时间最短的作业，将它们调入内存运行；3. 高优先权优先调度算法（FPF）：为了照顾紧迫型作业，从后备队列中选择若干个优先权最高的作业装入内存；4. 高响应比优先调度算法：作业的优先级随着等待时间的增加而以速率 a 提高，长作业在等待一定的时间后，必然有机会分配到处理机；5. 时间片轮转法：把所有的就绪进程按先来先服务的原则排成一个队列，队列中的每个进程分配一个时间片来执行。当执行的时间片用完时，由一个计时器发出时钟中断请求，调度程序便据此信号来停止该进程的执行，并将它送往就绪队列的末尾。可保证就绪队列中的所有进程在一给定的时间内均能获得一时间片的处理机执行时间；6. 多级反馈队列调度算法（目前公认的一种较好的进程调度算法）：一种综合调度策略1）应设置多个就绪队列，并为各个队列赋予不同的优先级。第一个队列的优先级最高，第二个队列次之，其余各队列的优先权逐个降低。该算法赋予各个队列中进程执行时间片的大小也各不相同，在优先权愈高的队列中，为每个进程所规定的执行时间片就愈小。例如，第二个队列的时间片要比第一个队列的时间片长一倍，……，第i+1个队列的时间片要比第i个队列的时间片长一倍。2）当一个新进程进入内存后，首先将它放入第一队列的末尾，按FCFS原则排队等待调度。当轮到该进程执行时，如它能在该时间片内完成，便可准备撤离系统；如果它在一个时间片结束时尚未完成，调度程序便将该进程转入第二队列的末尾，再同样地按FCFS原则等待调度执行；如果它在第二队列中运行一个时间片后仍未完成，再依次将它放入第三队列，……，如此下去，当一个长作业(进程)从第一队列依次降到第n队列后，在第n 队列便采取按时间片轮转的方式运行；3）仅当第一队列空闲时，调度程序才调度第二队列中的进程运行；仅当第1～(i-1)队列均空时，才会调度第i队列中的进程运行。如果处理机正在第i队列中为某进程服务时，又有新进程进入优先权较高的队列(第1～(i-1)中的任何一个队列)，则此时新进程将抢占正在运行进程的处理机，即由调度程序把正在运行的进程放回到第i队列的末尾，把处理机分配给新到的高优先权进程。
```



#### 12. 信号量

```
信号量：用于进程间传递信号的一个整数。在信号两上只可以进行三种操作，即初始化，递增，递减。这三种操作分别都是原子操作。递减作用于阻塞一个进程，递增作用于解除一个进程的阻塞。信号量也称为计数信号量或一般信号量。
```



#### 13. 操作系统的抽象

```
（1）文件作为所有的 I/O设备 的抽象；（2) 虚拟内存作为 I/O设备 + 主内存 的抽象；（3）进程作为 cpu处理器 + 主内存 + I/O设备 的抽象。
```



#### 14. 系统调用

```
用户处于用户态中，但如果需要调用操作系统中系统态级别的子功能，需要系统调用。用户通过系统调用来使用硬件设备，而不需要关心具体的硬件设备，简化了开发，是用户程序和硬件设备之间的桥梁。系统调用 API 处于用户程序和内核之间。用户程序只需关心系统调用 API，通过这些 API 来开发自己的应用，不用关心 API 的具体实现。内核则只要关心系统调用 API 的实现，而不必管它们是被如何调用的。
```



#### 15. 守护进程 daemon

```
系统在启动时就已经运行的进程，一般生命周期从系统启动到系统停止。
```



#### 16. MMU

```
内存管理单元（Memory Management Unit）有时被称为：分页内存管理单元 PMMU（Paged Memory Management Unit）是一种负责处理 CPU 的内存访问请求的计算机硬件，功能包括：1）虚拟地址到物理地址的转换（虚拟内存管理）；2）内存保护；3）中央处理器高速缓存的控制。MMU 位于处理器内核和总线之间（连接高速缓存和物理存储器）用来将虚拟地址映射到物理地址的数据结构被称为页表，通过数组来实现两个数组的相互关联。每个进程都会需要自己的页表，但是一个进程显然不需要一整个内存空间去存放。因此将页表分为多个级别（一级，二级）。一级页表中的 PTE（分页如果是空的），那么二级页表就不存在。只有一级页表才是总存在主存中的，二级页表只有最经常使用的才会存在于主存中。
```



#### 17. 信号 & 信号量

```
1）信号：是由用户、系统或者进程发送给目标进程的信息，以通知目标进程某个状态的改变或系统异常。2）信号量：信号量是一个特殊的变量，它的本质是计数器，信号量里面记录了临界资源的数目，有多少数目，信号量的值就为多少，进程对其访问都是原子操作（pv操作，p：占用资源，v：释放资源）。它的作用就是，调协进程对共享资源的访问，让一个临界区同一时间只有一个进程在访问它。区别：信号通知进程产生了某个事件，信号量用来同步进程。
```



#### 18. 大小端模式

```
1）大端数据高地址保存在低地址空间，低地址存在高地址空间2）小端数据高地址保存在高地址空间，低地址存在低地址空间
```



#### 19. 32 位机的内存

```
4GB2^32 = 4 * 1000 * 1000 * 1000
```



#### 20. 中断和异常

```
1）中断：外部硬件设备产生，通过中断触发器发送给 CPU，CPU 判断这是来自哪个硬件设备，发送给内核进行中断处理；因为中断是外部设备触发的，因此是异步事件。2）异常：当 CPU 处理程序时，会发送异常，程序不在内存中（缺页异常），除数为0（除0异常）等，CPU 此时也会发送给内核，要求内核处理异常。因为中断是 CPU 触发的，因此是同步事件。中断又可以分为可屏蔽中断和非可屏蔽中断，异常又分为故障、陷阱和异常中止3种。操作系统对中断和异常的处理是相同的，当 CPU 收到中断或者异常的信号时，它会暂停执行当前的程序或任务，通过一定的机制跳转到负责处理这个信号的相关处理程序中，在完成对这个信号的处理后再跳回到刚才被打断的程序或任务中。CPU 利用栈保护被暂停执行的程序的现场，依次压入栈中。
```



#### 21. 为什么进程上下文切换比线程上下文切换代价高

```
进程的切换需要：1）切换页目录以使用新的地址空间；			 			 2）切换内核栈和硬件上下文；线程的切换只需要：切换内核栈和硬件上下文；且进程上下文切换需要保存的内容比线程上下文切换的多，线程上下文是进程上下文的子集
```



#### 22. 为什么协程切换的代价比线程切换低

```
线程切换需要借助内核，每次切换需要一次用户态到内核态的切换，再从内核态切换到用户态。协程的切换只在用户态就可以完成
```





### 二、计算机网络



#### 1. TCP 三次握手 & 四次挥手

```
三次握手的状态：1）客户端：CLOSED、SYN-SENT、ESTABLISHED2）服务器：CLOSED、LISTEN、SYN-RECEIVED、ESTABLISHED第一次握手：SYN=1,Seq=X第二次握手：SYN=1,ACK=1,Seq=Y,Ack=X+1第三次握手：ACK=1,Seq=X+1,Ack=Y+1四次挥手的状态：1）客户端：ESTABLISHED、FIN-WAIT-1、FIN-WAIT-2、TIME-WAIT、CLOSED2）服务端：ESTABLISHED、CLOSE-WAIT、LAST-ACK、CLOSED第一次挥手：FIN=1,Seq=u第二次挥手：ACK=1,Seq=v,Ack=u+1第三次挥手：FIN=1,ACK=1,Seq=w,Ack=u+1第四次挥手：ACK=1,Seq=u+1,Ack=w+1
```



#### 2. 三次握手失败的后果

```
1）第一次握手失败，客户端重发 SYN 请求，若到达到重传次数上限，则建立连接的系统调用返回-1；2）第二次握手失败，客户端未收到 ACK 回复，则和第一次握手失败一样重传；3）第三次握手失败，服务器会一直重传第二次握手的 ACK，若超过上限仍未得到回复，accept() 系统调用返回 -1，建立失败。但客户端以为自己建立成功了，会发送消息，服务器若接收到客户端的消息，则 RST 报文给客户端，消除客户端的单方面连接
```



#### 3. CLOSE-WAIT & TIME-WAIT 的意义

```
1）CLOSE-WAIT 服务器在关闭前仍有消息需要发送，不能立即关闭2）TIME-WAIT 第四次挥手可能会丢失，如果立刻关闭，则服务器的第三次挥手 FIN 若重发的话，客户端已经关闭了，则客户端会回复一个 RST 给服务端，服务端会误认为有错误发生，但实际没有异常发生。
```



#### 4. TIME-WAIT 的 2MSL

```
MSL（Maximum Segment Lifetime)第四次挥手可能会丢失，需要一个 2MSL 的计时器，来确保客户端在 2MSL 内不会再收到来自服务器发来的 FIN 第三次挥手报文了，若收到 FIN 第三次挥手报文，则计时器需要重置。最终目的即确保服务器可以收到第四次挥手的 ACK 确认报文。
```



#### 5. TCP 拥塞控制 & 流量控制

```
区别：拥塞控制是全局的，希望不要有太多的数据注入到网络中，不知道拥塞到底发生在如初；流量控制是端到端通信。1. 拥塞控制：在某段时间，若对网络中某一资源的需求超过了该资源所能提供的可用部分，网络性能就要变坏，这种情况就叫做网络拥塞。TCP 通过维护一个拥塞窗口来进行拥塞控制，只要网络中没有出现拥塞，拥塞窗口的值就可以再增大一些，以便把更多的数据包发送出去。若网络出现拥塞，拥塞窗口即减小。分为4个阶段，慢启动、拥塞避免、快重传和快恢复：1）慢启动阶段：cwnd 呈现指数增长趋势（*=2）2）拥塞避免阶段：cwmd > ssthresh  呈现线性增长趋势（+=1），以防止很快出现拥塞状态；当拥塞真正发生时，将 cwmd 减为 1，ssthresh 减为一半3）快重传阶段：发送方只要一连接收到三个重复确认就应该立即重传对方尚未的报文段，而不必等到重传计时器超时后发送。由3个重复应答判断有包丢失，重新发送丢包的信息。例如发送端发送了1-4报文，接收端收到了1、3、4，则确认的 ACK 收到了4个1，因此需要立即重发2。提高网络吞吐量；4）快速恢复阶段：配合快重传，当收到三个重发 ACK，则 ssthresh 减半；cwmd 设置为 ssthresh 减半后的值，然后执行拥塞避免，cwmd线性增大。2. 流量控制：通过滑动窗口来实现，接收方返回的 ACK 会包含自己接收窗口的大小，以控制发送方发送的数据的大小（发送窗口大小）若接收窗口已满，发送方则不会发送消息，周期性向接收方查询来查看窗口是否变大，窗口非 0 时，继续发送数据。
```



#### 6. SYN 攻击

```
伪造大量不存在的 IP 地址，向 Server 不断地发送 SYN 包，Server 则回复确认包，并等待 Client 确认，由于源地址不存在，因此 Server 需要不断重发直至超时，这些伪造的SYN 包将长时间占用未连接队列，导致正常的 SYN 请求因为队列满而被丢弃。SYN 攻击是一种典型的 DoS/DDoS 攻击。常见的防御 SYN 攻击的方法有如下几种：1）缩短超时（SYN Timeout）时间2）增加最大半连接数3）过滤网关防护4）SYN cookies技术5）检测 IP 号的规律性，删选出虚假的 IP6）通过加密序列号（哈希），检测前后的序列号（哈希值）是否一致，删除不符的连接
```



#### 7. 半连接队列、全连接队列、TCP 连接重传

```
服务器第一次收到客户端的 SYN 之后，就会处于 SYN_RCVD 状态，此时双方还没有完全建立其连接，服务器会把此种状态下请求连接放在一个队列里，我们把这种队列称之为半连接队列。已经完成三次握手，并建立起连接的就会放在全连接队列中。服务器发送完 SYN-ACK 包，如果未收到客户确认包，服务器进行首次重传，等待一段时间仍未收到客户确认包，进行第二次重传。如果重传次数超过系统规定的最大重传次数，系统将该连接信息从半连接队列中删除。每次重传等待的时间不一定相同，一般会是指数增长，例如间隔时间为 1s，2s，4s，8s…
```



#### 8. TCP ISN 序列号

```
ISN（Initial Sequence Number）随时间而变化，ISN 可以看作是一个32比特的计数器，每 4ms 加1。这样选择序号的目的在于防止在网络中被延迟的分组在以后又被传送，而导致某个连接的一方对它做错误的解释。且动态的可以防止来自攻击者的攻击。
```



#### 9. TCP 的粘包和拆包

```
1. 粘包：1）写入数据小于套接字缓冲区大小，默认使用 Nagle 算法，收集多个小分组，一起发送给对端；2）发送数据太快，接收端来不及处理，发生粘包；2. 拆包：1）写入数据大于套接字缓冲区大小；2）进行 MSS（最大报文长度）大小的分段，当 TCP 报文的数据部分大于 MSS 时发生拆包；解决办法：1）使用分隔符（换行符）；2）在头部添加消息长度字段，接收端根据该长度解析消息；3）空位补足，但会浪费网络资源；
```



#### 10. TCP 标志位

```
1）ACK 应答域内有效，只有 0 和 1；2）RST 复位请求，用来复位产生错误的连接，也用来拒绝错误和非法数据包；3）SYN 同步序号，建立连接请求；4）FIN 断开连接请求；5）URG 紧急标志；6）PSH 推标志，尽快处理，接收端不应该做队列处理。
```



#### 11. TCP 和 UDP 区别

```
1. TCP（Transmission Control Protocol，传输控制协议）是面向连接的协议，在通信前需要通过三次握手先建立可靠的连接。在断开连接时需要4次挥手进行断连。UDP 是无连接的协议，通过发送数据报的形式传输数据；2. TCP 对系统资源的要求多，UDP 少。UDP 头部开销小（8字节），TCP（20-60字节）；3. TCP 是流模式，UDP 是数据报模式；4. TCP 保证数据正确性，UDP 可能丢包；5. TCP 保证数据顺序，UDP 不保证；6. TCP 只支持一对一， UDP 支持各种（一对一，一对多，多对一和多对多）。7. TCP 适合可靠传输的应用，例如文件传输。UDP 适合实时应用（IP电话、视频会议、直播等）。
```



#### 12. TCP 如何实现可靠传输，UDP 如何能够改进成可靠传输

```
1）数据分块：分割成 TCP 认为最适合发送的数据块；2）序列号和确认应答：每个发送的报文都进行编号，接收方收到数据后，都会进行确认应答（ACK），包含了确认序列号。接受方还可以根据序列号对数据包进行排序，将有序数据传输给应用层，丢弃重复的数据；3）校验和：TCP 保持它首部和数据部分的校验和，这是一个端到端的校验和，目的是检验传输过程中是否发生差错，若差错则丢弃；4）流量控制：TCP 连接的的双方都有一个固定大小的缓冲区，发送方发送的数据量不能超过接收端缓冲区的大小，若接收方来不及处理发送方的数据，则会提醒发送方降低发送的速率，防止丢包的发生。TCP 通过滑动窗口协议来支持流量控制；5）拥塞控制：当某个节点发生拥塞时，减少数据的发送；6）ARQ 协议：分为三种，停止等待 ARQ，回退 N 帧 ARQ和选择重传 ARQ；7）超时重传：发出一个报文段后，就启动一个定时器，如果超过一定时间没有收到确认，则重发该报文段。
```



#### 13. HTTPS & HTTP 区别

```
1）http 是直接和 tcp 通信，https 通过 http+ssl 加密；2）http 端口号为80，https 端口号为443；3）https 基于传输层、http 基于应用层；4）https 对于搜索引擎更友好，利于 seo，百度、谷歌等搜索引擎优先索引 https 网页ssl加密：发送密文的一方使用对方的公钥进行加密处理“对称的密钥”，然后对方用自己的私钥解密拿到“对称的密钥”，这样可以确保交换的密钥是安全的前提下，使用对称加密方式进行通信。所以，HTTPS 采用对称加密和非对称加密两者并用的混合加密机制。
```



#### 14. HTTP 请求报文和相应报文

```
http请求报文：请求行（请求方法+url）、请求头，请求体http响应报文：状态行（http版本+状态码）、响应头、响应体
```



#### 15. HTTP 1.0 1.1. 2.0 3.0

```
1. 1.0 & 1.1 区别	1）缓存处理，1.1 增加了更多的与缓存有关的策略；	2）节约带宽，1.0 将整个请求的内容发给客户端，1.1 增加了 range 头域，允许只请求部分资源；	3）长连接，1.0 默认短连接，1.1 默认长连接，支持一个 TCP 请求中传送多个 HTTP 请求和响应。1.0 要想维持长连接，需要指定 Connection 的首部字段值为 Keep-Alive；	4）状态码更多了	5）Host 请求头，早期一个 IP 对应一个主机名，后来多了虚拟主机的出现，一个物理机可以有多个虚拟主机，他们共享一个 IP。1.1 增加了 host 请求头，若不声明则返回 404 错误码。	2. 1.X & 2.0 区别	1）1.X 采用文本（字符串）传送，2.0 采用二进制传送，传输时数据封装成帧，帧组成数据流，流ID 具有标识和优先级的功能；	2）IO 多路复用，一个 HTTP 请求实现多个 HTTP 传输，通过 流ID 来识别；	3）头部压缩，双方维护一个头部信息表，每次只传递头部信息的索引，大大减小重传次数和数据量；	4）支持服务器推送，服务器可以在客户端未经许可的情况下给客户端发送推送。	3. 3.0	存在的问题：之前的协议都是基于 TCP，但 TCP 会有拥塞问题（慢启动，拥塞窗口大小设置问题）	3.0 基于 QUIC 协议实现，全称：快速 UDP 网络连接，目的为了解决 TCP 的问题，满足传输层和应用层多连接、低延迟的需求，融合了 TCP、TLS、HTTP 2.0。
```



#### 16. Get & Post 区别

```
GET 和 POST 是 HTTP 请求的两种基本方法1）GET 把参数包含在 URL 中（可能会不安全），POST 通过 request body 传递参数；2）GET 在浏览器回退时是无害的，POST 会再次提交请求；3）GET 产生的 URL 地址可以被 Bookmark，而 POST 不可以；4）GET 请求会被浏览器主动 cache，而 POST 不会，除非手动设置；5）GET 请求在 URL 中传送的参数是有长度限制的，而 POST 没有；6）对参数的数据类型，GET 只接受 ASCII 字符，而 POST 没有限制；7）GET 请求只能进行 url 编码，而 POST 支持多种编码方式；8）GET 请求参数会被完整保留在浏览器历史记录里，而 POST 中的参数不会被保留9）GET 是幂等的，POST 不是幂等的。幂等：其任意多次执行所产生的影响均与一次执行的影响相同。10）GET 和 POST 的适用情况GET：查找资源，且请求无持续性副作用，输入的字段的长度较短（1024字节）POST：请求资源有自作用（向数据库添加行），输入的字段长度会使得 url 较长，且传送的数据不是采用7位 ASCII 编码11）GET 产生一个 TCP 数据包，POST 产生两个 TCP 数据包。GET 中浏览器会把 http header 和 data 一并发送出去，服务器相应200。POST 中浏览器会先发送 header，服务器相应100 continue，浏览器再发送 data，服务器相应 200 ok（但不是所有浏览器都会发送两次，例如Firefox）12）GET 不一定比 POST 安全，因为用 GET 对数据进行查询，POST 主要对数据进行增删改，只要不把保密信息放进去还是安全的。
```



#### 17. 输入 URL 到返回网页的过程中发生了什么

```
1. 输入网址 URL。2. 浏览器获取url后，通过DNS解析获得网址的对应 IP 地址。   1）先查看浏览器缓存（浏览器会记录 DNS 一段时间）；   2）如果在浏览器缓存中不包含这个记录，则会使系统调用操作系统，获取操作系统的记录（保存最近的 DNS 查询缓存）；   3）如果上述两个步骤均不能成功获取 DNS 记录，继续搜索本地路由器缓存或 ISP 缓存；   4）如果任一缓存中有，会直接在屏幕中显示页面内容。若没有，则跳到第三步操作。3. 如果 本地/ISP DNS 服务器没有找到结果，它会发送一个递归查询请求，一层一层向高层 DNS 服务器做查询，直到查询到起始授权机构，最后把结果返回。4. 浏览器与服务器 通过 TCP 三次握手协商来建立一个 TCP 连接。5. 浏览器服务器 发送一个 HTTP 请求报文（包括请求头和请求体）6. 服务器处理请求并返回一个 HTTP 响应报文（包括响应头和响应体）7. 浏览器收到响应对象，判断 http 响应状态码，进行客户端渲染，生成 Dom 树、 Css 样式树、执行 Js 交互8. 构建渲染树，计算并布局。9. 浏览器调用渲染引擎绘制页面。
```



#### 18. Cookie & Session

```
由于 HTTP 协议是无状态的协议，所以服务端需要记录用户的状态时，通过创建 Session 来记录，并跟踪用户。第一次创建 Session 时，服务端会在 HTTP 协议中告诉客户端，需要在 Cookie 里面记录一个Session ID，以后每次请求把该会话 ID发送到服务器。若 Cookie 被禁用，则使用 URL 重写来跟踪会话，URL 后面会附加 sid=xxxxx。Cookie 则是存放在客户端用来保存用户信息的一种机制。1. cookie 数据存放在客户的浏览器上，session 数据放在服务器上。2. cookie 不是很安全，别人可以分析存放在本地的 cookie 并进行 cookie 欺骗,考虑到安全应当使用 session。3. session 会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能，考虑到减轻服务器性能方面，应当使用 cookie。4. 单个 cookie 保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。5. 建议：将登陆信息等重要信息存放为 session，其他信息如果需要保留则放在 cookie 中。
```



#### 19. 正向代理、反向代理、中间人攻击

```
1. 正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端；2. 反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端;3. 中间人攻击，即通过拦截正常的网络通信数据，并进行数据篡改和嗅探
```



#### 20. URI & URL

```
URI 是统一资源标识符；URL 是统一资源定位符；每个 URL 都是 URI，但不一定每个 URI 都是 URL。这是因为 URI 还包括一个子类，即统一资源名称 (URN)，它命名资源但不指定如何定位资源。
```



#### 21. 密码学相关

```
1）对称加密：分为流式、分组两种，加密和解密都是使用的同一个密钥。例如：DES、AES-GCM、ChaCha20-Poly1305等；2）非对称加密：加密使用的密钥和解密使用的密钥是不相同的，分别称为：公钥、私钥，公钥和算法都是公开的，私钥是保密的。非对称加密算法性能较低，但是安全性超强，由于其加密特性，非对称加密算法能加密的数据长度也是有限的。例如：RSA、DSA、ECDSA、 DH、ECDHE等；3）哈希算法：将任意长度的信息转换为较短的固定长度的值，通常其长度要比信息小得多，且算法不可逆。例如 MD5、SHA-1、SHA-2、SHA-256 等；4）数字签名：即在信息后面再加上一段内容（信息经过hash后的值），可以证明信息没有被修改过。hash值一般都会加密后（也就是签名）再和信息一起发送，以保证这个hash值不被修改。
```



#### 22. HTTP 状态码

```
常见1）200 ok，请求成功	 204 请求成功，但无内容2）301 永久重定向，资源被永久转移到其它 URL 上	 302 临时重定向	 303 临时重定向，但最好使用 GET 请求来请求资源3）400 语法错误	 401 需要认证信息	 403 forbidden	 404 资源不存在4）500 服务器未曾遇到的错误	 502 坏网关   503 服务器宕机
```



#### 23. DNS 原理

```

```



#### 24. HTTP 长短连接

```
实质上是 TCP 协议的长短连接1）HTTP1.0 中默认短连接，浏览器和服务器每进行一次 HTTP 操作，就建立一次连接，任务结束就中断连接。2）HTTP1.1 中默认长连接，在头部加入 Connection:keep-alive，如果客户端想要再次访问该服务器上的网页，会继续使用原来已经建立的连接，但只会保持一段时间，再服务器软件中（如Apache）中可以设定该时间。实现长连接需要服务器和客户端都支持长连接。
```



#### 25. IPv4 地址不够解决办法

```
1.NAT 网络地址转换（私网IP），还可以避免来自网络外部的攻击，隐藏并保护网络内部的计算机2.IPv6
```



### 三、HTTP







### 三、数据结构



#### 1. DFS 和 BFS

```
1）不用递归，BFS 类似队列，DFS 类似栈；2）深度优先算法占内存少，一棵一棵树遍历，适合深度较大的情况。广度优先算法占内存多，需要遍历完整个数据，适合深度较小的情况。
```



#### 2. 红黑树

```
应用：Java 集合中的 TreeSet 和 TreeMap，C++ STL 中的 set、map，以及 Linux 虚拟内存的管理，都是通过红黑树来实现的。特性：1）每个节点或者是黑色，或者是红色；2）根节点是黑色；3）每个叶子节点（NIL）是黑色。 （注意：这里叶子节点，是指为空（NIL或NULL）的叶子节点！）；4）如果一个节点是红色的，则它的子节点必须是黑色的；5）从一个节点到该节点的子孙节点的所有路径上包含相同数目的黑节点；红黑树的基本操作是添加、删除。在对红黑树进行添加或删除之后，都会用到旋转方法，添加或删除红黑树中的节点之后，红黑树就发生了变化，可能不满足红黑树的5条性质，也就不再是一颗红黑树了，而是一颗普通的树。而通过旋转，可以使这颗树重新成为红黑树。简单点说，旋转的目的是让树保持红黑树的特性。
```



#### 3. 排序算法

```
1. 快速排序分治法思想2. 堆排序   1）将带排序的序列构造成一个大顶堆（升序排列）；   2）将堆顶元素和最后一个元素交换，然后将剩下的节点重新构造成一个大顶堆；   3）重复步骤2；   最后一个非叶子节点的位置：arr.length / 2 -1   左节点索引：i * 2 + 1   右节点索引：i * 2 + 23. 归并排序分治法思想4. 桶排序...
```



#### 4. 链表 & 数组

```
链表：1. 插入删除速度快2. 内存利用率高3. 大小不固定，可扩展性强4. 查找效率低，不能随机查找，必须从头结点开始遍历数组：1. 插入删除效率低2. 内存利用率低，必须要有连续的内存空间3. 大小固定，不能动态扩展4. 随机访问性强，查找速度快
```



#### 5. 分治算法

```
将一个难以直接解决的大问题，分割成一些规模较小的相同问题，以便各个击破，分而治之，如排序算法(快速排序，归并排序)，傅立叶变换(快速傅立叶变换)一般步骤：1. 分解：将原问题分解为若干个规模较小，相互独立，与原问题形式相同的子问题；2. 解决：若子问题规模较小而容易被解决则直接解，否则递归地解各个子问题3. 合并：将各个子问题的解合并为原问题的解。
```



#### 6. 二分查找



#### 7. 排序算法的稳定性

```
对于序列中相同的记录，若经过排序，这些记录的相对次序保持不变，则称这种排序算法是稳定的，否则称为不稳定的。不稳定排序算法：堆排序、快速排序、希尔排序、直接选择排序；稳定排序算法：基数排序、冒泡排序、直接插入排序、折半插入排序、归并排序。
```



#### 8. 递归 & 迭代

```
递归：一个函数不断地调用函数自己；迭代：每一次迭代的结果会作为下一次迭代的初始值。递归的效率较低，且消耗更多的资源，因为递归的每一层返回点、局部量等开辟了栈来存储。递归次数过多容易造成栈溢出等。应该尽量避免。
```



#### 9. 二叉树

```
完全二叉树如果编号为i（1≤i≤n）的结点与满二叉树中编号为i的结点在二叉树中的位置相同，则这棵二叉树称为完全二叉树。
```



#### 10. hash

```
定义：将任意长度的二进制值串映射为固定长度的二进制值串。特点：1）不能反向推导出原始数据（单向性）；2）对输入数据敏感（该变一位即大变样）；3）哈希冲突概率小，即哈希值相同的概率小；4）计算速度快。如何结果哈希冲突：1）开放地址法（如果存在，则进行数字加减）；2）链式地址法：对于相同的值，使用链表进行连接，使用数组存储每个链表；3）建立公共溢出区，存放所有有哈希冲突的数据；4）再次做哈希处理。
```





### 四、数据库



#### 1. 数据库基本概念

```

```



#### 2. 索引 & B+ 树（Mysql）

```
1. 索引：用于快速找出在某个列中有一特定值的行，不使用索引，MySQL必须从第一条记录开始读完整个表索引的优点：1）可以大大提高数据查询的速度；索引的缺点：1）但创建索引和维护索引要耗费时间，并且随着数据量的增加所耗费的时间也会增加；2）对数据表进行增删改时索引也需要动态维护，因此数据的维护速度也增加了；3）索引会占据存储空间，使索引文件比数据文件更快到达上限值。如果数据更新频繁，不建议使用索引。InnoDB MySQL的数据库引擎之一。2. B+ 树的进化关系：二叉搜索树 -> 平衡二叉树 -> B 树 -> B+ 树1）二叉搜索树，虽然大多数情况下可以提高查询速度，但极端下二叉搜索树可能会退化成单链表，因此查询效率很低。2）平衡二叉树，需要树的左右子树高度差 <= 1，可以解决上述问题。但由于内存的易失性，一般来说还是会将数据和索引存储在磁盘中，磁盘的读取速度会很慢，因此需要将磁盘块一次性取出再读（不是一条一条读取）3）B 树，每个节点称为页，页即磁盘块，每个节点拥有更多的子节点，子节点的个数一般称为阶，因此高度会降低。B 树节点中不仅存储键值，也会存储数据。4) B+ 树，非叶子节点上是不存储数据的，仅存储键值，InnoDB 中页的默认大小是 16KB，因为如果不存储数据，可以存储更多的键值，因而树的阶数就会更大，树就会更矮更胖，这样查找数据进行磁盘的 IO 次数就可以减少，数据查询的效率就会更快。其次数据按照顺序排列，B+ 树中各个页之间通过双向链表连接，叶子节点中的数据是通过单向链表连接。不同的数据库引擎的 B+ 树实现也不同。在 MyISAM 中，B+ 树索引的叶子节点并不存储数据，而是存储数据的文件地址。B+ 树是聚集索引。
```



#### 3. 聚集索引 & 非聚集索引

```
1）聚集索引：数据行的物理顺序与列值（一般是主键的那一列）的逻辑顺序相同，一个表中只能拥有一个聚集索引；（汉语拼音的字母）2）非聚集索引：该索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同，一个表中可以拥有多个非聚集索引；（汉语拼音的偏旁）非聚集索引在索引没有覆盖到对应的列的时候需要进行二次查询，因此聚集索引的速度往往会更占优势。
```



#### 4. 数据库 Mysql & 算法

```

```



#### 5. ACID

```
事务有4个特征，分别是原子性、一致性、隔离性和持久性，简称事务的ACID特性。1. 原子性（atomicity)一个事务要么全部提交成功，要么全部失败回滚，不能只执行其中的一部分操作；2. 一致性（consistency)事务的执行不能破坏数据库数据的完整性和一致性，一个事务在执行之前和执行之后，数据库都必须处于一致性状态；3. 隔离性（isolation）事务的隔离性是指在并发环境中，并发的事务是相互隔离的，一个事务的执行不能被其他事务干扰；4. 持久性（durability）一旦事务提交，那么它对数据库中的对应数据的状态的变更就会永久保存到数据库中；
```



#### 6. Mysql group by





#### 7. B+索引 & hash索引

```
hash：等值查询效率高，不能排序，不能进行范围查询B+：数据有序，范围查询
```



#### 8. 数据库的事务隔离级别

```
当多个线程都开启事务操作数据库中的数据时，数据库系统要能进行隔离操作，以保证各个线程获取数据的准确性。三种误读：1）脏读：事务B读取事务A还没有提交的数据；2）不可重复读：指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据，并做了修改，造成两次读的不一致；3）幻读：一个事务在前后两次查询同一个范围的时候，后一次查询看到了前一次查询没有看到的行。不可重复读针对的是 update 或 delete，幻读针对 insert。这些误读都是由数据库读的一致性问题引起的。四个事务隔离级别：1）READ UNCOMMITED (未提交读)在 read uncommited级别, 事务中的修改,即使没有提交,对其他事务也都是可见的, 事务可以读取未提交的数据,这也被称为脏读(DIrty Read).这个级别会导致很多问题,从性能上说, Read Uncommited 不会比其他的级别好太多,但缺乏其他级别的很多好处,除非真的非常必要的理由,在实际应用中一般很少使用2）READ COMMITED (提交读)大多数数据库系统的默认隔离级别都是 Read Commited (但MySQL不是), Read Commited满足前面提到的隔离性的简单定义: 一个事务开始时,只能 "看见" 已经提交的事务所做的修改, 换句话说, 一个事务从开始直到提交之前,所做的任何修改对其他事务都是不可见的. 这个级别有时候也叫做不可重复读 (nonrepeatable read),因为两次执行同样的查询,可能会得到不一样的结果3）Repeatable Read (可重复读)Repeatable Read 解决了脏读的问题,该级别保证了同一个事务多次读取同样的记录的结果是一致的, 但是理论上,可重复读隔离级别还是无法解决另外一个幻读(Phantom Read)的问题, 所谓幻读, 指的是当某个事务再次读取某个范围内的记录时,另外一个事务又在该范围内插入了新的记录,当之前的事务再次读取该范围的记录时,会产生幻行(Phantom Row), InnoDB 和 XtraDB 存储引擎通过多版本并发控制(MVCC)解决了幻读的问题4）Serializable (可串行化)Serializable 是最高的隔离级别, 它通过强制事务串行执行, 避免了前面说的幻读的问题,简单来说,Serializable会在读取的每一行数据上都加锁,所以可能导致大量的超时和锁争用的问题,实际应用中也很少用到这个隔离级别,只有在非常需要确保数据的一致性而且可以接受没有并发的情况下,才考虑采用该级别可重复读是 MySQL的默认事务隔离级别
```



#### 9. 数据库分库分表

```
分库和分表的目的：减小数据库的单库单表负担，提高查询性能，缩短查询时间。分表目的：减小数据库的单表负担，将压力分散到不同的表上，同时因为表中的数据少了，可以调高数据的查询性能，缩短查询时间，可以缓解表锁问题。分表策略可以分为：垂直拆分和水平拆分水平分表：取模分表属于随机库内分表：仅仅解决了单表数据过大的问题，并没有把单表的数据分散到不同的物理机上，因此并不能减轻 MySQL 服务器的压力，仍然存在同一物理机的资源竞争瓶颈（CPU、内存、磁盘IO、网络带宽等）
```



#### 10. MySQL 中 CHAR & VARCHAR

```
1）char 长度不可变，用空格填充到指定长度大小varchar 长度可变2）char 存取速度比 varchar 快3）char 英文字符 1 字节，汉字 2 字节varchar 英文字符 1 字节，汉字 1 字节
```



#### 11. MySQL 索引引擎 InnoDB & MyISAM

```
1）事务：MyISAM 不支持，InnoDB 支持2）锁级别：MyISAM 表锁，InnoDB 行锁及外键约束3）MyISAM 存储表的总行数，InnoDB 不存储4）MyISAM 采用非聚集索引，B+树叶子存储指向数据文件的指针。InnoDB 主键索引采用聚集索引， B+树叶子存储数据使用场景：MyISAM 适用于插入不频繁，查询频繁的场景InnoDB 适合可靠性高，
```



#### 12. Redis 相关

```
redis 简介：一个开源的内存中的数据结构存储系统，可以用作：数据库、缓存和消息中间件五种数据类型：1）字符串（String），适合缓存、计数器、session；2）散列（Hash），适合缓存，比 String 节省空间；3）列表（List），适合时间轴，或者构成别的数据结构：栈（lpush+lpop）、队列（lpush+rpop）、有限结合（lpush+ltrim）、消息队列（lpush+brpop）；4）集合（Set），适合点赞、点踩、收藏等；5）有序集合（ZSet），适合排行榜。为什么那么快：1. 完全基于内存，绝大部分请求是内存操作；2. 数据结构简单 且 数据操作简单；3. Redis 单线程，避免了不必要的上下文切换和竞争，无多线程切换消耗CPU，无加锁释放锁的问题，不会出现死锁的问题；4. 使用多路I/O复用模型，非阻塞IO（epoll 是只轮询那些真正发出了事件的流，可以同时监察多个流的 I/O 事件）；5. 使用底层模型不同，Redis 自己构建了 VM 机制。为什么采用单线程：CPU 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存的大小或者网络带宽，因此单线程就够了。Redis怎样防止异常数据不丢失：1. RDB 持久化：将某个时间点的所有数据都存放到硬盘上（别的服务器也可以读取）；2. AOF 持久化：将写命令添加到 AOF 文件（Append Only File）的末尾。缓存穿透：客户持续向服务器发起对不存在服务器中数据的请求。客户先在 Redis 中查询，查询不到后去数据库中查询（增加接口层增加校验，id 不符的直接拦截，缓存和数据库中都找不到的数据就将 key-value 对写为 key-null，防止反复暴力攻击）；缓存击穿：一个很热门的数据，突然失效，大量请求到服务器数据库中（热点数据永不过期）；缓存雪崩：大量数据同一时间失效（1.缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生；2.如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中）。redis 适用场景：1. 缓存（最常见）2. 排行榜（Zset 适合）3. 计算器/限速器（Redis 中原子性的自增操作）4. 简单消息队列5. 好友关系6. ....redis key过期策略和淘汰策略：key 分为设置了过期时间的 key 和未设置过期时间的 key，其中过期策略只涉及设置了过期时间的key，淘汰策略涉及了两者。1）过期策略：分为主动过期、被动过期   主动过期：redis 将所有设置了过期时间的 key 放入一个字典中，然后隔段时间（1秒10次）去字典中随机挑选20个 key，查看是否过期，如果过期的比例超过了25%，则进行清理，直到过期比例降到25%以下；   被动过期：当 redis client 访问 key 时，redis server 会检查此 key 是否过期，如果过期了删除该 key 并返回 nil，反之返回对应的 key；2）淘汰策略：当前已用内存超过 maxmemory 限定时，触发主动过期清理策略；淘汰策略分为：volatile-lru：只对设置了过期时间的key进行LRU（默认值）allkeys-lru ： 删除lru算法的keyvolatile-random：随机删除即将过期keyallkeys-random：随机删除volatile-ttl ： 删除即将过期的noeviction ： 永不过期，返回错误redis 实现高并发： // TODOredis 实现分布式锁，及原理： // TODOredis 实现缓存： // TODO
```





### 五、C++



#### 1. C & C++的区别

```
1. C++是面向对象的语言，而C是面向过程的结构化编程语言2. C++具有重载、继承和多态三种特性 3. C++相比C，增加多许多类型安全的功能，比如强制类型转换等4. C++支持范式编程，比如模板类、函数模板等
```



#### 2. 引用 & 指针

```
1)  指针是一个实体，需要分配内存空间。引用只是变量的别名，不需要分配内存空间。2)  引用在定义的时候必须进行初始化，并且不能够改变。指针在定义的时候不一定要初始化，并且指向的空间可变。（注：不能有引用的值不能为NULL） 3)  有多级指针，但是没有多级引用，只能有一级引用。 4)  指针和引用的自增运算结果不一样（指针是指向下一个空间，引用是引用的变量值加1） 5)  sizeof 引用得到的是所指向的变量（对象）的大小，而sizeof 指针得到的是指针本身的大小。 6)  引用访问一个变量是直接访问，而指针访问一个变量是间接访问。 7)  使用指针前最好做类型检查，防止野指针的出现；8)  引用底层是通过指针实现的；9)  作为参数时也不同，传指针的实质是传值，传递的值是指针的地址；传引用的实质是传地址，传递的是变量的地址。
```



#### 3. 强制类型转换

```
1）static_cast：用于数据的强制类型转换，强制将一种数据类型转换为另一种数据类型。用于基本数据类型的转换。用于类层次之间的基类和派生类之间 指针或者引用 的转换（不要求必须包含虚函数，但必须是有相互联系的类），进行上行转换（派生类的指针或引用转换成基类表示）是安全的；进行下行转换（基类的指针或引用转换成派生类表示）由于没有动态类型检查，所以是不安全的，最好用 dynamic_cast 进行下行转换。可以将空指针转化成目标类型的空指针。可以将任何类型的表达式转化成 void 类型。2）const_cast：强制去掉常量属性，不能用于去掉变量的常量性，只能用于去除指针或引用的常量性，将常量指针转化为非常量指针或者将常量引用转化为非常量引用（注意：表达式的类型和要转化的类型是相同的）。3）reinterpret_cast：改变指针或引用的类型、将指针或引用转换为一个足够长度的整型、将整型转化为指针或引用类型。4）dynamic_cast：其他三种都是编译时完成的，动态类型转换是在程序运行时处理的，运行时会进行类型检查。只能用于带有虚函数的基类或派生类的指针或者引用对象的转换，转换成功返回指向类型的指针或引用，转换失败返回 NULL；不能用于基本数据类型的转换。在向上进行转换时，即派生类类的指针转换成基类类的指针和 static_cast 效果是一样的，（注意：这里只是改变了指针的类型，指针指向的对象的类型并未发生改变）。
```



#### 4. 内存泄漏

```
由于疏忽或错误导致的程序未能释放已经不再使用的内存。常指堆内存泄漏，因为堆是动态分配的，而且是用户来控制的，如果使用不当，会产生内存泄漏。解决办法：1）内部封装：将内存的分配和释放封装到类中，在构造的时候申请内存，析构的时候释放内存。2）智能指针内存泄漏检测工具：valgrind
```



#### 5. 智能指针

```
提出目的：为了解决动态内存分配时带来的内存泄漏以及多次释放同一块内存空间。具体分为：1）共享指针（shared_ptr）资源被多个指针共享，使用计数机制表明资源被几个指针共享，通过 use_count() 查看，可通过 unique_ptr、weak_ptr 来构造，调用 release() 释放资源所有权，计数 -1，当计数为 0 时，自动释放内存空间，从而避免内存泄漏。2）独占指针（unique_ptr）独享所有权的智能指针，资源只能被一个指针占用，该指针不能拷贝构造和赋值，但可以移动构造和移动赋值。3）弱指针（weak_ptr）指向 share_ptr 指向的对象，能够解决由 shared_ptr 带来的循环引用问题。
```



#### 6. 面向对象的三大特性

```
三大特性：封装、继承、多态多态：指同样的消息被不同的对象接收时导致不同的行为。1. overload(重载)（编译时多态）在 C++ 程序中，可以将语义、功能相似的几个函数用同一个名字表示，但参数不同（包括类型、顺序不同），即函数重载。（1）相同的范围（在同一个类中）；（2）函数名字相同；（3）参数不同；（4）virtual 关键字可有可无。2. override(覆盖，重写)（运行时多态）是指派生类函数覆盖基类函数，特征是：（1）不同的范围（分别位于派生类与基类）；（2）函数名字相同；（3）参数相同；（4）基类函数必须有 virtual 关键字。3. overwrite(隐藏)是指派生类的函数屏蔽了与其同名的基类函数，规则如下：（1）如果派生类的函数与基类的函数同名，但是参数不同。此时，不论有无 virtual 关键字，基类的函数将被隐藏（注意别与重载混淆）。（2）如果派生类的函数与基类的函数同名，并且参数也相同，但是基类函数没有virtual 关键字。此时，基类的函数被隐藏（注意别与覆盖混淆）。动态联编（运行时多态）通过虚函数表来实现，编译器处理虚函数的方法：给每个对象添加一个隐藏成员，用于保存一个指向函数地址数组的指针。这个数组称为“虚函数表”。虚函数表实质是一个指针数组，里面存的是虚函数的函数指针。
```



#### 7. 内存分区 & 堆栈

```
C++中，内存分为5个区：堆、栈、自由存储区、全局/静态存储区和常量存储区。1. 栈：是由编译器在需要时自动分配，不需要时自动清除的变量存储区。通常存放局部变量、函数参数等。2. 堆：是由 new 分配的内存块，由程序员释放（编译器不管），一般一个 new 与一个 delete 对应，一个 new[] 与一个 delete[] 对应。如果程序员没有释放掉，资源将由操作系统在程序结束后自动回收。堆的特性：1）堆中某个结点的值总是不大于或不小于其父结点的值，2）堆总是一棵完全二叉树。3. 自由存储区：是由 malloc 等分配的内存块，和堆十分相似，用 free 来释放。4. 全局/静态存储区：全局变量和静态变量被分配到同一块内存中（在 C 语言中，全局变量又分为初始化的和未初始化的，C++ 中没有这一区分）。5. 常量存储区：这是一块特殊存储区，里边存放常量，不允许修改。（注意：堆和自由存储区其实不过是同一块区域，new底层实现代码中调用了 malloc，new 可以看成是 malloc 智能化的高级版本）
```



#### 8. static

```
static 使函数、变量成为静态的，改变了变量的生存周期，使其生存期延长到程序运行结束。被其修饰的变量只会初始化一次。static 修饰类：1）修饰成员变量需要在类外初始化，属于类，被所有类的对象共享，static 修饰的变量不占类的空间；2）修饰的函数，属于类，被所有类的对象共享，不能访问类的非静态成员和外部函数，并且无 this 指针，只能访问静态成员（静态成员变量和静态函数）
```



#### 9. const

```
const 修饰变量，变量不能被修改；const 修饰成员函数，不能修改任何类型的成员变量，同时不能调用非 const 的成员函数。
```



#### 10. i++是原子操作吗

```
不是，其执行要分为3步：1、读内存到寄存器；2、在寄存器中自增；3、写回内存。
```



#### 11. STL 中的数据结构

```
1）vector：数组实现，拥有一段连续的内存空间，并且起始地址不变。因此能高效的进行随机存取，时间复杂度为o(1);但因为内存空间是连续的，所以在进行插入和删除操作时，会造成内存块的拷贝，时间复杂度为o(n)。另外，当数组中内存空间不够时，会重新申请一块内存空间并进行内存拷贝。vs 1.5倍，gcc 2倍常用这两种，倍数影响效率2）list：双向链表实现，因此内存空间是不连续的。只能通过指针访问数据，所以list的随机存取非常没有效率，时间复杂度为o(n);但由于链表的特点，能高效地进行插入和删除。对比：如果需要高效的随机存取，而不在乎插入和删除的效率，使用vector；如果需要大量的插入和删除，而不关心随机存取，则应使用list。3）map基于红黑树实现，红黑树具有自动排序的功能，因此map内部所有的数据，在任何时候，都是有序的。适合有顺序要求的问题。优点：有序；缺点：空间占用率高。4）unordered_map基于Hash表的数据结构，通过把关键码值映射到Hash表中一个位置来访问记录，查找的时间复杂度可达到O(1)，其在海量数据处理中有着广泛应用。适合查找问题。优点：查找速度非常的快，执行效率比map高。缺点：建表时间长。5）set基于红黑树实现，
```



#### 12. 指针函数 & 函数指针

```
1. 指针函数：返回指针的函数，其本质是一个函数，而该函数的返回值是一个指针   e.g. int *fun(int x,int y);2. 函数指针：其本质是一个指针变量，该指针指向该函数   e.g. int (*fun)(int x,int y);   可以将别的函数地址赋给它：fun = &Function；                         fun = Function;   调用函数指针的方式：x = (*fun)();                    x = fun();
```



#### 13. 拷贝构造函数

```
用其它对象的数据来初始化新对象的内存，在初始化阶段进行。P.S. 转换构造函数：将其它类型转换为当前类类型需要借助转换构造函
```



#### 14. malloc & new、delete & free

```
在使用的时候 new、delete 搭配使用，malloc、free 搭配使用。1）malloc、free 是库函数，而new、delete 是关键字。new 申请空间时，无需指定分配空间的大小，编译器会根据类型自行计算；malloc 在申请空间时，需要确定所申请空间的大小。2）new 申请空间时，返回的类型是对象的指针类型，无需强制类型转换，是类型安全的操作符；malloc 申请空间时，返回的是 void* 类型，需要进行强制类型的转换，转换为对象类型的指针。3）new 分配失败时，会抛出 bad_alloc 异常，malloc 分配失败时返回空指针。4）对于自定义的类型，new 首先调用 operator new() 函数申请空间（底层通过 malloc 实现），然后调用构造函数进行初始化，最后返回自定义类型的指针；delete 首先调用析构函数，然后调用 operator delete() 释放空间（底层通过 free 实现）。malloc、free 无法进行自定义类型的对象的构造和析构。5）new 操作符从自由存储区上为对象动态分配内存，而 malloc 函数从堆上动态分配内存。（自由存储区不等于堆）
```



#### 15. 编译过程

```
1）预编译：对伪指令（#开头）和特殊符号进行处理2）编译：读取源程序（字符流），对之进行词法和语法分析，将高级语言指令转换为功能等效的汇编代码（.cpp -> .s）3）汇编：把汇编语言代码翻译成目标机器指令的过程（.s -> .o）4）链接：将有关的目标文件彼此相连接，（.o -> 可执行）.cpp 文件中的函数引用了另一个 .cpp 文件中定义的符号或者调用了某个库文件中的函数。那链接的目的就是将这些文件对应的目标文件连接成一个整体静态链接：函数的代码将从其所在的静态链接库中被拷贝到最终的可执行程序中。动态链接：函数的代码被放到称作是动态链接库或共享对象的某个目标文件中。链接程序此时所作的只是在最终的可执行程序中记录下共享对象的名字以及其它少量的登记信息。在此可执行文件被执行时，动态链接库的全部内容将被映射到运行时相应进程的虚地址空间。动态链接程序将根据可执行程序中记录的信息找到相应的函数代码。对比：动态链接会比静态链接节省内存，因为可以共享对象的代码，但某些情况下可能会出现性能上的损害。#include 中的 "" 会在当前目录下搜索头文件，再到系统默认目录下搜索。
```



#### 16. 初始化列表的作用

```
1）类成员中存在常量，如 const int a，只能用初始化不能赋值；2）类成员中存在引用，同样只能使用初始化不能赋值；3）提高效率；4）只有通过初始化列表才能构造父类的 private 成员。也可以这么理解：一个类里的所有构造函数都是有参数的，那么这样的类如果作为别的类的成员变量，必须显式的初始化它，那么就只能通过成员初始化列表来完成初始化。
```



#### 17. 左值右值 & 左值右值引用

```
1）左值：指表达式结束后依然存在的持久对象；2）右值：表达式结束就不再存在的临时对象。区别：左值持久，右值短暂。左值引用 & 右值引用1）左值引用不能绑定到要转换的表达式、字面常量或返回右值的表达式。右值引用恰好相反，可以绑定到这类表达式，但不能绑定到一个左值上。2）右值引用必须绑定到右值的引用，通过 && 获得。右值引用只能绑定到一个将要销毁的对象上，因此可以自由地移动其资源。
```



#### 18. 深拷贝 & 浅拷贝

```
如果一个类拥有资源，该类的对象进行复制时，如果资源重新分配，就是深拷贝，否则就是浅拷贝。深拷贝：该对象和原对象占用不同的内存空间，既拷贝存储在栈空间中的内容，又拷贝存储在堆空间中的内容。浅拷贝：该对象和原对象占用同一块内存空间，仅拷贝类中位于栈空间中的内容。当类的成员变量中有指针变量时，最好使用深拷贝。
```



#### 19. new & malloc

```
new 是关键字，malloc 是库函数
```



#### 20. 野指针 & 悬空指针

```
1）悬空指针：若指针指向一块内存空间，当这块内存空间被释放后，该指针依然指向这块内存空间，此时，称该指针为“悬空指针”。例如：void *p = malloc(size);free(p); 此时，p 指向的内存空间已释放， p 就是悬空指针。2）野指针：“野指针”是指不确定其指向的指针，未初始化的指针为“野指针”。例如：void *p; 此时 p 是“野指针”。
```



#### 21. 常量指针和指针常量

```
1）常量指针：常量指针本质上是个指针，只不过这个指针指向的对象是常量。const int * p;int const * p;只要 const 在 * 左边指针指向的对象不能通过这个指针来修改，限制了通过这个指针修改变量的值。2）指针常量：指针常量的本质上是个常量，只不过这个常量的值是一个指针。const int var;int * const c_p = &var; 只要 const 位于 * 右侧指针的内容可以改变，但指针的指向不可以改变
```



#### 22. 虚函数 & 纯虚函数

```
1）含有纯虚函数的类称为抽象基类；2）虚函数可以直接使用，纯虚函数必须在派生类中实现后才能使用；3）定义形式不同：虚函数在定义时在普通函数的基础上加上 virtual 关键字，纯虚函数定义时除了加上 virtual 关键字还需要加上 = 0；4）虚函数必须实现，否则编译器会报错；5）对于实现纯虚函数的派生类，该纯虚函数在派生类中被称为虚函数，虚函数和纯虚函数都可以在派生类中重写；6）析构函数最好定义为虚函数，特别是对于含有继承关系的类；析构函数可以定义为纯虚函数，此时，其所在的类为抽象基类，不能创建实例化对象。
```



#### 23. 虚函数的实现机制

```
实现机制：虚函数通过虚函数表来实现。虚函数的地址保存在虚函数表中，在类的对象所在的内存空间中，保存了指向虚函数表的指针（称为“虚表指针”），通过虚表指针可以找到类对应的虚函数表。虚函数表解决了基类和派生类的继承问题和类中成员函数的覆盖问题，当用基类的指针来操作一个派生类的时候，这张虚函数表就指明了实际应该调用的函数。虚函数表相关知识点：虚函数表存放的内容：类的虚函数的地址。虚函数表建立的时间：编译阶段，即程序的编译过程中会将虚函数的地址放在虚函数表中。虚表指针保存的位置：虚表指针存放在对象的内存空间中最前面的位置，这是为了保证正确取到虚函数的偏移量。注：虚函数表和类绑定，虚表指针和对象绑定。即类的不同的对象的虚函数表是一样的，但是每个对象都有自己的虚表指针，来指向类的虚函数表。
```



#### 24. 构造函数、析构函数是否需要定义成虚函数

```
1）构造函数一般不定义为虚函数：1.从存储空间的角度考虑：构造函数是在实例化对象的时候进行调用，如果此时将构造函数定义成虚函数，需要通过访问该对象所在的内存空间才能进行虚函数的调用（因为需要通过指向虚函数表的指针调用虚函数表，虽然虚函数表在编译时就有了，但是没有虚函数的指针，虚函数的指针只有在创建了对象才有），但是此时该对象还未创建，便无法进行虚函数的调用。所以构造函数不能定义成虚函数。2.从使用的角度考虑：虚函数是基类的指针指向派生类的对象时，通过该指针实现对派生类的虚函数的调用，构造函数是在创建对象时自动调用的。3.从实现上考虑：虚函数表是在创建对象之后才有的，因此不能定义成虚函数。4.从类型上考虑：在创建对象时需要明确其类型。2）析构函数一般定义成虚函数：析构函数定义成虚函数是为了防止内存泄漏，因为当基类的指针或者引用指向或绑定到派生类的对象时，如果未将基类的析构函数定义成虚函数，会调用基类的析构函数，那么只能将基类的成员所占的空间释放掉，派生类中特有的就会无法释放内存空间导致内存泄漏。
```



#### 25. 静态绑定 & 动态绑定

```
1）静态类型：变量在声明时的类型，是在编译阶段确定的。静态类型不能更改。动态类型：目前所指对象的类型，是在运行阶段确定的。动态类型可以更改。2）静态绑定是指程序在 编译阶段 确定对象的类型（静态类型）。动态绑定是指程序在 运行阶段 确定对象的类型（动态类型）。3）发生的时期不同：如上。对象的静态类型不能更改，动态类型可以更改。注：对于类的成员函数，只有虚函数是动态绑定，其他都是静态绑定。
```



#### 26. 变量

```
1）全局变量：具有全局作用域。全局变量只需在一个源文件中定义，就可以作用于所有的源文件。当然，其他不包含全局变量定义的源文件需要用 extern 关键字再次声明这个全局变量。2）静态全局变量：具有文件作用域。它与全局变量的区别在于如果程序包含多个文件的话，它作用于定义它的文件里，不能作用到其它文件里，即被 static 关键字修饰过的变量具有文件作用域。这样即使两个不同的源文件都定义了相同名字的静态全局变量，它们也是不同的变量。3）局部变量：具有局部作用域。它是自动对象（auto），在程序运行期间不是一直存在，而是只在函数执行期间存在，函数的一次调用执行结束后，变量被撤销，其所占用的内存也被收回。4）静态局部变量：具有局部作用域。它只被初始化一次，自从第一次被初始化直到程序运行结束都一直存在，它和全局变量的区别在于全局变量对所有的函数都是可见的，而静态局部变量只对定义自己的函数体始终可见。
```



#### 27. 如何实现单例模式

```

```



#### 28. 内存池

```

```



#### 29. exit & return

```
1）return 是函数返回调用，如果返回的是 main函数，则为退出程序；	 exit 是在调用处退出程序，运行即退出。2）exit（0）为正常退出，	 exit（1）非零即为异常退出。	 3）return 是关键字，exit 是函数；	 return 是 c 语言提供，exit 是操作系统提供；	 return 是函数退出，exit 是进程退出；	 return 是语言级别的，表示调用堆栈返回，exit 是系统调用级别的，表示进程的结束；
```



#### 30. mutable

```
mutable 意味可变的、易变的，与 const 相反被 mutable 修饰的变量处于可变的状态，即使该变量处于一个被 const 修饰的成员函数中。
```



#### 31. Lambda 表达式

```

```



#### 32. constexpr

```
使常量表达式再编译阶段即计算出结果，而不必等到运行阶段。可用于修饰一个变量
```



#### 33. decltype

```

```



#### 34. 异常处理

```

```



#### 35. debug & release

```

```



#### 36. 基类的析构函数应该写为虚函数

```
如果基类的析构函数不是 virtual 的，那么如果是基类指针指向子类，在调用 delete 时只会释放基类的资源，以为没有 virtual 时是静态绑定的，指针的静态类型为基类指针，因此在delete的时候只会调用基类的析构函数，而不会调用派生类的析构函数。这样可能会造成内存泄漏（如果派生类无需要释放的资源则不会内存泄漏）。
```



#### 37. volatile

```

```



#### 38. 多线程

```

```





### 六、其他、模型框架、智力题、设计题



#### 1. IO 常用模型

```
同步和异步：描述的是用户线程与内核的交互方式：同步是指用户线程发起IO请求后需要等待或者轮询内核IO操作完成后才能继续执行；而异步是指用户线程发起IO请求后仍继续执行，当内核IO操作完成后会通知用户线程，或者调用用户线程注册的回调函数。阻塞和非阻塞：描述的是用户线程调用内核IO操作的方式：阻塞是指IO操作需要彻底完成后才返回到用户空间；而非阻塞是指IO操作被调用后立即返回给用户一个状态值，无需等到IO操作彻底完成。（1）同步阻塞IO（Blocking IO）：即传统的IO模型。（2）同步非阻塞IO（Non-blocking IO）：默认创建的 socket 都是阻塞的，非阻塞IO要求 socket 被设置为 NONBLOCK 。注意这里所说的 NIO 并非 Java 的 NIO（New IO）库。（3）IO多路复用（IO Multiplexing）：即经典的Reactor设计模式，有时也称为异步阻塞IO，Java中的Selector和Linux中的epoll都是这种模型。在Linux下包括了三种，select、poll、epoll。（4）异步IO（Asynchronous IO）：即经典的Proactor设计模式，也称为异步非阻塞IO。（5）信号驱动的IO（Signal Driven IO）：利用信号机制，让内核告知应用程序文件描述符的相关事件。
```



#### 2. select、poll、epoll

```
1）select 时间复杂度 O(n)只知道有 I/O 事件发生了，并不知道具体是哪个流（或者多个流），只好进行无差别轮询，上限 1024 个 fd；2）poll 时间复杂度 O(n)本质和 select 无差别，将用户传入的数组拷贝到内核空间，然后再查看每个 fd 对应的设备状态，只是没有了 最大连接数 的限制，因为其基于链表来存储；3）epoll 时间复杂度 O(1)可以理解为 event poll，由事件驱动（每个事件关联到 fd）而触发，因此复杂度大大降低，同样无 最大连接数 限制。epoll 有 EPOLLLT 和 EPOLLET 两种触发方式， LT （水平触发）是默认模式， ET（边缘触发） 是高速模式：LT 模式下，只要 fd 还有数据可以读，每次 epoll_wait 都会返回它的事件，提醒用户去操作；而 ET 模式下，只会提醒用户一次，直到下次再有数据流入的时候，才会再次提醒，因此用户需要每读一个 fd 将其 bufffer 中的内容读完，直到 read 的返回值 < 请求值，或者遇到 EAGAIN 错误。因此这种情况下，可以忽略系统中大量不需要关心的文件描述符，因此可以大大提高效率。三者本质都是同步 I/O，因此他们都需要在读写事件就绪后自己读写，即读写过程是阻塞的。而异步 I/O 则无需自己读写，异步 I/O 会将数据从内核空间拷贝到用户空间。但也不是 epoll 就是最好的，若连接数较少且都十分活跃的情况下，select 和 poll 的性能会好于 epoll，因为epoll 的通知机制需要很多函数回调。
```



#### 3. 线程池原理

```
1. 为什么用线程池：   1）降低资源消耗。通过重复利用已创建的线程降低线程创建、销毁线程造成的消耗；   2）提高响应速度。当任务到达时，任务可以不需要等待线程创建就能立即执行；   3）提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配、调优和监控。   2. 自定义线程池参数   1）corePoolSize	核心线程数量，线程池维护线程的最少数量；   2）maximumPoolSize 线程池维护线程的最大数量；   3）keepAliveTime	线程池除核心线程外的其他线程的最长空闲时间，超过该时间的空闲线程会被销毁；   4）unit	keepAliveTime的单位，TimeUnit中的几个静态属性：NANOSECONDS、MICROSECONDS、MILLISECONDS、SECONDS；   5）workQueue 线程池所使用的任务缓冲队列；   6）threadFactory 线程工厂，用于创建线程，一般用默认的即可；   7）handler 线程池对拒绝任务的处理策略。
```



#### 5. 消息队列

```

```



#### 6. docker & 虚拟机 & k8s

```
docker 是一个开源的容器引擎，开发者可以打包应用到依赖的一个可移植的容器中。其基于镜像创建。创建速度可以到达秒级，镜像会较小。可以快速的交付和部署。其直接使用实际物理机的硬件资源，并直接利用系统内核。不像虚拟机会在建立一个 Guest OS，docker 无 Guest OS。相当于 docker 和宿主机共用内核。docker engine 是用来运行和管理容器的核心软件。利用 namespace 实现了系统环境的隔离，利用了 cgroup 实现了资源的限制，利用镜像实例实现跟环境的隔离。隔离级别：虚拟机是操作系统级别的资源隔离，而容器本质上是进程级的资源隔离。k8s 管理 docker 集群，且还支持别的容器技术，例如 rocket。k8s 可以实现容器集群的自动化部署、自动化缩容、维护等功能。
```



### 七、设计模式



#### 1. 单例模式（手撕）

```
保证一个类仅有一个实例，并提供一个访问它的全局访问点。1）饿汉模式：程序只要开始，就将其实例化，用到的时候就省去了实例的时间，因此速度和反应较快。2）懒汉模式：只有用到这个用例的时候，再将他实例化，不会浪费，所以效率要高一些。
```

```c++
// 饿汉模式// 线程安全，无需加锁#include <iostream>using namespace std;class A {public:		static A* getInstance() {				if (instance == nullptr) {						instance = new A();				}				return instance;		}private:		static A* instance;		A() {};		A(const A& temp) {};		A& operator=(const A& temp) {};};A* A::instance = nullptr;int main() {		A* a = A::getInstance();		A* b = A::getInstance();		if (a == b) cout << "a == b" << endl;				return 0;}
```

```c++
// 懒汉模式#include <iostream>using namespace std;class A {public:		static A* getInstance() {				if (instance == nullptr) {						instance = new A();				}				return instance;		}protected:  	A() {};private:		static A* instance;		A(const A& temp) {};		A& operator=(const A& temp) {};};A* A::instance = new A();int main() {		A* a = A::getInstance();		A* b = A::getInstance();		if (a == b) cout << "a == b" << endl;				return 0;}
```



#### 2. 工厂模式

```
1）简单工厂模式：主要用于创建对象。用一个工厂来根据输入的条件产生不同的类，然后根据不同类的虚函数得到不同的结果。程序只要开始，就将其实例化，用到的时候就省去了实例的时间，因此速度和反应较快。（根据不同类的虚函数得到不同的结果）2）工厂方法模式...3）抽象工厂模式...
```



#### 3. 观察者模式

```
定义了一种一对多的关系，让多个观察对象同时监听一个主题对象，主题对象发生变化时，会通知所有的观察者，使他们能够更新自己。
```



#### 4. 装饰模式

```
动态地给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生成派生类更为灵活。
```



#### 5. 生产者消费者模式（非典型设计模式）

```
产生数据的模块，称为生产者；而处理数据的模块，就称为消费者；其中模块是个泛指（可以是类、函数、线程、进程）数据单元：每次生产者放到缓冲区的，就是一个数据单元；每次消费者从缓冲区取出的，也是一个数据单元；缓冲区：队列 FIFO优点：1）解耦合，使消费者和生产者不相互依赖；2）支持并发，生产出的数据丢到缓冲区即可做别的事情；3）支持忙闲不均，若消费者来不及处理缓冲区数据，可以通过暂存在缓冲区，然后慢慢处理；线程模式：在线程方式下，生产者和消费者各自是一个线程，生产者写队列，消费者读队列。当队列为空，消费者就稍息（稍事休息）；当队列满（达到最大长度），生产者就稍息。整个流程并不复杂。进程模式：（socket）生产线程 -> 发送缓冲区 -> 发送线程 -> 发送端的 TCP API-> 接收端的 TCP API -> 接收线程 -> 接收缓冲区 -> 消费线程改进方式：1）环形缓冲区（存储空间的分配出现问题／释放非常频繁）；2）双缓冲区...
```



#### 6. 中间件

```
消息中间件作用（3个）：1）异步化提升性能（异步调用，性能调高，例如：用户->A->B，加了 MQ 之后不用再等待B模块的回复了）；2）降低耦合度（例如A，B两个模块（用户->A->B），B模块出错了，需要A模块再处理返回错误或者异常，A再返回用户调用失败，增加中间件之后可以B模块再次调用 MQ，实现解耦）；3）流量削峰（例如A qps10000，B qps 6000，增加了 MQ 之后可以提高 qps 到1w，这样等流量峰值过去，就可以再释放 MQ 中的消息），常用在电商秒杀，抢票系统中；
```



#### 7. 一些系统中的基本概念

```
1）QPS 每秒查询率（Query Per Second），是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准，是应用于特定场景的吞吐量；2）RT 响应时间，是指系统对请求作出响应的时间；3）TPS 吞吐量（Throughput Per Second），是指系统在单位时间内处理请求的数量；
```



### 八、设计题



#### 1. 有10亿个电话号码，如何找出重复的

```
1. 通过哈希算法，将10亿个电话号码按照哈希值落在多个文件中，重复的电话号码有相同的哈希值，肯定位于一个文件中，这样就可以分别对每个文件排序删除重复的电话号码。哈希切割：例如电话号码（取余 1000，放入 0-999 号文件中，大小可以自定），ip 地址转为 10 进制数字。2.使用位图来进行处理。比如说这10亿个数的范围为【0-10亿】，那么就申请一个10亿的数组，数组类型为boolen，只有0和1，0表示没有，1表示有。这样自然而然的就删掉了重复的部分。
```



#### 2. TopK 问题

```
建立一个大小为 K 的最小堆，遍历数组 N，每次入堆的时间复杂度为 logK，因此总的时间复杂度为 NlogK。
```



#### 3. id 全局唯一且自增

```
推特的雪花算法，长度为 64 bit，41 bit 的时间戳，10 bit 的工作 id，12 bit 的序列号。分布式系统内不会产生 ID 碰撞，效率较高，每秒可以产生 26 万个 ID 左右。
```



#### 4. 赛马问题

```
1）64匹马8个跑道，找出最快的4匹马8组各跑一遍，接着各组第一再跑一遍。这样就只剩下（画图）对角线的 10 匹马，并且找到第1，因此找到第一那组的第二，让其他八匹马跑一遍，若第一那组的第三跑第一，则前四确定（8+1+1=10），不然就找前三和第二跑一场（8+1+1+1=11）2）36匹马6赛道，找出最快的3匹马：6+1+1=8
```



#### 5. 设计 RPC 框架

```

```



#### 6. 用 Redis 设计一天人数量的统计系统（key-value）

```

```





### 九、思考题



#### 1. 项目中遇到的挑战

```

```



#### 2. 对未来技术栈和发展的思考

```

```



#### 3. 遇到困难怎么解决的

```

```



### 十、Linux



#### 1. Linux 下检测文件是否存在

```
find /tmp - name 1*/tmp 为路径，1* 为以 1 开头的文件名
```



#### 2. Linux 查看端口状态

```
netstat参数：-u udp 端口-t tcp 端口
```



#### 3. Linux 常见命令

```
ps -ef 查看进程top 查看 cpu、内存awk 三剑客，文本搜索awk 的正则匹配
```



#### 4. top 命令

```
动态显示进程信息第一行：现在时间，运行时间，用户数量，负载（3个）第二行：进程情况第三行：cpu 状态第四行：内存状态第五行：swap 分区信息第七行以下：各进程（任务）的状态监控
```



#### 5. ps awx ef

```
Process Status，提供进程的一次性查看功能：1）进程正在运行和运行的状态；2）进程是否结束；3）进程有没有僵死；4）哪些进程占用了过多的资源。ps -ef：System V 风格ps aux：BSD 风格，aux 会截断 command 列ps -ef | grep java第一个字段 UID（用户）第二个字段 PID（进程号）第三个字段 PPID（父进程号）第四个字段 C（CPU 使用资源的百分比）第五个字段 STIME（系统启动时间）...
```







### 十一、Golang



#### 1. 协程

```
线程的两个问题：1）系统线程会占用非常多的内存空间，2）过多的线程切换会占用大量的系统时间。1）协程可以理解为用户级线程；2）协程运行在线程之上，当一个协程执行完成后，可以选择主动让出，让另一个协程运行在当前线程之上；3）协程并没有增加线程数量，只是在线程的基础之上通过分时复用的方式运行多个协程。4）协程的切换在用户态完成，切换的代价比线程从用户态到内核态的代价小很多。5）线程是抢占式的，协程是非抢占式的，所以需要用户自己释放使用权来切换到其他协程。
```



#### 2. 数组和切片

```
1）数组（array）：定长，不可追加元素，值拷贝（传值），长度是数组类型的一部分；	[10]int 和 [20]int 的一部分2）切片（slice）：可变长，引用拷贝（传引用）
```



#### 3. goroutine

```
特征：1）执行非阻塞，不会等待；2）调度器不能保证多个 goroutine 执行次序；3）go 后面的函数的返回值会被忽略；4）无父子 goroutine 概念，所有 goroutine 被平等地调度和执行；5）程序会为 main 函数单独创建一个 goroutine，遇到别的 go 关键字再创建其他 goroutine；
```



#### 4. chan

```
通道 channel，go 哲学是 “不通过共享内存来通信，通过通信来共享内存”通道可以分为有缓冲和无缓冲，无缓冲的 len 和 cap 都是 0，有缓冲的 len 代表没有被读取的元素数和 cap 代表整个通道容量可通过无缓冲的通道来实现 goroutines 之间的同步
```



#### 5. waitgroup

```
计数信号量，可实现同步Add() 计数器 +1Done() 计数器 -1Wait() 计数器为0，否则一直阻塞着
```





### 十二、Java



#### 1. Hashmap 

```
特性：1）HashMap 存储键值对实现快速存取，允许为null，key值不可重复，若key值重复则覆盖；2）非同步，线程不安全；3）底层是 hash 表，不保证有序。底层原理：基于 hashing 的原理，jdk8 后采用 数组+链表+红黑树的数据结构，通过 put 和 get 存储和获取对象，
```



#### 2. Hashmap 冲突如何解决

```

```





#### 3. 垃圾回收机制（GC）

```

```





### 十三、框架



#### 1. Flask

```

```



#### 2. Beego

```

```

