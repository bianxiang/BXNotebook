POSIX thread (pthread) libraries

http://www.yolinux.com/TUTORIALS/LinuxTutorialPosixThreads.html

[toc]

The POSIX thread libraries are a standards based thread API for C/C++. It allows one to spawn a new concurrent process flow. Threads require less overhead than "forking" or spawning a new process because the system does not initialize a new system virtual memory space and environment for the process. While most effective on a multiprocessor system, gains are also found on uniprocessor systems which exploit latency in I/O and other system functions which may halt process execution. (One thread may execute while another is waiting for I/O or some other system latency.) All threads within a process share the same address space. A thread is spawned by defining a function and its arguments which will be processed in the thread. The purpose of using the POSIX thread library in your software is to execute software faster.

## 线程基础

线程的操作包括线程额创建、终止、同步（joins, blocking）、scheduling、数据管理和进程交互。

A thread does not maintain a list of created threads, nor does it know the thread that created it.

进程中所哟线程共享相同的地址空间。

进程中的线程共享：

- Process instructions
- Most data
- open files (descriptors)
- signals and signal handlers
- 当前工作目录
- User and group id

每个线程独有的：

- 线程ID
- set of registers, stack pointer
- stack for local variables, return addresses
- signal mask
- priority
- Return value: errno

pthread函数返回0表示成功。

## 线程创建和终止

Example: pthread1.c

```cpp
    #include <stdio.h>
    #include <stdlib.h>
    #include <pthread.h>

    void *print_message_function( void *ptr );

    main()
    {
         pthread_t thread1, thread2;
         const char *message1 = "Thread 1";
         const char *message2 = "Thread 2";
         int iret1, iret2;

        /* 创建独立的线程 */
         iret1 = pthread_create(&thread1, NULL, print_message_function, (void*) message1);
         if(iret1)
         {
             fprintf(stderr, "Error - pthread_create() return code: %d\n",iret1);
             exit(EXIT_FAILURE);
         }

         iret2 = pthread_create(&thread2, NULL, print_message_function, (void*) message2);
         if(iret2)
         {
             fprintf(stderr, "Error - pthread_create() return code: %d\n",iret2);
             exit(EXIT_FAILURE);
         }

         printf("pthread_create() for thread 1 returns: %d\n", iret1);
         printf("pthread_create() for thread 2 returns: %d\n", iret2);

         /* 等待线程结束 */
         pthread_join( thread1, NULL);
         pthread_join( thread2, NULL);

         exit(EXIT_SUCCESS);
    }

    void *print_message_function( void *ptr )
    {
         char *message;
         message = (char *) ptr;
         printf("%s \n", message);
    }
```

编译：

- C编译器：`cc -pthread pthread1.c (or cc -lpthread pthread1.c)`
- `C++`编译器：`g++ -pthread pthread1.c (or g++ -lpthread pthread1.c)`

The GNU compiler now has the command line option "-pthread" while older versions of the compiler specify the pthread library explicitly with "`-lpthread`".

运行：`./a.out`

结果：

```
Thread 1
Thread 2
Thread 1 returns: 0
Thread 2 returns: 0
```

细节：

以下情况导致线程终止：显示调用`pthread_exit()`，方法返回，或调用`exit()`导致进程终止。


[pthread_create](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_create)：创建新线程：

```cpp
    int pthread_create(pthread_t * thread,
    	const pthread_attr_t * attr,
        void * (*start_routine)(void *),
        void *arg);
```

参数：

- `thread`：返回线程ID(unsigned long int defined in bits/pthreadtypes.h)
- `attr`：Set to NULL if default thread attributes are used. (else define members of the struct `pthread_attr_t` defined in bits/pthreadtypes.h) 属性包括：
	- detached state (joinable? Default: `PTHREAD_CREATE_JOINABLE`. Other option: `PTHREAD_CREATE_DETACHED`)
    - scheduling policy (real-time? `PTHREAD_INHERIT_SCHED`,`PTHREAD_EXPLICIT_SCHED`, `SCHED_OTHER`)
    - scheduling parameter
    - inheritsched attribute (Default: `PTHREAD_EXPLICIT_SCHED` Inherit from parent thread: `PTHREAD_INHERIT_SCHED`)
    - scope (Kernel threads: `PTHREAD_SCOPE_SYSTEM` User threads: `PTHREAD_SCOPE_PROCESS` Pick one or the other not both.)
    - guard size
    - stack address (See unistd.h and bits/posix_opt.h `_POSIX_THREAD_ATTR_STACKADDR`)
    - stack size (default minimum `PTHREAD_STACK_SIZE` set in pthread.h),
- `void * (*start_routine)`：pointer to the function to be threaded. 函数只有一个参数：指向void的指针。
- `*arg`：传给函数的参数。如想指定多个参数，可以构造一个结构体。


函数[pthread_join](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_join)：等待另一个线程结束

```
    int pthread_join(pthread_t th, void **thread_return);
```

参数：

- `th` - thread suspended until the thread identified by `th` terminates, either by calling `pthread_exit()` or by being cancelled.
- `thread_return` - If `thread_return` is not NULL, the return value of `th` is stored in the location pointed to by `thread_return`.


[pthread_exit](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_exit)：terminate the calling thread

```cpp
    void pthread_exit(void *retval);
```
参数：

- `retval` - Return value of `pthread_exit()`.

This routine kills the thread. The `pthread_exit()` function never returns. If the thread is not detached, the thread id and return value may be examined from another thread by using `pthread_join()`.

Note: the return pointer `*retval`, must not be of local scope otherwise it would cease to exist once the thread terminates.

[C++ pitfalls]: The above sample program will compile with the GNU C and C++ compiler g++. The following function pointer representation below will work for C but not **C++**. Note the subtle differences and avoid the pitfall below:

```cpp
    void print_message_function( void *ptr );
    ...
    ...
    iret1 = pthread_create(&thread1, NULL, (void*)&print_message_function, (void*) message1);
    ...
    ...
```

## 线程同步

线程库提供三种同步机制：

- mutexes：互斥排它锁：Block access to variables by other threads. This enforces exclusive access by a thread to a variable or set of variables.
- joins：Make a thread wait till others are complete (terminated).
- 条件变量：数据类型`pthread_cond_t`

### 互斥量（Mutexes）

Mutexes are used to prevent data inconsistencies due to operations by multiple threads upon the same memory area performed at the same time or to prevent race conditions where an order of operation upon the memory is expected. Mutexes are used for serializing shared resources such as memory. Anytime a global resource is accessed by more than one thread the resource should have a Mutex associated with it. One can apply a mutex to protect a segment of memory ("critical region") from other threads. Mutexes can be applied only to threads in a single process and do not work between processes as do semaphores.

Example threaded function:

不带互斥量：

```cpp
    int counter=0;

    /* Function C */
    void functionC()
    {
       counter++
    }
```

带互斥量：

```cpp
    /* 注意：变量和互斥量的作用域相同 */
    pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;
    int counter=0;

    /* Function C */
    void functionC()
    {
       pthread_mutex_lock( &mutex1 );
       counter++
       pthread_mutex_unlock( &mutex1 );
    }
```

Code listing: mutex1.c：

```cpp
    #include <stdio.h>
    #include <stdlib.h>
    #include <pthread.h>

    void *functionC();
    pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;
    int counter = 0;

    main()
    {
       int rc1, rc2;
       pthread_t thread1, thread2;

       if( (rc1=pthread_create( &thread1, NULL, &functionC, NULL)) )
       {
          printf("Thread creation failed: %d\n", rc1);
       }

       if( (rc2=pthread_create( &thread2, NULL, &functionC, NULL)) )
       {
          printf("Thread creation failed: %d\n", rc2);
       }

       /* 让主线程等剩下两个线程，否则主线程调退出，进程结束，那两个线程也就结束了 */
       pthread_join( thread1, NULL);
       pthread_join( thread2, NULL);

       exit(EXIT_SUCCESS);
    }

    void *functionC()
    {
       pthread_mutex_lock( &mutex1 );
       counter++;
       printf("Counter value: %d\n",counter);
       pthread_mutex_unlock( &mutex1 );
    }
```

编译：`cc -pthread mutex1.c`（或`cc -lpthread mutex1.c` for older versions of the GNU compiler which explicitly reference the library）

运行：`./a.out`。

When a mutex lock is attempted against a mutex which is held by another thread, the thread is blocked until the mutex is unlocked. When a thread terminates, the mutex does not unless explicitly unlocked. Nothing happens by default.

Man Pages:

- [pthread_mutex_lock()](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_mutex_lock) - acquire a lock on the specified mutex variable. If the mutex is already locked by another thread, this call will block the calling thread until the mutex is unlocked.
- [pthread_mutex_unlock()](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_mutex_unlock) - unlock a mutex variable. An error is returned if mutex is already unlocked or owned by another thread.
- [pthread_mutex_trylock()](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_mutex_trylock) - attempt to lock a mutex or will return error code if busy. 用于防止死锁。

### Joins

A join is performed when one wants to wait for a thread to finish. A thread calling routine may launch multiple threads then wait for them to finish to get the results. One waits for the completion of the threads with a join.

Sample code: join1.c

```cpp

    #include <stdio.h>
    #include <pthread.h>

    #define NTHREADS 10
    void *thread_function(void *);
    pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;
    int counter = 0;

    main()
    {
       pthread_t thread_id[NTHREADS];
       int i, j;

       for(i=0; i < NTHREADS; i++)
       {
          pthread_create(&thread_id[i], NULL, thread_function, NULL);
       }

       for(j=0; j < NTHREADS; j++)
       {
          pthread_join(thread_id[j], NULL);
       }

       /* 此时所有线程都已完成 */
       printf("Final counter value: %d\n", counter);
    }

    void *thread_function(void *dummyPtr)
    {
       printf("Thread number %ld\n", pthread_self());
       pthread_mutex_lock( &mutex1 );
       counter++;
       pthread_mutex_unlock( &mutex1 );
    }
```

Compile: `cc -pthread join1.c` (or `cc -lpthread join1.c` for older versions of the GNU compiler which explicitly reference the library)

Run: `./a.out`

Man Pages:

- [pthread_create()](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_create) - create a new thread
- [pthread_join()](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_join) - wait for termination of another thread
- [pthread_self()](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_self) - return identifier of current thread

### Condition Variables

A condition variable is a variable of type `pthread_cond_t` and is used with the appropriate functions for waiting and later, process continuation. The condition variable mechanism allows threads to suspend execution and relinquish the processor until some condition is true. A condition variable must always be associated with a **mutex** to avoid a race condition created by one thread preparing to wait and another thread which may signal the condition before the first thread actually waits on it resulting in a deadlock. The thread will be perpetually waiting for a signal that is never sent. Any mutex can be used, there is no explicit link between the mutex and the condition variable.

Man pages of functions used in conjunction with the condition variable:

- 创建/销毁：
	- [pthread_cond_init](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_cond_init)
	- `pthread_cond_t cond = PTHREAD_COND_INITIALIZER;`
    - [pthread_cond_destroy](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_cond_destroy)
- 等待条件：
	- [pthread_cond_wait](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_cond_wait) - unlocks the mutex and waits for the condition variable cond to be signaled.
    - [pthread_cond_timedwait](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_cond_timedwait) - place limit on how long it will block.
- 根据条件唤醒线程：
	- [pthread_cond_signal](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_cond_signal) - restarts one of the threads that are waiting on the condition variable cond.
    - [pthread_cond_broadcast](http://man.yolinux.com/cgi-bin/man2html?cgi_command=pthread_cond_broadcast) - wake up all threads blocked by the specified condition variable.

Example code: cond1.c

```cpp
    #include <stdio.h>
    #include <stdlib.h>
    #include <pthread.h>

    pthread_mutex_t count_mutex = PTHREAD_MUTEX_INITIALIZER;
    pthread_cond_t condition_var = PTHREAD_COND_INITIALIZER;

    void *functionCount1();
    void *functionCount2();
    int count = 0;
    #define COUNT_DONE  10
    #define COUNT_HALT1  3
    #define COUNT_HALT2  6

    main()
    {
       pthread_t thread1, thread2;

       pthread_create( &thread1, NULL, &functionCount1, NULL);
       pthread_create( &thread2, NULL, &functionCount2, NULL);

       pthread_join( thread1, NULL);
       pthread_join( thread2, NULL);

       printf("Final count: %d\n",count);

       exit(EXIT_SUCCESS);
    }

    // Write numbers 1-3 and 8-10 as permitted by functionCount2()
    void *functionCount1()
    {
       for(;;)
       {
          // Lock mutex and then wait for signal to relase mutex
          pthread_mutex_lock( &count_mutex );

          // Wait while functionCount2() operates on count
          // mutex unlocked if condition varialbe in functionCount2() signaled.
          pthread_cond_wait( &condition_var, &count_mutex );
          count++;
          printf("Counter value functionCount1: %d\n",count);

          pthread_mutex_unlock( &count_mutex );

          if(count >= COUNT_DONE) return(NULL);
        }
    }

    // Write numbers 4-7
    void *functionCount2()
    {
        for(;;)
        {
           pthread_mutex_lock( &count_mutex );

           if( count < COUNT_HALT1 || count > COUNT_HALT2 )
           {
              // Condition of if statement has been met.
              // Signal to free waiting thread by freeing the mutex.
              // Note: functionCount1() is now permitted to modify "count".
              pthread_cond_signal( &condition_var );
           }
           else
           {
              count++;
              printf("Counter value functionCount2: %d\n",count);
           }

           pthread_mutex_unlock( &count_mutex );

           if(count >= COUNT_DONE) return(NULL);
        }

    }
```

Compile: `cc -pthread cond1.c` (or `cc -lpthread cond1.c` for older versions of the GNU compiler which explicitly reference the library)

Run: `./a.out`

Results:

```
Counter value functionCount1: 1
Counter value functionCount1: 2
Counter value functionCount1: 3
Counter value functionCount2: 4
Counter value functionCount2: 5
Counter value functionCount2: 6
Counter value functionCount2: 7
Counter value functionCount1: 8
Counter value functionCount1: 9
Counter value functionCount1: 10
Final count: 10
```

Note that `functionCount1()` was halted while count was between the values `COUNT_HALT1` and `COUNT_HALT2`. The only thing that has been ensures is that `functionCount2` will increment the count between the values `COUNT_HALT1` and `COUNT_HALT2`. Everything else is random.

The logic conditions (the "if" and "while" statements) must be chosen to insure that the "signal" is executed if the "wait" is ever processed. Poor software logic can also lead to a deadlock condition.

Note: Race conditions abound with this example because count is used as the condition and can't be locked in the while statement without causing deadlock.

## Thread Scheduling

When this option is enabled, each thread may have its own scheduling properties. Scheduling attributes may be specified:

- during thread creation
- by dynamically by changing the attributes of a thread already created
- by defining the effect of a mutex on the thread's scheduling when creating a mutex
- by dynamically changing the scheduling of a thread during synchronization operations.

The threads library provides default values that are sufficient for most cases.

## Thread Pitfalls

- Thread safe code: In C, local variables are dynamically allocated on the stack. Therefore, any function that does not use static data or other shared resources is thread-safe. Thread-unsafe functions may be used by only one thread at a time in a program and the uniqueness of the thread must be ensured. Many non-reentrant functions return a pointer to static data. This can be avoided by returning dynamically allocated data or using caller-provided storage. An example of a non-thread safe function is strtok which is also not re-entrant. The "thread safe" version is the re-entrant version `strtok_r`.
- Mutex Deadlock: This condition occurs when a mutex is applied but then not "unlocked". This causes program execution to halt indefinitely. It can also be caused by poor application of mutexes or joins. Be careful when applying two or more mutexes to a section of code. If the first `pthread_mutex_lock` is applied and the second `pthread_mutex_lock` fails due to another thread applying a mutex, the first mutex may eventually lock all other threads from accessing data including the thread which holds the second mutex. The threads may wait indefinitely for the resource to become free causing a deadlock. It is best to test and if failure occurs, free the resources and stall before retrying.

```cpp
 ...
    pthread_mutex_lock(&mutex_1);
    while ( pthread_mutex_trylock(&mutex_2) )  /* Test if already locked   */
    {
       pthread_mutex_unlock(&mutex_1);  /* Free resource to avoid deadlock */
       ...
       /* stall here   */
       ...
       pthread_mutex_lock(&mutex_1);
    }
    count++;
    pthread_mutex_unlock(&mutex_1);
    pthread_mutex_unlock(&mutex_2);
    ...
```

The order of applying the mutex is also important. The following code segment illustrates a potential for deadlock:

```cpp
 void *function1()
    {
       ...
       pthread_mutex_lock(&lock1);           // Execution step 1
       pthread_mutex_lock(&lock2);           // Execution step 3 DEADLOCK!!!
       ...
       ...
       pthread_mutex_lock(&lock2);
       pthread_mutex_lock(&lock1);
       ...
    } 

    void *function2()
    {
       ...
       pthread_mutex_lock(&lock2);           // Execution step 2
       pthread_mutex_lock(&lock1);
       ...
       ...
       pthread_mutex_lock(&lock1);
       pthread_mutex_lock(&lock2);
       ...
    } 
  
    main()
    {
       ...
       pthread_create(&thread1, NULL, function1, NULL);
       pthread_create(&thread2, NULL, function2, NULL);
       ...
    }
```

If function1 acquires the first mutex and function2 acquires the second, all resources are tied up and locked.

- Condition Variable Deadlock: The logic conditions (the "if" and "while" statements) must be chosen to insure that the "signal" is executed if the "wait" is ever processed.

## （未）Thread Debugging

## （未）Thread Man Pages

## （未）Links

