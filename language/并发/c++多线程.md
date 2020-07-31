

##线程内存模型
###C/C++并发编程（1）—— 并发/并行、多线程内存模型
https://www.jianshu.com/p/298296e9a887
为了在性能和易编程性之间找到平衡，C++11提出了“sequential consistency for data race free programs”内存模型，即没有数据竞跑（data race）的程序符合顺序一致性。数据竞跑是指多个线程在没有同步的情况下去访问相同的内存位置[5]。所以，在C11/C++11后，我们只要对多线程之间需要同步的变量和操作，使用正确的同步原语进行同步，就能保证程序的执行符合顺序一致性。编译器、多核CPU能保证其优化措施不会破坏顺序一致性。

另外，C11/C++11标准还明确了“内存位置”的定义。

一个内存位置要么是标量，要么是一组紧邻的具有非零长度的位域。
两个线程可以互不干扰地对不同的内存位置进行读写操作

比如有如下的结构体：

struct
{
int a : 17;
int b : 15;
} x;
两个线程分别读写a和b，是否会互相干扰呢？毕竟CPU是按32/64位来取操作数的，而不是按17/15位来的。C11/C++11之前这样的操作是未定义的，按C11/C++标准规定a和b则属于同一个内存位置。两个线程分别对a、b进行读写操作是会相互干扰的，需要进行同步。或者将a、b分割成两个内存位置：

struct
{
int a : 17;    // 内存位置1
int : 0;
int b : 15;    // 内存位置2
} x;
这样编译器会自动自行内存对齐，保证两个线程分别读写a、b互不干扰。


https://blog.csdn.net/qq_40273354/article/details/78494504

###线程局部存储(thread_local)
https://www.jianshu.com/p/8df45004bbcb

线程局部存储在其它语言中都是以库的形式提供的(库函数或类)。但在C++11中以关键字的形式，做为一种存储类型出现，由此可见C++11对线程局部存储的重视。
 
thread_local修饰的变量具有如下特性:

变量在线程创建时生成(不同编译器实现略有差异，但在线程内变量第一次使用前必然已构造完毕)。
线程结束时被销毁(析构，利用析构特性，thread_local变量可以感知线程销毁事件)。
每个线程都拥有其自己的变量副本。
thread_local可以和static或extern联合使用，这将会影响变量的链接属性。


下面代码演示了thread_local变量在线程中的生命周期

// thread_local.cpp

class A {
public:
  A() {
    std::cout << std::this_thread::get_id()
              << " " << __FUNCTION__
              << "(" << (void *)this << ")"
              << std::endl;
  }

  ~A() {
    std::cout << std::this_thread::get_id()
              << " " << __FUNCTION__
              << "(" << (void *)this << ")"
              << std::endl;
  }

  // 线程中，第一次使用前初始化
  void doSth() {
  }
};

thread_local A a;

int main() {
  a.doSth();
  std::thread t([]() {
    std::cout << "Thread: "
              << std::this_thread::get_id()
              << " entered" << std::endl;
    a.doSth();
  });

  t.join();

  return 0;
}
运行该程序

$> g++ -std=c++11 -o debug/tls.out ./thread_local.cpp
$> ./debug/tls.out
01 A(0xc00720)
Thread: 02 entered
02 A(0xc02ee0)
02 ~A(0xc02ee0)
01 ~A(0xc00720)
$>
变量a在main线程和t线程中分别保留了一份副本，以下时序图表明了两份副本的生命周期。


https://blog.csdn.net/y396397735/article/details/81271339
https://blog.csdn.net/woshi_caibi/article/details/71124390

##线程同步
###条件变量 condition_variable
http://www.cplusplus.com/reference/condition_variable/condition_variable/
http://www.cplusplus.com/reference/condition_variable/condition_variable/notify_all/

https://www.cnblogs.com/huty/p/8516997.html


####std::condition_variable_any::wait_for
http://www.cplusplus.com/reference/condition_variable/condition_variable_any/

Wait for timeout or until notified
The execution of the current thread (which shall currently lock lck) is blocked during rel_time, or until notified (if the latter happens first).

// condition_variable_any::wait_for example

std::condition_variable_any cv;

int value;

void read_value() {
  std::cin >> value;
  cv.notify_one();
}

int main ()
{
  std::cout << "Please, enter an integer (I'll be printing dots): ";
  std::thread th (read_value);

  std::mutex mtx;
  mtx.lock();
  while (cv.wait_for(mtx,std::chrono::seconds(1))==std::cv_status::timeout) {
    std::cout << '.';
  }
  std::cout << "You entered: " << value << '\n';
  mtx.unlock();

  th.join();

  return 0;
}
 Edit & Run


Possible output:

Please, enter an integer (I'll be priniting dots): .....20
You entered: 20


###linux 多线程的线程控制和线程通信
https://blog.csdn.net/chenjiayi_yun/article/details/18059665
1、Linux 线程概念
     进程与线程之间是有区别的，不过Linux内核只提供了轻量进程的支持，而其所谓的线程本质上在内核里仍然是进程。

     进程是资源分配的单位，同一进程中的多个线程共享该进程的资源。Linux中的线程只是在被创建时clone了父进程的资源，因此clone出来的进程表现为线程，只是它有共享父进程资源的特性。

   程序与线程库相链接即可支持Linux平台上的多线程，在程序中需包含头文件pthread. h，在编译链接时使用命令： 
gcc -D -REENTRANT -lpthread xxx. c

其中-REENTRANT宏使得相关库函数(如stdio.h、errno.h中函数) 是可重入的、线程安全的(thread-safe)，-lpthread则意味着链接库目录下的libpthread.a或libpthread.so文件。

流行的线程模型有LinuxThreads 和 NPTL。使用线程库需要2.0以上版本的Linux内核,及相应版本的C库(libc 或glibc )。

参考：http://www.ibm.com/developerworks/cn/linux/l-threading.html

2、线程控制 
（1）线程创建 

进程被创建时，系统会为其创建一个主线程，而要在进程中创建新的线程，则可以调用pthread_create： 
pthread_create(pthread_t *thread, const pthread_attr_t *attr, void * (start_routine)(void*), void *arg);


start_routine为新线程的入口函数，arg为传递给start_routine的参数。 

每个线程都有自己的线程ID，以便在进程内区分。线程ID在pthread_create调用时回返给创建线程的调用者；一个线程也可以在创建后使用pthread_self()调用获取自己的线程ID： 

pthread_self (void) ;


（2）线程退出 

线程的退出方式： 

1）执行完成后隐式退出
2）由线程本身显示调用pthread_exit 函数退出
pthread_exit (void * retval) ;

3）被其他线程用pthread_cance函数终止

pthread_cance (pthread_t thread) ;


在某线程中调用此函数，可以终止由参数thread 指定的线程。 

如果一个线程要等待另一个线程的终止，可以使用pthread_join函数，该函数的作用是调用pthread_join的线程将被挂起直到线程ID为参数thread的线程终止： 

pthread_join (pthread_t thread, void** threadreturn);

3、线程通信 
（1）线程互斥 

互斥意味着“排它”，即两个线程不能同时进入被互斥保护的代码。Linux下可以通过pthread_mutex_t 定义互斥体机制完成多线程的互斥操作，该机制的作用是对某个需要互斥的部分，在进入时先得到互斥体，如果没有得到互斥体，表明互斥部分被其它线程拥有，此时欲获取互斥体的线程阻塞，直到拥有该互斥体的线程完成互斥部分的操作为止。 

下面的代码实现了对共享全局变量x1 用互斥体mutex 进行保护的目的： 
int x1; // 进程中的全局变量 
pthread_mutex_t mutex; 
pthread_mutex_init(&mutex, NULL); //按缺省的属性初始化互斥体变量mutex 
pthread_mutex_lock(&mutex); // 给互斥体变量加锁 
… //对变量x1 的操作 
phtread_mutex_unlock(&mutex); // 给互斥体变量解除锁


（2）线程同步 

同步就是线程等待某个事件的发生。只有当等待的事件发生线程才继续执行，否则线程挂起并放弃处理器。当多个线程协作时，相互作用的任务必须在一定的条件下同步。 
1）条件变量

Linux下的C语言编程有多种线程同步机制，最典型的是条件变量(condition variable)。

pthread_cond_init用来创建一个条件变量，其函数原型为： 

pthread_cond_init (pthread_cond_t *cond, const pthread_condattr_t *attr);


pthread_cond_wait和pthread_cond_timedwait用来等待条件变量被设置，值得注意的是这两个等待调用需要一个已经上锁的互斥体mutex，这是为了防止在真正进入等待状态之前别的线程有可能设置该条件变量而产生竞争。

pthread_cond_wait的函数原型为： 

pthread_cond_wait (pthread_cond_t *cond, pthread_mutex_t *mutex);


pthread_cond_broadcast用于设置条件变量，即使得事件发生，这样等待该事件的线程将不再阻塞： 

pthread_cond_broadcast (pthread_cond_t *cond) ;

pthread_cond_signal则用于解除某一个等待线程的阻塞状态： 

pthread_cond_signal (pthread_cond_t *cond) ;


pthread_cond_destroy(pthread_cond_t *cond)  则用于释放一个条件变量的资源。 

pthread_cond_destroy(pthread_cond_t *cond) ；



pthread_cond_timedwait 计时等待方式

int pthread_cond_timedwait(pthread_cond_t *cond, pthread_mutex_t *mutex, const struct timespec *abstime);

如果在给定时刻前条件没有满足，则返回ETIMEOUT，结束等待，其中abstime以与time()系统调用相同意义的绝对时间形式出现，0表示格林尼治时间1970年1月1日0时0分0秒。



pthread_cond_timedwait 和pthread_cond_wait，都必须和一个互斥锁配合，以防止多个线程同时请求pthread_cond_wait()（或pthread_cond_timedwait()，下同）的竞争条件（Race Condition）。mutex互斥锁必须是普通锁（PTHREAD_MUTEX_TIMED_NP）或者适应锁（PTHREAD_MUTEX_ADAPTIVE_NP），且在调用pthread_cond_wait()前必须由本线程加锁（pthread_mutex_lock()），而在更新条件等待队列以前，mutex保持锁定状态，并在线程挂起进入等待前解锁。在条件满足从而离开pthread_cond_wait()之前，mutex将被重新加锁，以与进入pthread_cond_wait()前的加锁动作对应。
激发条件有两种形式，pthread_cond_signal()激活一个等待该条件的线程，存在多个等待线程时按入队顺序激活其中一个；而pthread_cond_broadcast()则激活所有等待线程。


###读写锁
https://www.cnblogs.com/i80386/p/4478021.html
https://www.cnblogs.com/i80386/p/4478021.html
https://www.cnblogs.com/defen/p/4410232.html
https://blog.csdn.net/yand789/article/details/27324295

####gcc 原子操作函数
https://blog.csdn.net/chenjiayi_yun/article/details/16333779

1、原子操作的api函数
（1）直接操作数的原子操作
第一组返回更新前的值，第二组返回更新后的值

type __sync_fetch_and_add (type *ptr, type value, ...)
type __sync_fetch_and_sub (type *ptr, type value, ...)
type __sync_fetch_and_or (type *ptr, type value, ...)
type __sync_fetch_and_and (type *ptr, type value, ...)
type __sync_fetch_and_xor (type *ptr, type value, ...)
type __sync_fetch_and_nand (type *ptr, type value, ...)

type __sync_add_and_fetch (type *ptr, type value, ...)
type __sync_sub_and_fetch (type *ptr, type value, ...)
type __sync_or_and_fetch (type *ptr, type value, ...)
type __sync_and_and_fetch (type *ptr, type value, ...)
type __sync_xor_and_fetch (type *ptr, type value, ...)
type __sync_nand_and_fetch (type *ptr, type value, ...)

type可以是1,2,4或8字节长度的int类型，即：

int8_t / uint8_t
int16_t / uint16_t
int32_t / uint32_t
int64_t / uint64_t
 

后面的可扩展参数(...)用来指出哪些变量需要memory barrier,因为目前gcc实现的是full barrier（类似于linux kernel 中的mb(),表示这个操作之前的所有内存操作不会被重排序到这个操作之后）,所以可以略掉这个参数。

（2）比较后操作数的原子操作
bool __sync_bool_compare_and_swap (type *ptr, type oldval type newval, ...)
type __sync_val_compare_and_swap (type *ptr, type oldval type newval, ...)


这两个函数提供原子的比较和交换，如果*ptr == oldval,就将newval写入*ptr,
第一个函数在相等并写入的情况下返回true.
第二个函数在返回操作之前的值。
 
(3)其他原子操作
type __sync_lock_test_and_set (type *ptr, type value, ...)
   将*ptr设为value并返回*ptr操作之前的值。

void __sync_lock_release (type *ptr, ...)
     将*ptr置0

