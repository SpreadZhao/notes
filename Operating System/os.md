# 操作系统笔记

## Process

### Definition

​	An abstraction of a running program

### Internal Structure

* Code Segment(Read-Only)
* Stack Segment
* Data Segment
* 分区：方便安全

### Address Space: 进程之间互不相干

* Kernel Space：系统内核
* User Space：应用程序
* Kernel Mode = Kernel Space + Kernel Privilege
* User Mode = User Space + User Privilege
* Kernel Mode和User Mode区别：<u>Kernel下代码可访问硬件</u>

### PCB - Process Control Block

* 开2个记事本，咋知道关的是哪一个 -> 通过PCB中的PID

### Program和Process区别

* Program 有 Code Segment, Data Segment, Address Space, **没有 PCB, Stack Segment**

### Stack Segment

​		比如 a() 调用 b() 再调用 c(), c() 里有变量t1, t2, t3, b() 里有y1, y2, y3, 则**c() 最后执行，最先执行完毕，t1, t2, t3 最先被分配，最先被释放，正好使用stack管理。**下面是一个例子：

 ```c
 #include <stdio.h>
 #include <string.h>
 int f(char* p){
     long low;	//long在c中是8byte
     long str;
     long top;
     
     low = 1;
     top = 3;
     strncpy((char*)(&str), p, 16);	//现在拷贝16byte
     return 1;
 }
 
 //main会覆盖top还是low呢？
 int main(){
     char mstr[16];
     printf("size of long %ld\n", sizeof(long));
     strncpy(mstr, "12345678abcdefgh", 16);
     f(mstr);
 }
 ```

​		在上面的例子中，我们首先需要了解英特尔的CPU架构：

<img src="img/intelcpu.png" alt="img" style="zoom:50%;" />

由于是从低地址拷贝到高地址，因此拷贝的16byte会将low覆盖。

### Process Model

​		一会儿切一个Process在CPU的一个核上

<img src="img/processmodel.png" alt="img" style="zoom:67%;" />

### Process State

<img src="img/processstate.png" alt="img" style="zoom:67%;" />

#### Process Creation

> **ps tree, 由 systemd(1号)生成其他进程**

##### Four time for creation

* System Initialization

  > 通过1号Process生成其他Process参与初始化界面等

* Execution of a process creation system call by a running process

  > 用系统调用通过一个运行的Process生成另一个Process, **fork()**

* A user request to create a new process

  > 比如./haha

* Initiation of a batch job (批处理任务)

  > 比如bash

##### Implementation-Creation: fork, exec

* Fork

  > Code
  >
  > ```c
  >   #include <unistd.h>
  >   #include <stdio.h>
  >   int main(){
  >       pid_t pid;
  >       /* 子进程拿到的pid是0 */
  >       pid = fork();
  >       /* 父进程拿到的pid是子进程的，>0 */
  >       
  >       //子进程会走这里
  >       if(pid == 0){
  >           while(1){
  >               sleep(1);
  >               printf("Kylin\n");
  >           }
  >       }
  >       //父进程会走这里
  >       if(pid > 0){
  >           while(1){
  >               sleep(1);
  >               printf("My Favorite\n");
  >           }
  >       }
  >   }
  > ```
  >
  > Result
  >
  > <img src="img/myfvkylin.png" alt="img" style="zoom:67%;" />|<img src="img/father2993.png" alt="img" style="zoom:60%;" />
  >
  > **注意：子进程从fork返回处开始执行**
  >
  > *可能的疑问：子进程的PID是0是不是因为从返回处开始执行，导致没有初始化呢？*
  >
  > Another example
  >
  > ```c
  > #include <unistd.h>
  > #include <stdio.h>
  > int main(){
  >     pid_t pid;
  >     pid = fork();
  >     
  >     if(pid == 0){
  >         sleep(1);
  >         printf("Kylin\n");
  >     }
  >     
  >     sleep(1);
  >     printf("My Favorite\n");
  > }
  > ```
  >
  > * Question: 该程序会打印几次Kylin，几次My Favorite？
  > * Ans: 1, 2(父进程不会走==0，只会打My Favorite，子进程即打Kylin，也打My Favorite)

* Execl

  > **不会生成Process，在子进程中使用，用于替换代码，用新的，和父进程不一样的**
  >
  > Code
  >
  > ```c
  > #include <unistd.h>
  > #include <stdio.h>
  > int main(){
  >     pid_t pid;
  >     pid = fork();
  >     
  >     if(pid == 0){
  >         /*
  >         	子进程会执行"ls -l"命令，不会打
  >         	haha，因为execl就是子进程的尽头
  >         */
  >         execl("/bin/ls", "-l", 0);
  >         printf("haha\n");
  >     }
  >     
  >     /* 父进程pid > 0，不断打出hehe */
  >     while(1){
  >         sleep(1);
  >         printf("hehe\n");
  >     }
  > }
  > ```
  >
  > Result
  >
  > ![img](img/execl.png)

#### Process Termination

时机

* Normal exit (voluntary) **老死**

* Error exit (voluntary) **病死， 打开程序发现打不开，有个返回码**

* Fatal error (involuntary) **事故死， 非法访问，被OS干掉**

* Killed by another process (involuntary) **谋杀**

  >Fatal Error
  >
  >```c
  >#include <unistd.h>
  >#include <stdio.h>
  >int main(){
  >    char* p;
  >    p = 0x0;
  >    *p = 'a';
  >    while(1){
  >        sleep(1);
  >        printf("haha\n");
  >    }
  >}
  >```
  >
  >Result
  >
  >![img](img/fatalerror.png)
  >
  >Reason
  >
  >* OS强制退出，因为访问了不让访问的0x0

  >Kill Process
  >
  >`kill -9 <pid> //-9表示杀掉，kill只是发指令`
  >
  >现在有一个进程是3539号
  >
  >* 当`kill -9 3539`后的状态
  >
  >  ![img](img/kill3539.png)
  >
  >* 与此同时，看到3539变为僵尸态
  >
  >  ![img](img/3539zb.png)
  >
  >现在有俩进程，3700的父进程是3699
  >
  >* 若`kill -9 3699`(也就是把孩子他爸干掉)会
  >
  >  ![img](img/kill3699.png)
  >
  >* 与此同时，发现这个孩子被1号进程接管
  >
  >  ![img](img/adopted.png)

##### Process Termination Implementation

* 尸检时

  > 释放Code Segment, Data Segment, Stack Segment

* 尸检完

  > 释放PCB(**Z+这种状态就在PCB里**)

* Address Space

  > 会释放Page Table

* PCB里的内容

  ![img](img/pcbcontent.png)

### Process Model Implementation

#### Process Switching

<img src="img/psswitch.png" alt="img" style="zoom:67%;" />

## Thread

### An example

```c
#include <stdio.h>
#include <pthread.h>

int a;

void* th(void* p){
    int i = 0;
    while(1){
        a = 1; i++;
        sleep(1);
        if(i <= 5){
            printf("haha\n");
        }
    }
}

int main(){
    int i = 0;
    a = 0;
    pthread_t myth;
    pthread_create(&myth, NULL, th, NULL);
    while(1){
        i++; sleep(1);
        if(i <= 5) printf("a = %d, hehe\n", a);
    }
}
```

Result

<img src="img/threadhaha.png" alt="img" style="zoom:50%;" />

### Definition

>进程中正在执行的代码片段，其可以与其他片段并发执行
>
>**<u>一个Process的不同Thread不共享Stack</u>**

### Thread Model

<img src="img/threadmodel.png" alt="img" style="zoom:67%;" />

### Why Thread?

1. 在一个application里有多个活动，其中一些会block，这时把app分成几个能并行的顺序线程，模型会更简单
2. Thread比Process更容易创建/消除
3. ![img](img/whythread3.png)
4. Finally, threads are useful on systems with multiple CPUs, where real parallelism is possible.

### Implementation of  thread model

* TCB(Thread Control Block)

  <img src="img/tcb.png" alt="img" style="zoom:67%;" />

* Three Implementation Way

  1. In User Space

     <img src="img/tinus.png" alt="img" style="zoom:67%;" />

     >优点
     >
     >* 可在不支持Thread的OS上实现
     >* 线程切换快，不用陷入内核
     >* 允许每个Thread有自己的Scheduling Algorithm
     >* 有较好的可扩展性
     >
     >问题：如何实现系统调用
     >
     ><img src="img/howsyscall.png" alt="img" style="zoom:67%;" />

  2. In Kernel Space

     <img src="img/tinks.png" alt="img" style="zoom:67%;" />

     >* 创建Thread要用系统调用，进入Kernel Space
     >
     >* Process Table保存每个Process的状态等
     >
     ><img src="img/tinkspb.png" alt="img" style="zoom:67%;" />

  3. Hybrid

     <img src="img/thybrid.png" alt="img" style="zoom:50%;" />

     ><img src="img/thbdpb.png" alt="img" style="zoom:67%;" />

### POSIX Thread

* IEEE定义的线程包：pthread

  <img src="img/tphread.png" alt="img" style="zoom:67%;" />

* Why POSIX?

  > 可移植，通用

### Pop-Up Thread

* Definition, Advantage

  ><img src="img/pptd.png" alt="img" style="zoom:67%;" />
  >
  ><img src="img/pptd2.png" alt="img" style="zoom:50%;" />

* Why Pop-Up Thread?

  >One reason: remove block
  >
  >传统：将Process或Thread阻塞在一个receive系统调用上，等待message，而Pop-Up Thread在message来时才创建，remove了block

## IPC(Inter Process Communication)

### Race Conditions

<img src="img/rccd.png" alt="img" style="zoom:50%;" />

### Critical Region

>**The part of the program where the shared memory is accessed is called the critical region**

### How to avoid race conditions?

* Mutual Exclusion(十字路口)

  >Avoid two processes access critical region at the same time

* **Implementation**

  * **需要满足以下几个时机**

    1. No two processes may be simutaneously inside their critical regions

       >**不同时在critical region**

    2. No assumptions may be made about speeds of the number of CPUs

       >**要是假设了，移植性不好。因为CPU很快，在快的机子上好使，在慢的上就不行**
       >
       >*一个十字路口，一辆车超快，一辆车超慢，那快的wu一下就过去了，根本不会有race condition，但是对于慢的机子，这俩车有多快就不一定了*

    3. No process running outside its critical region may block other processes

       >**不能在critical region里sleep(占着茅坑不拉屎)**

    4. No process should have to wait forever to enter its critical region

       >**不能饿死**

  * Proposals for achieving mutual exclusion

    * Disabling interrupts

      >**关掉中断，CPU不处理中断处理，不执行Scheduling，其他Process都别想运行**
      >
      >*<u>不好，对CPU数量假设了，若有多个CPU，关了中断也没用，其它CPU上的Process还会运行</u>*

    * Lock variables

      1. Strict Alternation

         <img src="img/strictat.png" alt="img" style="zoom:67%;" />

         >**缺点：2个Process/Thread的顺序是固定的，TURN有个初值，则一定是某一个人先来**

      2. Peterson's solution

         <img src="img/ptsolu.png" alt="img" style="zoom:67%;" />

      3. <img src="img/tsl.png" alt="img" style="zoom:67%;" />

      4. Another TSL - XCHG(Intel)

         <img src="img/xchg.png" alt="img" style="zoom:67%;" />

         >**Exercise：gcc用C语言嵌入汇编语言**

    * TSL Busy Waiting缺点

      * 反复查锁，浪费CPU时间
      * Cause Priority inversion problem

  * 改进：发现别人上锁了，歇一会 -> Sleep and Wakeup

    * Implementation - with Producer Consumer

      ```c
      #define N 100	/*number of slots in the buffer*/
      int count = 0;	/*number of items in the buffer*/
      
      void producer(void){
          int item;
          while (TRUE) { 	/*repeat forever*/
              item = produce item( ); 			/*generate next item*/
              if (count == N) sleep(); 			/*if buffer is full, go to sleep*/
              inser t item(item); 				/*put item in buffer*/
              count = count + 1; 					/*increment count of items in buffer*/
              if (count == 1) wakeup(consumer); 	/*was buffer empty?*/
      	}
      }
      
      void consumer(void)
      {
          int item;
          while (TRUE) { 							/*repeat forever*/
              if (count == 0) sleep(); 			/*if buffer is empty, got to sleep*/
              item = remove item(); 				/*take item out of buffer*/
              count = count − 1; 					/*decrement count of items in buffer*/
              if (count == N − 1) wakeup(producer); /*was buffer full?*/
              consume item(item); 				/*pr int item*/
          }
      }
      ```

      >  **问题：若comsumer先来，count == 0，<u>正要sleep但还没sleep的时候</u>，Process切换到producer，最后wakeup，但这时consumer还没睡呢，导致<u>信号丢失</u>。之后，consumer睡了，producer把仓库填满后也睡了，永远睡下去**

    * 改进 - Semaphore

      ```c
      #define N 100							/*number of slots in the buffer*/
      typedef int semaphore; 					/*semaphores are a special kind of int*/
      semaphore mutex = 1;					 /*controls access to critical region*/
      semaphore empty = N; 					/*counts empty buffer slots*/
      semaphore full = 0; 					/*counts full buffer slots*/
      
      void producer(void){
          int item;
          while (TRUE) { 						/*TRUE is the constant 1*/
              item = produce item( ); 		/*generate something to put in buffer*/
              down(&empty); 					/*decrement empty count*/
              down(&mutex); 					/*enter critical region*/
              inser t item(item); 			/*put new item in buffer*/
              up(&mutex); 					/*leave critical region*/
              up(&full); 						/*increment count of full slots*/
          }
      }
      
      void consumer(void)
      {
          int item;
          while (TRUE) { /*infinite loop*/
              down(&full); 					/*decrement full count*/
              down(&mutex); 					/*enter critical region*/
              item = remove item( ); 			/*take item from buffer*/
              up(&mutex); 					/*leave critical region*/
              up(&empty); 					/*increment count of empty slots*/
              consume item(item); 			/*do something with the item*/
          }
      }
      ```

      >**down：-1，若减完>0,继续；=0，继续，但不能再down了;<0，把自己放进睡眠队列**

      **Semaphore - System Call**

      >Code
      >
  ><img src="img/semacode.png" alt="img" style="zoom: 67%;" />
      >
      >Result
      >
      ><img src="img/semares.png" alt="img" style="zoom:67%;" />

      >  2个Process和Semaphore
      >
      > <img src="img/twosem.png" alt="img" style="zoom: 67%;" />
      >
      > 使用GDB调试
      >
      > <img src="img/semgdb.png" alt="img" style="zoom:67%;" />

    * Semaphore Disadvantage
    
      >**实现在Kernel Space**，生成和消除代价高
    
  * Mutex: Simplified Semaphore

    >**实现在User Space**

    Pthread Calls

    | Thread call           | Description                 |
    | --------------------- | --------------------------- |
    | Pthread_mutex_init    | Create a mutex              |
    | Pthread_mutex_destroy | Destroy an existion mutex   |
    | Pthread_mutex_lock    | Acquire a lock or **block** |
    | Pthread_mutex_trylock | Acquire a lock or **fail**  |
    | Pthread_mutex_unlock  | Release a lock              |

    Example

    ![img](img/mutexex.png)

  * Mutex, Semaphore区别

    * 量级
    * **Mutex进程完了就没了，除非塞到Share Memory**

  * Mutex Implementation in ASM

    ```assembly
    mutex lock:
        TSL REGISTER,MUTEX 				 ; copy mutex to register and set mutex to 1
        CMP REGISTER,#0 				 ; was mutex zero?
        JZE ok 							; if it was zero, mutex was unlocked, so return
        CALL thread yield 				 ; mutex is busy; schedule another thread
        JMP mutex lock 					 ; try again
    ok: RET 							; return to caller; critical region entered
    mutex unlock:
        MOVE MUTEX,#0 					 ; store a 0 in mutex
        RET 							; return to caller
    ```

  * Mutex Other: Conditional Variables

    ![img](img/cdvb.png)

    ![img](img/cdvb2.png)

  * Monitor

    >**Semaphore problem: easy to <u>deadlock</u>**
    >
    >What is Dead lock?
    >
    >![img](img/ddlk.png)
    >
    >结论：
    >
    >​	**使用Semaphore要小心**

    Solution: High level abstraction

    >A monitor is **a collection of procedures, variables, and data structures** that are all **grouped together** in a special kind of module or package. Processes may call the procedures in a monitor whenever they want to, but they **can't directly access the monitor's internal data structures** from procedures declared outside the monitor.

    **Monitor Important Feature**

    > Only one process can be active *in a monitor* at any instant.

    Example - Pascal语言

    ```pascal
    monitor example
    	integer i;
    	condition c;
    	
    	procedure producer();
    	.
    	.
    	.
    	end;
    	
    	procedure consumer();
    	.
    	end;
    end monitor;
    ```

    Monitor - Pseudo Pascal

    ```pascal
    monitor ProducerConsumer
        condition full, empty;
        integer count;
        
        procedure insert(item: integer);
        begin
            if count = N then wait(full);
            insert item(item);
            count := count + 1;
            if count = 1 then signal(empty)
        end;
        
        function remove: integer;
        begin
            if count = 0 then wait(empty);
            remove = remove item;
            count := count − 1;
            if count = N − 1 then signal(full)
        end;
        
        count := 0;
    end monitor;
    
    procedure producer;
    begin
        while true do
        begin
            item = produce item;
            ProducerConsumer.insert(item)
        end
    end;
    
    procedure consumer;
    begin
        while true do
        begin
            item = ProducerConsumer.remove;
            consume item(item)
    	end
    end;
    ```

    Monitor - Java(synchronized)

    ```java
    public class ProducerConsumer {
        static final int N = 100; 						// constant giving the buffer size
        static producer p = new producer( ); 			 // instantiate a new producer thread
        static consumer c = new consumer( );			 // instantiate a new consumer thread
        static our monitor mon = new our monitor( );	  // instantiate a new monitor
        
        public static void main(String args[]) {
            p.start(); 									// start the producer thread
            c.start(); 									// start the consumer thread
        }
        
        static class producer extends Thread {
            public void run() {						// run method contains the thread code
                int item;
                while (true) { 						// producer loop
                    item = produce_item();
                    mon.insert(item);
                }
            }
            private int produce_item() { ... } 		// actually produce
        }
        
        static class consumer extends Thread {
            public void run() {					//run method contains the thread code
                int item;
                while (true) { 							// consumer loop
                    item = mon.remove();
                    consume_item (item);
                }
            }
            private void consume_item(int item) { ... }	// actually consume
        }
        
        static class our monitor { 						// this is a monitor
            private int buffer[] = new int[N];
            private int count = 0, lo = 0, hi = 0; 		 // counters and indices
            public synchronized void insert(int val) {
                if (count == N) go to sleep( ); 	 // if the buffer is full, go to sleep
                buffer [hi] = val; 					// inser t an item into the buffer
                hi = (hi + 1) % N; 					// slot to place next item in
                count = count + 1; 					// one more item in the buffer now
                if (count == 1) notify();			 // if consumer was sleeping, wake it up
        	}
            
            public synchronized int remove() {
                int val;
                if (count == 0) go to sleep(); 		// if the buffer is empty, go to sleep
                val = buffer [lo]; 					// fetch an item from the buffer
                lo = (lo + 1) % N; 					// slot to fetch next item from
                count = count − 1;					 // one few items in the buffer
                if (count == N − 1) notify(); 		// if producer was sleeping, wake it up
                return val;
            }
            private void go_to_sleep( ) { 
                try{wait( );} catch(Interr uptedException exc) {};}
            }
    }
    ```

  * Message Passing

    Monitor Disadvantage

    >Monitor之间也需要互斥，因为管程也是一个编程语言概念，**编译器必须要识别管程并用某种方式对其互斥进行安排**。但实际中，**如何让编译器知道哪些过程属于管程，哪些不属于？**
    >
    >如果一个分布式系统具有多个CPU，并且每个CPU拥有自己的私有内存，他们通过一个局域网相连，那么，这些原语将失效。这里的结论是：Semaphore太低级了，而管程在少数几种编程语言之外又无法使用，并且，这些原语均未提供机器间的信息交换方法，所以还需要其他的方法——Message Passing
    >
    >即，Monitor，Semaphore, Mutex解决的也都是一台电脑内部的Process, Thread互斥，没解决多台电脑合作时的互斥

    Message Passing System Call

    * 像Semaphore而不像Monitor，是系统调用而不是语言成分

      > Message Passing使用两条原语来实现**进程**间通信
      >
      > ```c
      > send(destination, &message);
      > receive(source, &message);
      > ```

    * Example

      ```c
      #define N 100							/*number of slots in the buffer*/
      void producer(void)
      {
          int item;
          message m; 							/*message buffer*/
          while (TRUE) {
              item = produce item( );			 /*generate something to put in buffer*/
              receive(consumer, &m); 			 /*wait for an empty to arrive*/
              build message(&m, item); 		 /*constr uct a message to send*/
              send(consumer, &m);				 /*send item to consumer*/
          }
      }
      void consumer(void)
      {
          int item, i;
          message m;
          for (i = 0; i < N; i++) send(producer, &m); 	  /*send N empties*/
          while (TRUE) {
              receive(producer, &m); 						/*get message containing item*/
              item = extract item(&m); 					/*extract item from message*/
              send(producer, &m); 						/*send back empty reply*/
              consume item(item); 						/*do something with the item*/
          }
      }
      ```

  * Barriers

    用于一组Process同步

    <img src="img/barrier.png" alt="img" style="zoom:67%;" />

## Scheduling

* 用于所有Process之间(之前的IPC是两个或几个Process之间)

### Problem

* **一些Process已经就绪，哪个该放到CPU上跑呢？**
* **调度为何不是时时刻刻发生，而是有间隔的发生？**

### When to Schedule?

* Process creation
* Process exit
* Process blocks on I/O
* I/O interrupt (**比如网络包到了，会发一个中断，运行接包的Process**)

### Scheduling Algorithm

>Category
>
>1. Preemptive vs Non-Preemptive
>
>2. Batch
>
>>批处理任务执行的好，就是
>>
>>* Throughput(吞吐量) -> Max
>>* Turnaround time -> Min
>>* CPU utilization(利用率) -> Max
>
>3. Interactive
>
>4. Real time
>
>Ps
>
>* Throughput - the number of processes that complete their execution per time unit
>* Turnaround time - the interval from submission to completion
>* Waiting time - amount of time a process has been waiting in the ready queue
>* Response time - amount of time it takes from when a request was submitted unitl the first response is produced, not output (for time-sharing environment)

#### FCFS Example

| Process | Burst Time |
| ------- | ---------- |
| P1      | 24         |
| P2      | 3          |
| P3      | 3          |

Arrive order: P1, P2, P3

解决思路：画Gannt Chart

<img src="img/fcfsgc.png" alt="img" style="zoom:50%;" />

Turnaround time for

> P1 = 24; P2 = 27; P3 = 30

Average turnaround time

> (24 + 27 + 30) / 3 = 27

#### SJF Example(Non Preemptive)

| Process | Arrival Time | Burst Time |
| ------- | ------------ | ---------- |
| P1      | 0.0          | 7          |
| P2      | 2.0          | 4          |
| P3      | 4.0          | 1          |
| P4      | 5.0          | 4          |

* 因为P1先到，到的时候P2, P3, P4还没来，所以先P1
* P1完事，此时，P2, P3, P4都来了，选最短的P3先来

Gannt Chart

<img src="img/sjfnpgc.png" alt="img" style="zoom:50%;" />

Average turnaround time

>[(7 - 0) + (8 - 4) + (12 - 2) + (16 - 5)] / 4 = 8

#### SJF Example(Preemptive)

| Process | Arrival Time | Burst Time |
| ------- | ------------ | ---------- |
| P1      | 0.0          | 7          |
| P2      | 2.0          | 4          |
| P3      | 4.0          | 1          |
| P4      | 5.0          | 4          |

* P1先来，2s后，P2来了，P1还剩5s，但是P2只用4s，所以先P2
* P2运行2s后，P3又来了，此时P1->5s, P2->2s, P3->1s, 所以P3
* 1s后，P3完事，P4来，此时P2->2s, P1->5s, P4->4s，所以先P2, 后P4，最后P1

Gannt Chart

<img src="img/sjfpgc.png" alt="img" style="zoom:50%;" />

Average turnaround time

>[(16 - 0) + (7 - 2) + (5 - 4) + (11 - 5) / 4]  = 7

### Interactive System Scheduling

#### Round Robin

<img src="img/rr.png" alt="img" style="zoom:67%;" />

#### Priority Scheduling

<img src="img/prioritysc.png" alt="img" style="zoom:67%;" />

#### Multiple Queue

<img src="img/mqueue.png" alt="img" style="zoom:67%;" />

#### Guaranteed Scheduling

若Process已经Ready，保证10s内能运行1s

> He gives you a "promise" and make sure he will keep it.
>
> * Example
>   * Promise: n processes in system, each will get 1/n CPU time
>   * Resource Reservation Scheduling Algorithms

**应用：花多少米，得多少时间**

#### Lottery Scheduling

* Give processes lottery tickets for various system resources, such as CPU time
* Whenever a scheduling decision has to be made, a lottery ticket is chosen at random, and **the process holding that ticket gets the resource**

#### Fair-Share Scheduling(FSS)

* 可以看做Guaranteed Scheduling的特例
* A和B交一样钱，但是A有10000000个Process，B就1个，则CPU全被A给抢了，那么就要保证A和B不管有几个Process，CPU时间都要平分

#### Real-Time Scheduling

* Hard Real-Time：必须在时限前搞定(**飞机计算，否则飞机炸**)
* Soft Real-Time：可以通融(**网络视频，卡了还行**)

### Schedulable

>可调度序列
>
>There's existed one scheduling sequence that make **every process** meet their deadline

### Policy Versus Mechanism

Separate the scheduling mechanism from the scheduling policy

* Key Idea: User can decide which scheduling algorithm is to use

* Scheduling algorithm is parameterized in some way, but the parameters can be filled in by user processes

  > **Exercise: 把Thread绑到CPU的一个核上(Linux)**

### Thread Scheduling

| Implementation in: | Kernel Space | User Space                      |
| ------------------ | ------------ | ------------------------------- |
| Cost               | Big          | Small                           |
| Other              | /            | 可实现应用特定Scheduler，效果好 |

## Classical IPC Problems

### Dining Philosophers Problem

* 哲学家：吃/思考
* 吃需要2个fork
* 一次拿一个fork
* 解决问题，还要防止deadlock

A wrong solution - may cause deadlock

```c
#define N 5 								/*number of philosophers*/
void philosopher(int i) 					 /*i: philosopher number, from 0 to 4*/
{
    while (TRUE) {
        think(); 							/*philosopher is thinking*/
        take fork(i); 						/*take left for k*/
        take fork((i+1) % N); 				 /*take right for k; % is modulo operator*/
        eat(); 								/*yum-yum, spaghetti*/
        put fork(i); 						/*put left for k back on the table*/
        put fork((i+1) % N); 				 /*put right for k back on the table*/
    }
}
```

**如果5个人都拿左边的fork，全部sleep -> deadlock**

解决：引入中控

<img src="img/eatf.png" alt="img" style="zoom:67%;" />

> IPC Design
>
> * Prevent deadlock
> * 尽量多并发

### Readers and writers Problem

<img src="img/raw.png" alt="img" style="zoom:80%;" />

**如果有读者，那么读者随便进，写者不能进，因为后来的读者，rc != 1，不会走down(&db)这句话**

### Sleeping Barber

* 理发店里有一位理发师、一把理发椅和n把供等候理发的顾客坐的椅子
* 如果没有顾客，理发师便在理发椅上睡觉
* 一个顾客到来时，它必须叫醒理发师
* 如果理发师正在理发时又有顾客来到，则如果有空椅子可坐，就坐下来等待，否则就离开

```c
#define CHAIRS 5               /* # chairs for waiting customers */
typedef int semaphore;         /* use your imagination */
semaphore customers = 0;       /* # of customers waiting for service */
semaphore barbers = 0;         /* # of barbers waiting for customers */
semaphore mutex = 1;           /* for mutual exclusion */

//critical region
int waiting = 0;               /* customers are waiting (not being cut) */
 
void barber(void){
    white (TRUE) {
        /**
        * 没有顾客，睡觉；
        * 有顾客，down完还要接着剪
        */
        down(&customers);      /* go to sleep if # of customers is 0 */
        
        /* 若能执行到这儿，waiting肯定被加了 */
        down(&mutex);          /* acquire access to 'waiting' */
        waiting = waiting − 1; /* decrement count of waiting customers */
        up(&barbers);          /* one barber is now ready to cut hair */
        up(&mutex);            /* release 'waiting' */
        
        cut_hair();            /* cut hair (outside critical region) */
    }
}
 
void customer(void){
    down(&mutex);              /* enter critical region */
    if (waiting < CHAIRS) {    /* if there are no free chairs, leave */
        
        //抢椅子
        waiting = waiting + 1; /* increment count of waiting customers */
        
        up(&customers);        /* wake up barber if necessary */
        up(&mutex);            /* release access to 'waiting' */
        down(&barbers);        /* go to sleep if # of free barbers is 0 */
        get_haircut();         /* be seated and be serviced */
    } else {
        //椅子满，走人
        up(&mutex);            /* shop is full; do not wait */
    }
}
```

### Driver and  Seller

原则

* IPC -> 同步，互斥，同步互斥
* 同步：semaphore初值为0，需要等别人的进程要p操作，被别人等的进程要v操作
* 互斥：semaphore初值为进程数量

```c
//司机和售票员问题
Semaphore driver = 0, door = 0;

/*
 司机的活动：启动车辆，正常行车，到站停车
 售票员的活动：关车门，售票，开车门
 	注意：当发车时间到，售票员关门后司机才能开车，售票员开始售票；
 	到站时，司机停车后，售票员才能打开车门
*/
Driver(){
    /* 要等售票员关门后才能开车，等别人 */
    P(drive);
    
    Drive();
    CarMove();
    Stop();
    
    /* 停车后允许售票员开门 */
    V(door);
}

Ticket_Seller(){
    DoorClose();
    
    /* 关门后允许司机开车 */
    V(drive);
    SellTicket();
    
    /* 要等司机停车后才能开门，等别人 */
    P(door);
    DoorOpen();
}
```

