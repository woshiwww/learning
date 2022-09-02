**临界区**就是一个更新共享资源的代码段，一次只允许一个线程执行改代码段。

这一章使用POSIX线程库(也经常称为Pthreads线程库)来实现共享内存的访问。

###  一 、进程、线程和Pthreads

进程：正在运行的一个程序的实例。不同进程的内存块是私有的，其他进程不能直接访问，除非操作系统进行干涉。

线程：线程共享进程的代码段、数据段和打开的文件资源，拥有自己独立的栈和程序计数器。

Pthreads不是一个编程语言，而是一个C语言编写的共享库。它的API只有在支持POSIX的系统上才有效，比如Linux、Mac OS等。

### 二、进程的hello world

注意：使用pthreads库在编译时需要使用链接器-lpthread。

`gcc -g -Wall -o a.out main.c -lpthread`

使用多线程来输出hello world。

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

// 创建一个全局变量，代表总的线程数。这个全局变量是被所有的线程共享，而在函数中的局部变量，则被每个线程独享。
int thread_count;
void * HelloWorld(void * rank);

int main(int argc, char * argv[])
{
    long thread;
    pthread_t * ThreadHandles;

    // 从命令行获得线程的数量
    //thread_count = strtol(argv[1], NULL, 10);
    thread_count = 5;
    // 为每个线程创建空间
    ThreadHandles = (pthread_t *) malloc(sizeof(pthread_t)*thread_count);

    for(thread = 0; thread <thread_count; thread++)
    {
        pthread_create(&ThreadHandles[thread],NULL,HelloWorld,(void *) thread);
    }
    printf("Hello from the main thread\n");
    for(thread = 0; thread <thread_count; thread++)
    {
        pthread_join(ThreadHandles[thread], NULL);
    }
    free(ThreadHandles);
    return 0;

}
void * HelloWorld(void * rank)
{
    long my_rank = (long) rank;
    printf("Hello from thread %ld of %d\n",my_rank, thread_count);
    return NULL;
}
```

其中需要使用两个函数，分别为pthread_create()和pthread_join()

```c
int pthread_create(pthread_t * thread_p, const pthread_attr_t * attr_p, void * (*func)(void *), void *);
功能：
    创建线程，并运行相关的线程函数。
参数：
	第一个参数是一个指针，指向对应的pthread_t对象。
	第二个参数通常设置为NULL。
	第三个参数指向将要运行的函数。
	第四个参数指向将要运行函数的参数，都是指针的形式。
返回值：
    0表示成功，-1表示失败。

int pthread_join(pthread_t tid,  void **status);
功能：
	阻塞线程，直到指定线程终止，当这个函数返回之后，应用程序可回收与已终止线程关联的任何数据存储空间。
参数：
    第一个参数是需要等待的线程,指定的线程必须位于当前的进程中，而且不得是分离线程。
    第二个参数是线程所执行的函数返回值（返回值地址需要保证有效），其中status可以为NULL
返回值：
    0表示成功
```

### 三、使用多线程实现矩阵的向量乘法

A是一个m×n的矩阵，x是一个n维的列向量，y是一个m维的列向量。A×x = y。

串行计算的程序如下所示：

```c
for(int i = 0; i < m; i++)
{
    y[i] = 0.0;
    for(int j = 0; j < n; j++){
        y[i] += A[i][j] * x[j];
    }
}
```

串行程序需要挨个计算，通过将循环的外层循环进行分块，可以并行的计算。假设m=6，此时有三个线程，可以给每个线程分配两行。**如果线程0负责处理第0行和第1行，则线程0需要访问矩阵A中的第一行和第二行，以及列向量x中的每个元素。这意味着最低限度下，需要将列向量x设为全局变量。**

如何确定哪个线程负责处理几行数据呢？通过数学的方法就可以很好的确定

t为线程序号，m为矩阵的行数，线程i负责的范围为

第一行：i × m/n

最后一行：（i+1）× m/n - 1     因为是从0行开始的，所以需要减1。

假设A、m、n、x、y都设为全局变量，程序如下。

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

int A[3][3] = {{1,2,3}, {4,5,6},{7,8,9}};
int x[3] = {1,1,1};
int y[3];
int m = 3;
int n = 3;
int thread_count = 3;

void *cacu_matrix(void * rank);
void print_matrix(int *y);

int main()
{
    pthread_t * thread_space;
    thread_space = (pthread_t *)malloc(thread_count * sizeof(pthread_t));
    for(int i = 0; i<thread_count; i++)
    {
        pthread_create(&thread_space[i],NULL,cacu_matrix,(void *)(long)i);
    }
    for(int i = 0; i<thread_count; i++)
    {
        pthread_join(thread_space[i],NULL);
    }
    print_matrix(y);
    return 0;
}

//计算矩阵，每个线程都有属于自己的那部分数据。
void *cacu_matrix(void *rank)
{
    long thread_rank = (long) rank;
    for (int i = thread_rank * (m/thread_count); i < (thread_rank+1) * (m/thread_count);i++)
    {
        y[i] = 0;
        for(int j =0; j < n; j++)
        {
            y[i] += A[i][j] * x[j];
        }
    }
    return NULL;
}

//打印计算后的值
void print_matrix(int *y)
{
    for(int i = 0; i < m; i++)
    {
        printf("%d\n",y[i]);
    }
}
```

在这个程序中，每个线程负责自己的数据，除了列向量y之外，所有的共享变量没有被改写。没有涉及几个线程共同处理同一数据的情况。

### 四、互斥量

处于忙等待的线程会持续的使用CPU，和串行程序相比反而会增加程序的运行时间，所以忙等待不是最理想的限制临界区的方法，但是忙等待能保证线程的执行顺序。为了限制每次只有一个线程访问临界区，还有两种更好的方法：**互斥量和信号量**。

#### 互斥量

Pthread为互斥量提供了一个特殊类型：pthread_mutex_t，在使用这个类型之前，必须由系统对其进行初始化。使用如下函数：

```c
int pthread_mutex_init(pthread_mutex_t * mutex_p,const pthread_mutexattr_t * attr_p);
功能：
    初始化互斥量
参数：
    第一个参数：互斥量的地址
    第二个参数：通常设置为NULL
返回值：
    返回0表示运行成功，其他则表示失败
//########################################################################################
当一个pthread程序使用完互斥量之后，应该调用函数：
int pthread_mutex_destroy(pthread_mutex_t * mutex_p);
功能：
    销毁一个指定的互斥量
参数：
    互斥量的地址
返回值：
    返回0表示运行成功，其他则表示失败
    
//########################################################################################
int pthread_mutex_lock(pthread_mutex_t *mutex);
功能：
	对互斥锁上锁，获得临界区的访问权，若互斥锁已经上锁，则调用者阻塞，直到互斥锁解锁后再上锁。
参数：
	mutex：互斥锁地址。
返回值：
	成功：0
	失败：非 0 错误码
   //########################################################################################
int pthread_mutex_unlock(pthread_mutex_t *mutex);
功能：
	线程退出临界区时，对指定的互斥锁解锁。
参数：
	mutex：互斥锁地址。
返回值：
	成功：0
	失败：非0错误码
```

**使用互斥量时，线程访问临界区的顺序时随机的，由系统负责分配。**

#### 生产者-消费者同步和信号量

信号量是一种特殊类型，为unsigned int。可以赋值为0，1，2，......。大多数情况下，只赋值0，1，此时称为二元信号量。可以认为0时是上了锁的互斥量，1是未上锁的互斥量。要将一个信号量用作互斥量，首先需要先将信号量初始化为1，即处于开锁状态。如果要使用信号量，需要#include<semaphore.h>,信号量的数据类型为结构sem_t数。

信号量和互斥量的区别：

信号量是没有个体拥有权的，主线程将所有的信号量初始化为0，表示加锁，其他线程都能对任何信号量调用sem_post和sem_wait函数。

在使用信号量时也要对其进行初始化：

```c
int sem_init(sem_t * semaphore_p,int shared,unsigned initial_val);
功能：
    创建信号量，并为信号量赋初始值
参数：
    semaphore_p：指向信号量的指针
    shared：不为０时信号量在进程间共享，否则只能为当前进程的所有线程共享
    initial_val：信号量的初始值
返回值：
    0成功，-1失败

int sem_destroy(sem_t  *sem);//清理信号量占有的资源，当调用该函数，而有线程等待此信号量时，将会返回错信息。

int sem_wait(sem_t  *sem); //以阻塞的方式等待信号量，当信号量的值大于零时，执行该函数信号量减一，当信号量                            //为零时，调用该函数的线程将会阻塞。
int sem_post(sem_t  *sem);  // 对信号量+1

```

一个线程需要等待另一个线程执行某种操作的同步方式，有时称为生产者-消费者同步模型。
