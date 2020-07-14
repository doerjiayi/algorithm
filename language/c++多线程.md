#多线程

##原子变量

###atomic_flag 
C++11 并发指南六(atomic 类型详解一 atomic_flag 介绍)
https://www.cnblogs.com/haippy/p/3252056.html

C++11特性（13）：原子操作及原子数据类型（1）
https://blog.csdn.net/caychen/article/details/79711118

https://blog.csdn.net/WizardtoH/article/details/81111549
https://www.cnblogs.com/FateTHarlaown/p/8919235.html
https://en.cppreference.com/w/cpp/atomic/atomic_flag/test_and_set


###c++并发编程之原子操作的实现原理
https://www.cnblogs.com/zhanghu52030/p/9167014.html

###C++11多线程-原子操作
C++11多线程-原子操作(1)
https://www.jianshu.com/p/c0da859a7ce0
C++11多线程-原子操作(2)
https://www.jianshu.com/p/fc6fce6543a9
C++11多线程-内存模型
https://www.jianshu.com/p/7d237771dc94

##c++11 线程池学习笔记 (一) 任务队列
https://www.cnblogs.com/itdef/p/8454389.html

##C++11多线程-mutex(2)
https://www.jianshu.com/p/8bd389d4ed83

##C++11多线程-条件变量(std::condition_variable)
https://www.jianshu.com/p/a31d4fb5594f

前面我们介绍了线程(std::thread)和互斥量(std::mutex)，互斥量是多线程间同时访问某一共享变量时，保证变量可被安全访问的手段。在多线程编程中，还有另一种十分常见的行为：线程同步。线程同步是指线程间需要按照预定的先后次序顺序进行的行为。C++11对这种行为也提供了有力的支持，这就是条件变量。条件变量位于头文件condition_variable下。本章我们将简要介绍一下该类，在文章的最后我们会综合运用std::mutex和std::condition_variable，实现一个chan类，该类可在多线程间安全的通信，具有广泛的应用场景。

1. std::condition_variable
条件变量提供了两类操作：wait和notify。这两类操作构成了多线程同步的基础。

1.1 wait
wait是线程的等待动作，直到其它线程将其唤醒后，才会继续往下执行。下面通过伪代码来说明其用法：

std::mutex mutex;
std::condition_variable cv;

// 条件变量与临界区有关，用来获取和释放一个锁，因此通常会和mutex联用。
std::unique_lock lock(mutex);
// 此处会释放lock，然后在cv上等待，直到其它线程通过cv.notify_xxx来唤醒当前线程，cv被唤醒后会再次对lock进行上锁，然后wait函数才会返回。
// wait返回后可以安全的使用mutex保护的临界区内的数据。此时mutex仍为上锁状态
cv.wait(lock)
需要注意的一点是, wait有时会在没有任何线程调用notify的情况下返回，这种情况就是有名的spurious wakeup。因此当wait返回时，你需要再次检查wait的前置条件是否满足，如果不满足则需要再次wait。wait提供了重载的版本，用于提供前置检查。

template <typename Predicate>
void wait(unique_lock<mutex> &lock, Predicate pred) {
    while(!pred()) {
        wait(lock);
    }
}
除wait外, 条件变量还提供了wait_for和wait_until，这两个名称是不是看着有点儿眼熟，std::mutex也提供了_for和_until操作。在C++11多线程编程中，需要等待一段时间的操作，一般情况下都会有xxx_for和xxx_until版本。前者用于等待指定时长，后者用于等待到指定的时间。

1.2 notify
了解了wait，notify就简单多了：唤醒wait在该条件变量上的线程。notify有两个版本：notify_one和notify_all。

notify_one 唤醒等待的一个线程，注意只唤醒一个。
notify_all 唤醒所有等待的线程。使用该函数时应避免出现惊群效应。
其使用方式见下例：

std::mutex mutex;
std::condition_variable cv;

std::unique_lock lock(mutex);
// 所有等待在cv变量上的线程都会被唤醒。但直到lock释放了mutex，被唤醒的线程才会从wait返回。
cv.notify_all(lock)
2. 线程间通信 - chan的实现
有了上面的基础我们就可以设计我们的线程间通讯工具"chan"了。我们的设计目标：

在线程间安全的传递数据。golang社区有一句经典的话：不要通过共享内存来通信，要通过通信来共享内存。
消除线程线程同步带来的复杂性。
我们先来看一下chan的实际使用效果, 生产者-消费者（一个生产者，多个消费者）

using namespace std::chrono;

// 消费数据 
void consume(chan<int> ch, int thread_id) {
    int n;
    while(ch >> n) {
        printf("[%d] %d\n", thread_id, n);
        std::this_thread::sleep_for(milliseconds(100));
    }
}

int main() {
    chan<int> chInt(3);
    // 消费者
    std::thread consumers[5];
    for (int i = 0; i < 5; i++) {
        consumers[i] = std::thread(consume, chInt, i+1);
    }
    // 生产数据 
    for (int i = 0; i < 16; i++) {
        chInt << i;
    }
    chInt.close();  // 数据生产完毕
    for (std::thread &thr: consumers) {
        thr.join();
    }
    return 0;
}
附: 源码(可在github上下载到)
下面附上chan.simple.h的实现，是chan的较为简单的实现，完整实现请去github下载。该代码在g++和vc 2015下均编译通过，其它平台未验证。


// chan.simple.h

template <typename T>
class chan {
    class queue_t {
        mutable std::mutex mutex_;
        std::condition_variable cv_;
        std::list<T> data_;
        const size_t capacity_;  // data_容量
        const bool enable_overflow_;
        bool closed_ = false;   // 队列是否已关闭
        size_t pop_count_ = 0;  // 计数，累计pop的数量
    public:
        queue_t(size_t capacity) :
            capacity_(capacity == 0 ? 1 : capacity),
            enable_overflow_(capacity == 0) {
        }
        bool is_empty() const {
            return data_.empty();
        }
        size_t free_count() const {
            // capacity_为0时，允许放入一个，但_queue会处于overflow状态
            return capacity_ - data_.size();
        }
        bool is_overflow() const {
            return enable_overflow_ && data_.size() >= capacity_;
        }
        bool is_closed() const {
            std::unique_lock<std::mutex> lock(this->mutex_);
            return this->closed_;
        }
        // close以后的入chan操作会返回false, 而出chan则在队列为空后才返回false
        void close() {
            std::unique_lock<std::mutex> lock(this->mutex_);
            this->closed_ = true;
            if (this->is_overflow()) {
                // 消除溢出
                this->data_.pop_back();
            }
            this->cv_.notify_all();
        }
        template <typename TR>
        bool pop(TR &data) {
            std::unique_lock<std::mutex> lock(this->mutex_);
            this->cv_.wait(lock, [&]() { return !is_empty() || closed_; });
            if (this->is_empty()) {
                return false;  // 已关闭
            }
            data = this->data_.front();
            this->data_.pop_front();
            this->pop_count_++;
            if (this->free_count() == 1) {
                // 说明以前是full或溢出状态
                this->cv_.notify_all();
            }
            return true;
        }
        template <typename TR>
        bool push(TR &&data) {
            std::unique_lock<std::mutex> lock(mutex_);
            cv_.wait(lock, [this]() { return free_count() > 0 || closed_; });
            if (closed_) {
                return false;
            }
            data_.push_back(std::forward<TR>(data));
            if (data_.size() == 1) {
                cv_.notify_all();
            }
            // 当queue溢出,需等待queue回复正常
            if (is_overflow()) {
                const size_t old = this->pop_count_;
                cv_.wait(lock, [&]() { return old != pop_count_ || closed_; });
            }
            return !this->closed_;
        }
    };
    std::shared_ptr<queue_t> queue_;

public:
    explicit chan(size_t capacity = 0) {
        queue_ = std::make_shared<queue_t>(capacity);
    }
    // 支持拷贝
    chan(const chan &) = default;
    chan &operator=(const chan &) = default;
    // 支持move
    chan(chan &&) = default;
    chan &operator=(chan &&) = default;
    // 入chan，支持move语义
    template <typename TR>
    bool operator<<(TR &&data) {
        return queue_->push(std::forward<TR>(data));
    }
    // 出chan(支持兼容类型的出chan)
    template <typename TR>
    bool operator>>(TR &data) {
        return queue_->pop(data);
    }
    // close以后的入chan操作返回false, 而出chan则在队列为空后才返回false
    void close() {
        queue_->close();
    }
    bool is_closed() const {
        return queue_->is_closed();
    }
};


##C++11多线程-异步运行(1)之std::promise
https://www.jianshu.com/p/7945428c220e

前面介绍了C++11的std::thread、std::mutex以及std::condition_variable，并实现了一个多线程通信的chan类，虽然由于篇幅的限制，该实现有些简陋，甚至有些缺陷，但对于一般情况应该还是够用了。在C++11多线程系列的最后会献上chan的最终版本，敬请期待。
本文将介绍C++11的另一大特性：异步运行(std::async)。async顾名思义是将一个函数A移至另一线程中去运行。A可以是静态函数、全局函数，甚至类成员函数。在异步运行的过程中，如果A需要向调用者输出结果怎么办呢？std::async完美解决了这一问题。在了解async的解决之道前，我们需要一些知识储备，那就是：std::promise、std::packaged_task和std::future。异步运行涉及的内容较多，我们会分几节来讲。

1. std::promise
std::promise是一个模板类: template<typename R> class promise。其泛型参数R为std::promise对象保存的值的类型，R可以是void类型。std::promise保存的值可被与之关联的std::future读取，读取操作可以发生在其它线程。std::promise允许move语义(右值构造，右值赋值)，但不允许拷贝(拷贝构造、赋值)，std::future亦然。std::promise和std::future合作共同实现了多线程间通信。

1.1 设置std::promise的值
通过成员函数set_value可以设置std::promise中保存的值，该值最终会被与之关联的std::future::get读取到。需要注意的是：set_value只能被调用一次，多次调用会抛出std::future_error异常。事实上std::promise::set_xxx函数会改变std::promise的状态为ready，再次调用时发现状态已要是reday了，则抛出异常。

 
using namespace std::chrono;

void read(std::future<std::string> *future) {
    // future会一直阻塞，直到有值到来
    std::cout << future->get() << std::endl;
}

int main() {
    // promise 相当于生产者
    std::promise<std::string> promise;
    // future 相当于消费者, 右值构造
    std::future<std::string> future = promise.get_future();
    // 另一线程中通过future来读取promise的值
    std::thread thread(read, &future);
    // 让read等一会儿:)
    std::this_thread::sleep_for(seconds(1));
    // 
    promise.set_value("hello future");
    // 等待线程执行完成
    thread.join();

    return 0;
}
// 控制台输: hello future
与std::promise关联的std::future是通过std::promise::get_future获取到的，自己构造出来的无效。一个std::promise实例只能与一个std::future关联共享状态，当在同一个std::promise上反复调用get_future会抛出future_error异常。
共享状态。在std::promise构造时，std::promise对象会与共享状态关联起来，这个共享状态可以存储一个R类型的值或者一个由std::exception派生出来的异常值。通过std::promise::get_future调用获得的std::future与std::promise共享相同的共享状态。

1.2 当std::promise不设置值时
如果promise直到销毁时，都未设置过任何值，则promise会在析构时自动设置为std::future_error，这会造成std::future.get抛出std::future_error异常。

 
using namespace std::chrono;

void read(std::future<int> future) {
    try {
        future.get();
    } catch(std::future_error &e) {
        std::cerr << e.code() << "\n" << e.what() << std::endl;
    }
}

int main() {
    std::thread thread;
    {
        // 如果promise不设置任何值
        // 则在promise析构时会自动设置为future_error
        // 这会造成future.get抛出该异常
        std::promise<int> promise;
        thread = std::thread(read, promise.get_future());
    }
    thread.join();

    return 0;
}
上面的程序在Clang下输出：

future:4
The associated promise has been destructed prior to the associated state becoming ready.
1.3 通过std::promise让std::future抛出异常
通过std::promise::set_exception函数可以设置自定义异常，该异常最终会被传递到std::future，并在其get函数中被抛出。

void catch_error(std::future<void> &future) {
    try {
        future.get();
    } catch (std::logic_error &e) {
        std::cerr << "logic_error: " << e.what() << std::endl;
    }
}

int main() {
    std::promise<void> promise;
    std::future<void> future = promise.get_future();

    std::thread thread(catch_error, std::ref(future));
    // 自定义异常需要使用make_exception_ptr转换一下
    promise.set_exception(
        std::make_exception_ptr(std::logic_error("caught")));
    
    thread.join();
    return 0;
}
// 输出：logic_error: caught
std::promise虽然支持自定义异常，但它并不直接接受异常对象：

// std::promise::set_exception函数原型
void set_exception(std::exception_ptr p);
自定义异常可以通过位于头文件exception下的std::make_exception_ptr函数转化为std::exception_ptr。

1.4 std::promise<void>
通过上面的例子，我们看到std::promise<void>是合法的。此时std::promise.set_value不接受任何参数，仅用于通知关联的std::future.get()解除阻塞。

1.5 std::promise所在线程退出时
std::async(异步运行)时，开发人员有时会对std::promise所在线程退出时间比较关注。std::promise支持定制线程退出时的行为：

std::promise::set_value_at_thread_exit 线程退出时，std::future收到通过该函数设置的值。
std::promise::set_exception_at_thread_exit 线程退出时，std::future则抛出该函数指定的异常。
关于std::promise就是这些，本文从使用角度介绍了std::promise的能力以及边界，读者如果想更深入了解该类，可以直接阅读一下源码。




##packaged_task
https://www.jianshu.com/p/72601d82f3df

上一篇介绍的std::promise通过set_value可以使得与之关联的std::future获取数据。本篇介绍的std::packaged_task则更为强大，它允许传入一个函数，并将函数计算的结果传递给std::future，包括函数运行时产生的异常。下面我们就来详细介绍一下它。

2. std::package_task
在开始std::packaged_task之前我们先看一段代码，对std::packaged_task有个直观的印象，然后我们再进一步介绍。

//#include <thread>   // std::thread
//#include <future>   // std::packaged_task, std::future
//#include <iostream> // std::cout

int sum(int a, int b) {
    return a + b;
}

int main() {
    std::packaged_task<int(int,int)> task(sum);
    std::future<int> future = task.get_future();

    // std::promise一样，std::packaged_task支持move，但不支持拷贝
    // std::thread的第一个参数不止是函数，还可以是一个可调用对象，即支持operator()(Args...)操作
    std::thread t(std::move(task), 1, 2);
    // 等待异步计算结果
    std::cout << "1 + 2 => " << future.get() << std::endl;

    t.join();
    return 0;
}
/// 输出: 1 + 2 => 3
std::packaged_task位于头文件#include <future>中，是一个模板类

template <class R, class... ArgTypes>
class packaged_task<R(ArgTypes...)>
其中R是一个函数或可调用对象，ArgTypes是R的形参。与std::promise一样，std::packaged_task支持move，但不支持拷贝(copy)。std::packaged_task封装的函数的计算结果会通过与之联系的std::future::get获取(当然，可以在其它线程中异步获取)。关联的std::future可以通过std::packaged_task::get_future获取到，get_future仅能调用一次，多次调用会触发std::future_error异常。
std::package_task除了可以通过可调用对象构造外，还支持缺省构造(无参构造)。但此时构造的对象不能直接使用，需通过右值赋值操作设置了可调用对象或函数后才可使用。判断一个std::packaged_task是否可使用，可通过其成员函数valid来判断。

2.1 std::packaged_task::valid
该函数用于判断std::packaged_task对象是否是有效状态。当通过缺省构造初始化时，由于其未设置任何可调用对象或函数，valid会返回false。只有当std::packaged_task设置了有效的函数或可调用对象，valid才返回true。

//#include <future>   // std::packaged_task, std::future
//#include <iostream> // std::cout

int main() {
    std::packaged_task<void()> task; // 缺省构造、默认构造
    std::cout << std::boolalpha << task.valid() << std::endl; // false

    std::packaged_task<void()> task2(std::move(task)); // 右值构造
    std::cout << std::boolalpha << task.valid() << std::endl; // false

    task = std::packaged_task<void()>([](){});  // 右值赋值, 可调用对象
    std::cout << std::boolalpha << task.valid() << std::endl; // true

    return 0;
}
上面的示例演示了几种valid为false的情况，程序输出如下

false
false
true
2.2 std::packaged_task::operator()(ArgTypes...)
该函数会调用std::packaged_task对象所封装可调用对象R，但其函数原型与R稍有不同:

void operator()(ArgTypes... );
operator()的返回值是void，即无返回值。因为std::packaged_task的设计主要是用来进行异步调用，因此R(ArgTypes...)的计算结果是通过std::future::get来获取的。该函数会忠实地将R的计算结果反馈给std::future，即使R抛出异常(此时std::future::get也会抛出同样的异常)。

//#include <future>   // std::packaged_task, std::future
//#include <iostream> // std::cout

int main() {
    std::packaged_task<void()> convert([](){
        throw std::logic_error("will catch in future");
    });
    std::future<void> future = convert.get_future();

    convert(); // 异常不会在此处抛出

    try {
        future.get();
    } catch(std::logic_error &e) {
        std::cerr << typeid(e).name() << ": " << e.what() << std::endl;
    }

    return 0;
}
/// Clang下输出: St11logic_error: will catch in future
为了帮忙大家更好的了解该函数，下面将Clang下精简过的operator()(Args...)的实现贴出，以便于更好理解该函数的边界，明确什么可以做，什么不可以做。

template<class _Rp, class ..._ArgTypes>
class packaged_task<_Rp(_ArgTypes...)> {
    __packaged_task_function<_Rp_(_ArgTypes...)> __f_;
    promise<_Rp> __p_;  // 内部采用了promise实现

public:
    // 构造、析构以及其它函数...

    void packaged_task<_Rp(_ArgTypes...)>::operator()(_ArgTypes... __args) {
        if (__p_.__state_ == nullptr)
            __throw_future_error(future_errc::no_state);
        if (__p_.__state_->__has_value())  // __f_不可重复调用
            __throw_future_error(future_errc::promise_already_satisfied);

        try {
            __p_.set_value(__f_(std::forward<_ArgTypes>(__args)...));
        } catch (...) {
            __p_.set_exception(current_exception());
        }
    }
};
2.3 让std::packaged_task在线程退出时再将结果反馈给std::future
std::packaged_task::make_ready_at_thread_exit函数接收的参数与operator()(_ArgTypes...)一样，行为也一样。只有一点差别，那就是不会将计算结果立刻反馈给std::future，而是在其执行时所在的线程结束后，std::future::get才会取得结果。

2.4 std::packaged_task::reset
与std::promise不一样， std::promise仅可以执行一次set_value或set_exception函数，但std::packagged_task可以执行多次，其奥秘就是reset函数

template<class _Rp, class ..._ArgTypes>
void packaged_task<_Rp(_ArgTypes...)>::reset()
{
    if (!valid())
        __throw_future_error(future_errc::no_state);
    __p_ = promise<result_type>();
}
通过重新构造一个promise来达到多次调用的目的。显然调用reset后，需要重新get_future，以便获取下次operator()执行的结果。由于是重新构造了promise，因此reset操作并不会影响之前调用的make_ready_at_thread_exit结果，也即之前的定制的行为在线程退出时仍会发生。

std::packaged_task就介绍到这里，下一篇将会完成本次异步运行的整体脉络，将std::async和std::future一起介绍结大家。

附: C++11多线程中的样例代码的编译及运行
g++ -std=c++11 <Your Cpp File>
./a.out


##future+async
https://www.jianshu.com/p/58dea28d1a95

前面两章多次使用到std::future，本章我们就来揭开std::future庐山真面目。最后我们会引出std::async，该函数使得我们的并发调用变得简单，优雅。

3. std::future
前面我们多次使用std::future的get方法来获取其它线程的结果，那么除这个方法外，std::future还有哪些方法呢。

enum class future_status
{
    ready,
    timeout,
    deferred
};
template <class R>
class future
{
public:
    // retrieving the value
    R get();
    // functions to check state
    bool valid() const noexcept;

    void wait() const;
    template <class Rep, class Period>
    future_status wait_for(const chrono::duration<Rep, Period>& rel_time) const;
    template <class Clock, class Duration>
    future_status wait_until(const chrono::time_point<Clock, Duration>& abs_time) const;

    shared_future<R> share() noexcept;
};
以上代码去掉了std::future构造、析构、赋值相关的代码，这些约束我们之前都讲过了。下面我们来逐一了解上面这些函数。

3.1 get
这个函数我们之前一直使用，该函数会一直阻塞，直到获取到结果或异步任务抛出异常。

3.2 share
std::future允许move，但是不允许拷贝。如果用户确有这种需求，需要同时持有多个实例，怎么办呢? 这就是share发挥作用的时候了。std::shared_future通过引用计数的方式实现了多实例共享同一状态，但有计数就伴随着同步开销(无锁的原子操作也是有开销的)，性能会稍有下降。因此C++11要求程序员显式调用该函数，以表明用户对由此带来的开销负责。std::shared_future允许move，允许拷贝，并且具有和std::future同样的成员函数，此处就不一一介绍了。当调用share后，std::future对象就不再和任何共享状态关联，其valid函数也会变为false。

3.3 wait
等待，直到数据就绪。数据就绪时，通过get函数，无等待即可获得数据。

3.4 wait_for和wait_until
wait_for、wait_until主要是用来进行超时等待的。wait_for等待指定时长，wait_until则等待到指定的时间点。返回值有3种状态：

ready - 数据已就绪，可以通过get获取了。
timeout - 超时，数据还未准备好。
deferred - 这个和std::async相关，表明无需wait，异步函数将在get时执行。
3.5 valid
判断当前std::future实例是否有效。std::future主要是用来获取异步任务结果的，作为消费方出现，单独构建出来的实例没意义，因此其valid为false。当与其它生产方(Provider)通过共享状态关联后，valid才会变得有效，std::future才会发挥实际的作用。C++11中有下面几种Provider，从这些Provider可获得有效的std::future实例：

std::async
std::promise::get_future
std::packaged_task::get_future
既然std::future的各种行为都依赖共享状态，那么什么是共享状态呢?

4. 共享状态
共享状态其本质就是单生产者-单消费者的多线程并发模型。无论是std::promise还是std::packaged_task都是通过共享状态，实现与std::future通信的。还记得我们在std::condition_variable一节给出的chan类么。共享状态与其类似，通过std::mutex、std::condition_variable实现了多线程间通信。共享状态并非C++11的标准，只是对std::promise、std::future的实现手段。回想我们之前的使用场景，共享状态可能具有如下形式(c++11伪代码):

template<typename T>
class assoc_state {
protected:
    mutable mutex mut_;
    mutable condition_variable cv_;
    unsigned state_ = 0;
    // std::shared_future中拷贝动作会发生引用计数的变化
    // 当引用计数降到0时，实例被delete
    int share_count_ = 0;
    exception_ptr exception_; // 执行异常
    T value_;  // 执行结果

public:
    enum {
        ready = 4,  // 异步动作执行完，数据就绪
        // 异步动作将延迟到future.get()时调用
        // (实际上非异步，只不过是延迟执行)
        deferred = 8,
    };

    assoc_state() {}
    // 禁止拷贝
    assoc_state(const assoc_state &) = delete;
    assoc_state &operator=(const assoc_state &) = delete;
    // 禁止move
    assoc_state(assoc_state &&) = delete;
    assoc_state &operator=(assoc_state &&) = delete;

    void set_value(const T &);
    void set_exception(exception_ptr p);
    // 需要用到线程局变存储
    void set_value_at_thread_exit(const T &);
    void set_exception_at_thread_exit(exception_ptr p);

    void wait();
    future_status wait_for(const duration &) const;
    future_status wait_until(const time_point &) const;

    T &get() {
        unique_lock<mutex> lock(this->mut_);
        // 当_state为deferred时，std::async中
        // 的函数将在sub_wait中调用
        this->sub_wait(lock);
        if (this->_exception != nullptr)
            rethrow_exception(this->_exception);
        return _value;
    }
private:
    void sub_wait(unique_lock<mutex> &lk) {
        if (state_ != ready) {
            if (state_ & static_cast<unsigned>(deferred)) {
                state_ &= ~static_cast<unsigned>(deferred);
                lk.unlock();
                __execute();  // 此处执行实际的函数调用
            } else {
                cv_.wait(lk, [this](){return state == ready;})
            }
        }
    }
};
以上给出了get的实现(伪代码)，其它部分虽然没实现，但assoc_state应该具有的功能，以及对std::promise、std::packaged_task、std::future、std::shared_future的支撑应该能够表达清楚了。未实现部分还请读者自行补充一下，权当是练手了。
有兴趣的读者可以阅读llvm-libxx(https://github.com/llvm-mirror/libcxx)的源码，以了解更多细节，对共享状态有更深掌握。

5. std::async
std::async可以看作是对std::packaged_task的封装(虽然实际并一定如此，取决于编译器的实现，但共享状态的思想是不变的)，有两种重载形式:

//#define FR typename result_of<typename decay<F>::type(typename decay<Args>::type...)>::type

// 不含执行策略
template <class F, class... Args>
future<FR> async(F&& f, Args&&... args);
// 含执行策略
template <class F, class... Args>
future<FR> async(launch policy, F&& f, Args&&... args);
define部分是用来推断函数F的返回值类型，我们先忽略它，以后有机再讲。两个重载形式的差别是一个含执行策略，而另一个不含。那么什么是执行策略呢？执行策略定义了async执行F(函数或可调用求对象)的方式，是一个枚举值：

enum class launch {
    // 保证异步行为，F将在单独的线程中执行
    async = 1,
    // 当其它线程调用std::future::get时，
    // 将调用非异步形式, 即F在get函数内执行
    deferred = 2,
    // F的执行时机由std::async来决定
    any = async | deferred
};
不含加载策略的版本，使用的是std::launch::any，也即由std::async函数自行决定F的执行策略。那么C++11如何确定std::any下的具体执行策略呢，一种可能的办法是：优先使用async策略，如果创建线程失败，则使用deferred策略。实际上这也是Clang的any实现方式。std::async的出现大大减轻了异步的工作量。使得一个异步调用可以像执行普通函数一样简单。

//#include <iostream> // std::cout, std::endl
//#include <future>   // std::async, std::future
//#include <chrono>   // seconds
using namespace std::chrono;

int main() {
    auto print = [](char c) {
        for (int i = 0; i < 10; i++) {
            std::cout << c;
            std::cout.flush();
            std::this_thread::sleep_for(milliseconds(1));
        }
    };
    // 不同launch策略的效果
    std::launch policies[] = {std::launch::async, std::launch::deferred};
    const char *names[] = {"async   ", "deferred"};
    for (int i = 0; i < sizeof(policies)/sizeof(policies[0]); i++) {
        std::cout << names[i] << ": ";
        std::cout.flush();
        auto f1 = std::async(policies[i], print, '+');
        auto f2 = std::async(policies[i], print, '-');
        f1.get();
        f2.get();
        std::cout << std::endl;
    }
    return 0;
}
以上代码输出如下：

async   : +-+-+-+--+-++-+--+-+
deferred: ++++++++++----------
进行到现在，C++11的async算是结束了，尽管还留了一些疑问，比如共享状态如何实现set_value_at_thread_exit效果。我们将会在下一章节介绍C++11的线程局部存储，顺便也解答下该疑问。


##thread_local
https://www.jianshu.com/p/8df45004bbcb


##c++ future
https://blog.csdn.net/lijinqi1987/article/details/78909479
