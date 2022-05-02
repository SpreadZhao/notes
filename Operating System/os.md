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

##### Process Termination

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

##### Process Switching

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

         >**小练习：gcc用C语言嵌入汇编语言**

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
  
    

