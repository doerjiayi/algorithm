#c++
c++编程思想

##c++反射
https://www.cnblogs.com/lizhanwu/p/4428990.html

##原子变量
https://blog.csdn.net/caychen/article/details/79711118
https://www.cnblogs.com/haippy/p/3252056.html

https://blog.csdn.net/WizardtoH/article/details/81111549
https://www.cnblogs.com/FateTHarlaown/p/8919235.html
https://en.cppreference.com/w/cpp/atomic/atomic_flag/test_and_set

##原子原理
https://www.cnblogs.com/zhanghu52030/p/9167014.html

##字节对齐
https://blog.csdn.net/cclethe/article/details/79659590

##operator()
http://www.360doc.com/content/18/0526/09/54097382_757112296.shtml

##operator[]()
https://blog.csdn.net/qq_37474728/article/details/82586068

##std::ref和std::cref使用
https://blog.csdn.net/lmb1612977696/article/details/81543802

##c++可变模板
https://www.cnblogs.com/wxquare/p/4743180.html

##c++虚表
###虚函数实现原理
https://blog.csdn.net/weixin_40237626/article/details/82313339
一般继承（有虚函数覆盖）
1）覆盖的f()函数被放到了虚表中原来父类虚函数的位置。
2）没有被覆盖的函数依旧。


多重继承（无虚函数覆盖）
1） 每个父类都有自己的虚表。
2） 子类的成员函数被放到了第一个父类的表中。（所谓的第一个父类是按照声明顺序来判断的）
这样做就是为了解决不同的父类类型的指针指向同一个子类实例，而能够调用到实际的函数。


多重继承（有虚函数覆盖）
下面我们再来看看，如果发生虚函数覆盖的情况。

我们在子类中覆盖了父类的f()函数。

安全性
每次写C++的文章，总免不了要批判一下C++。这篇文章也不例外。通过上面的讲述，相信我们对虚函数表有一个比较细致的了解了。水可载舟，亦可覆舟。下面，让我们来看看我们可以用虚函数表来干点什么坏事吧。

一、通过父类型的指针访问子类自己的虚函数

我们知道，子类没有重载父类的虚函数是一件毫无意义的事情。因为多态也是要基于函数重载的。虽然在上面的图中我们可以看到Base1的虚表中有Derive的虚函数，但我们根本不可能使用下面的语句来调用子类的自有虚函数：

 

Base1 *b1 = new Derive();

b1->f1(); //编译出错

任何妄图使用父类指针想调用子类中的未覆盖父类的成员函数的行为都会被编译器视为非法，所以，这样的程序根本无法编译通过。但在运行时，我们可以通过指针的方式访问虚函数表来达到违反C++语义的行为。

二、访问non-public的虚函数

另外，如果父类的虚函数是private或是protected的，但这些非public的虚函数同样会存在于虚函数表中，所以，我们同样可以使用访问虚函数表的方式来访问这些non-public的虚函数，这是很容易做到的。

如：

class Base {
private:
virtual void f() { cout << "Base::f" << endl; }

};

class Derive : public Base{

};

typedef void(*Fun)(void);

 
void main() {
Derive d;
Fun pFun = (Fun)*((int*)*(int*)(&d)+0);
pFun();
}


##effective c++
https://www.cnblogs.com/deepllz/p/9171908.html
https://blog.csdn.net/qq_34536551/article/details/87293775
https://blog.csdn.net/yusiguyuan/article/details/41950323
https://blog.csdn.net/u011619422/article/details/44218473

##vector都有哪些接口，插入操作
https://www.cnblogs.com/yocichen/p/10574819.html

##bind function
https://www.cnblogs.com/bencai/p/9124654.html
https://www.cnblogs.com/SZxiaochun/p/8017349.html

##stoi
https://blog.csdn.net/qq_33221533/article/details/82119031

##boost
https://www.cnblogs.com/gaowengang/p/8994370.html
https://www.cnblogs.com/LyndonYoung/articles/5288618.html
https://www.boost.org/users/history/version_1_71_0.html

##dll
https://blog.csdn.net/huangyimo/article/details/81749340
https://blog.csdn.net/liyuanbhu/article/details/50365287

##windows 和linux 同步api对比

https://blog.csdn.net/chenjiayi_yun/article/details/8719476
https://blog.csdn.net/chenjiayi_yun/article/details/8790828
https://blog.csdn.net/chenjiayi_yun/article/details/8797807

##char数组和十六进制格式化sprintf
https://blog.csdn.net/huanghxyz/article/details/84568155

#c++11

##shared_ptr原理分析及实现
https://blog.csdn.net/peng864534630/article/details/77932574

##c++ future
https://blog.csdn.net/lijinqi1987/article/details/78909479

##c++11 智能指针 unique_ptr、shared_ptr与weak_ptr
https://www.cnblogs.com/lsgxeva/p/7788061.html

C++11中有unique_ptr、shared_ptr与weak_ptr等智能指针(smart pointer)，定义在<memory>中。

可以对动态资源进行管理，保证任何情况下，已构造的对象最终会销毁，即它的析构函数最终会被调用。

###unique_ptr
unique_ptr持有对对象的独有权，同一时刻只能有一个unique_ptr指向给定对象（通过禁止拷贝语义、只有移动语义来实现）。

unique_ptr指针本身的生命周期：从unique_ptr指针创建时开始，直到离开作用域。

离开作用域时，若其指向对象，则将其所指对象销毁(默认使用delete操作符，用户可指定其他操作)。

void mytest()
{
    std::unique_ptr<int> up1(new int(11));   // 无法复制的unique_ptr
    //unique_ptr<int> up2 = up1;        // err, 不能通过编译
    std::cout << *up1 << std::endl;   // 11
    std::unique_ptr<int> up3 = std::move(up1);    // 现在p3是数据的唯一的unique_ptr
    std::cout << *up3 << std::endl;   // 11
    //std::cout << *up1 << std::endl;   // err, 运行时错误
    up3.reset();            // 显式释放内存
    up1.reset();            // 不会导致运行时错误
    //std::cout << *up3 << std::endl;   // err, 运行时错误
    std::unique_ptr<int> up4(new int(22));   // 无法复制的unique_ptr
    up4.reset(new int(44)); //"绑定"动态对象
    std::cout << *up4 << std::endl; // 44
    up4 = nullptr;//显式销毁所指对象，同时智能指针变为空指针。与up4.reset()等价
    std::unique_ptr<int> up5(new int(55));
    int *p = up5.release(); //只是释放控制权，不会释放内存
    std::cout << *p << std::endl;
    //cout << *up5 << endl; // err, 运行时错误
    delete p; //释放堆区资源
    return;
}

int main()
{
    mytest();
    system("pause");
    return 0;
}

###shared_ptr
shared_ptr

shared_ptr允许多个该智能指针共享第“拥有”同一堆分配对象的内存，这通过引用计数（reference counting）实现，会记录有多少个shared_ptr共同指向一个对象，一旦最后一个这样的指针被销毁，也就是一旦某个对象的引用计数变为0，这个对象会被自动删除。

void mytest()
{
    std::shared_ptr<int> sp1(new int(22));
    std::shared_ptr<int> sp2 = sp1;
    std::cout << "cout: " << sp2.use_count() << std::endl; // 打印引用计数
    std::cout << *sp1 << std::endl;
    std::cout << *sp2 << std::endl;
    sp1.reset(); // 显示让引用计数减一
    std::cout << "count: " << sp2.use_count() << std::endl; // 打印引用计数
    std::cout << *sp2 << std::endl; // 22
    return;
}

int main()
{
    mytest();
    system("pause");
    return 0;
}

###weak_ptr
weak_ptr

weak_ptr是为配合shared_ptr而引入的一种智能指针来协助shared_ptr工作，它可以从一个shared_ptr或另一个weak_ptr对象构造，它的构造和析构不会引起引用计数的增加或减少。没有重载 * 和 -> 但可以使用lock获得一个可用的shared_ptr对象

weak_ptr的使用更为复杂一点，它可以指向shared_ptr指针指向的对象内存，却并不拥有该内存，而使用weak_ptr成员lock，则可返回其指向内存的一个share_ptr对象，且在所指对象内存已经无效时，返回指针空值nullptr。

注意：weak_ptr并不拥有资源的所有权，所以不能直接使用资源。
可以从一个weak_ptr构造一个shared_ptr以取得共享资源的所有权。

void check(std::weak_ptr<int> &wp)
{
    std::shared_ptr<int> sp = wp.lock(); // 转换为shared_ptr<int>
    if (sp != nullptr)
    {
        std::cout << "still: " << *sp << std::endl;
    } 
    else
    {
        std::cout << "still: " << "pointer is invalid" << std::endl;
    }
}


void mytest()
{
    std::shared_ptr<int> sp1(new int(22));
    std::shared_ptr<int> sp2 = sp1;
    std::weak_ptr<int> wp = sp1; // 指向shared_ptr<int>所指对象
    std::cout << "count: " << wp.use_count() << std::endl; // count: 2
    std::cout << *sp1 << std::endl; // 22
    std::cout << *sp2 << std::endl; // 22
    check(wp); // still: 22
    sp1.reset();
    std::cout << "count: " << wp.use_count() << std::endl; // count: 1
    std::cout << *sp2 << std::endl; // 22
    check(wp); // still: 22
    sp2.reset();
    std::cout << "count: " << wp.use_count() << std::endl; // count: 0
    check(wp); // still: pointer is invalid
    return;
}

int main()
{
    mytest();
    system("pause");
    return 0;
}

##c++11 类默认函数的控制："=default" 和 "=delete"函数
https://www.cnblogs.com/lsgxeva/p/7787438.html
//c++11 类默认函数的控制："=default" 和 "=delete"函数

/*
C++ 的类有四类特殊成员函数，它们分别是：默认构造函数、析构函数、拷贝构造函数以及拷贝赋值运算符。
这些类的特殊成员函数负责创建、初始化、销毁，或者拷贝类的对象。
如果程序员没有显式地为一个类定义某个特殊成员函数，而又需要用到该特殊成员函数时，则编译器会隐式的为这个类生成一个默认的特殊成员函数。
*/

// C++11 标准引入了一个新特性："=default"函数。程序员只需在函数声明后加上“=default;”，就可将该函数声明为 "=default"函数，编译器将为显式声明的 "=default"函数自动生成函数体。
class X
{ 
public: 
    X() = default; //该函数比用户自己定义的默认构造函数获得更高的代码效率
    X(int i)
    { 
        a = i; 
    }

private: 
    int a; 
}; 

X obj;

// "=default"函数特性仅适用于类的特殊成员函数，且该特殊成员函数没有默认参数。
class X1
{
public:
    int f() = default;      // err , 函数 f() 非类 X 的特殊成员函数
    X1(int, int) = default;  // err , 构造函数 X1(int, int) 非 X 的特殊成员函数
    X1(int = 1) = default;   // err , 默认构造函数 X1(int=1) 含有默认参数
};

// "=default"函数既可以在类体里（inline）定义，也可以在类体外（out-of-line）定义。
class X2
{
public:
    X2() = default; //Inline defaulted 默认构造函数
    X2(const X&);
    X2& operator = (const X&);
    ~X2() = default;  //Inline defaulted 析构函数
};

X2::X2(const X&) = default;  //Out-of-line defaulted 拷贝构造函数
X2& X2::operator= (const X2&) = default;   //Out-of-line defaulted  拷贝赋值操作符


// 为了能够让程序员显式的禁用某个函数，C++11 标准引入了一个新特性："=delete"函数。程序员只需在函数声明后上“=delete;”，就可将该函数禁用。
class X3
{
public:
    X3();
    X3(const X3&) = delete;  // 声明拷贝构造函数为 deleted 函数
    X3& operator = (const X3 &) = delete; // 声明拷贝赋值操作符为 deleted 函数
};

// "=delete"函数特性还可用于禁用类的某些转换构造函数，从而避免不期望的类型转换
class X4
{
public:
    X4(double)
    {

    }
    X4(int) = delete;
};

// "=delete"函数特性还可以用来禁用某些用户自定义的类的 new 操作符，从而避免在自由存储区创建类的对象
class X5
{
public:
    void *operator new(size_t) = delete;
    void *operator new[](size_t) = delete;
};


void mytest()
{
    X4 obj1;
    X4 obj2=obj1;   // 错误，拷贝构造函数被禁用
    X4 obj3;
    obj3=obj1;     // 错误，拷贝赋值操作符被禁用
    X5 *pa = new X5;      // 错误，new 操作符被禁用
    X5 *pb = new X5[10];  // 错误，new[] 操作符被禁用
    return;
}


int main()
{
    mytest();
    system("pause");
    return 0;
}


##c++11 move 
###左值、左值引用、右值、右值引用
https://www.cnblogs.com/SZxiaochun/p/8017475.html

1、左值和右值的概念
         左值是可以放在赋值号左边可以被赋值的值；左值必须要在内存中有实体；
         右值当在赋值号右边取出值赋给其他变量的值；右值可以在内存也可以在CPU寄存器。
         一个对象被用作右值时，使用的是它的内容(值)，被当作左值时，使用的是它的地址。

2、引用
        引用是C++语法做的优化，引用的本质还是靠指针来实现的。引用相当于变量的别名。
        引用可以改变指针的指向，还可以改变指针所指向的值。
        引用的基本规则：

声明引用的时候必须初始化，且一旦绑定，不可把引用绑定到其他对象；即引用必须初始化，不能对引用重定义；
对引用的一切操作，就相当于对原对象的操作。
3、左值引用和右值引用
    3.1 左值引用
         左值引用的基本语法：type &引用名 = 左值表达式；
    3.2 右值引用
        右值引用的基本语法type &&引用名 = 右值表达式；
        右值引用在企业开发人员在代码优化方面会经常用到。
        右值引用的“&&”中间不可以有空格。


###C++ move构造函数和move赋值
https://www.jianshu.com/p/f027aaf95fcf

看一下下面这个例子
template<class T>
class Auto_ptr3
{
    T* m_ptr;
public:
    Auto_ptr3(T* ptr = nullptr)
        :m_ptr(ptr)
    {
    }
    ~Auto_ptr3()
    {
        delete m_ptr;
    }
    // Copy constructor
    // Do deep copy of a.m_ptr to m_ptr
    Auto_ptr3(const Auto_ptr3& a)
    {
        m_ptr = new T;
        *m_ptr = *a.m_ptr;
    }
    // Copy assignment
    // Do deep copy of a.m_ptr to m_ptr
    Auto_ptr3& operator=(const Auto_ptr3& a)
    {
        // Self-assignment detection
        if (&a == this)
            return *this;
        // Release any resource we're holding
        delete m_ptr;
        // Copy the resource
        m_ptr = new T;
        *m_ptr = *a.m_ptr;
        return *this;
    }
    T& operator*() const { return *m_ptr; }
    T* operator->() const { return m_ptr; }
    bool isNull() const { return m_ptr == nullptr; }
};
 
class Resource
{
public:
    Resource() { std::cout << "Resource acquired\n"; }
    ~Resource() { std::cout << "Resource destroyed\n"; }
};
 
Auto_ptr3<Resource> generateResource()
{
    Auto_ptr3<Resource> res(new Resource);
    return res; // this return value will invoke the copy constructor
}
 
int main()
{
    Auto_ptr3<Resource> mainres;
    mainres = generateResource(); // this assignment will invoke the copy assignment
    return 0;
}
输出：

Resource acquired
Resource acquired
Resource destroyed
Resource acquired
Resource destroyed
Resource destroyed
好多构造函数和析构函数执行，下面分析一下每个打印代表的步骤
1)nside generateResource() new Resource构造函数调用
2)generateResource 返回main,res调用copy构造函数给一个临时变量

res在generateResource退出后调用析构函数
4)临时变量调用copy构造函数给mainres
临时变量调用析构函数
main退出，mainres 调用析构函数
优化如下：
 
template<class T>
class Auto_ptr4
{
    T* m_ptr;
public:
    Auto_ptr4(T* ptr = nullptr)
        :m_ptr(ptr)
    {
    }
    ~Auto_ptr4()
    {
        delete m_ptr;
    }
    // Copy constructor
    // Do deep copy of a.m_ptr to m_ptr
    Auto_ptr4(const Auto_ptr4& a)
    {
        m_ptr = new T;
        *m_ptr = *a.m_ptr;
    }
    // Move constructor
    // Transfer ownership of a.m_ptr to m_ptr
    Auto_ptr4(Auto_ptr4&& a)
        : m_ptr(a.m_ptr)
    {
        a.m_ptr = nullptr; // we'll talk more about this line below
    }
    // Copy assignment
    // Do deep copy of a.m_ptr to m_ptr
    Auto_ptr4& operator=(const Auto_ptr4& a)
    {
        // Self-assignment detection
        if (&a == this)
            return *this;
        // Release any resource we're holding
        delete m_ptr;
        // Copy the resource
        m_ptr = new T;
        *m_ptr = *a.m_ptr;
        return *this;
    }
    // Move assignment
    // Transfer ownership of a.m_ptr to m_ptr
    Auto_ptr4& operator=(Auto_ptr4&& a)
    {
        // Self-assignment detection
        if (&a == this)
            return *this;
        // Release any resource we're holding
        delete m_ptr;
        // Transfer ownership of a.m_ptr to m_ptr
        m_ptr = a.m_ptr;
        a.m_ptr = nullptr; // we'll talk more about this line below
        return *this;
    }
    T& operator*() const { return *m_ptr; }
    T* operator->() const { return m_ptr; }
    bool isNull() const { return m_ptr == nullptr; }
};
 
class Resource
{
public:
    Resource() { std::cout << "Resource acquired\n"; }
    ~Resource() { std::cout << "Resource destroyed\n"; }
};
 
Auto_ptr4<Resource> generateResource()
{
    Auto_ptr4<Resource> res(new Resource);
    return res; // this return value will invoke the move constructor
}
 
int main()
{
    Auto_ptr4<Resource> mainres;
    mainres = generateResource(); // this assignment will invoke the move assignment
    return 0;
}
输出:

Resource acquired
Resource destroyed
1)第一步同上
2）函数返回，保存返回值给临时变量，调用move constructed函数，只进行指针copy
3）函数返回,res回收，由于res不指向任何值，因此没操作
4）临时变量赋值给mainres，同第2不
5）main退出，临时变量不指向任何值，无操作
6）main退出，mainres 调用析构函数"Resource destroyed"执行

那么什么条件下执行copy构造函数，什么情况下调用Move构造函数
1） 如果我们给一个l-value赋值的时候，调用copy构造函数
2）如果给一个r-value赋值的时候，调用Move构造函数，因为r-value都是临时的，将要被销毁的
在上面的例子中，generateResource（）函数返回值调用的是move构造函数，而不是copy构造函数，跟第一条不符？C++定义函数返回值是调用move即使是给l-value复制。


https://www.cnblogs.com/xiaoshiwang/p/9582325.html
https://www.cnblogs.com/LearningTheLoad/p/7690052.html

##c/c++ 右值引用，forward关键字
https://www.cnblogs.com/xiaoshiwang/p/9589008.html

c++ forward关键字
forward的由来：模板函数中的推导类型，作为另一函数的参数时，不管实参是什么类型，作为另一个参数的实参时，都变成了左值。因为C++里规定函数的形参就是左值，不过调用侧的实参是否是右值。所以，调用的另一个函数的形参即使用T&& arg来声明，传过去的也是左值，编译不过，因为不能自动把左值转化成右值，除非使用std::move。forward就是为了解决这个问题的。
forward() 函数的作用：保持住实参的类型。
include <iostream>
using namespace std;

void rvalue_call(int& v){
  cout << "& call" << endl;
}
void rvalue_call(int&& v){
  cout << "&& call" << endl;
}
void rvalue_call(const int& v){
  cout << "const & call" << endl;
}
void rvalue_call(const int&& v){
  cout << "const && call" << endl;
}

template<typename T>
void func(T&& a){
  rvalue_call(a);
}

int main(void){
  int x = 1;
  func(x);//实参为左值                                           
  int& y = x;
  func(y);//实参为左值引用                                       
  func(std::move(y));//实参为右值引用                            
  func(100);//实参为右值引用          
  const int a = 11;
  func(a);//实参为左值常引用   
  func(std::move(a));//实参为右值常引用   
}
执行结果：

& call
& call
& call
& call
const & call
const & call
上面的例子即使传递的是右值，但也不能够调用到

void rvalue_call(int&& v)
void rvalue_call(const int&& v)
解决办法：加std::forward

using namespace std;

void rvalue_call(int& v){
  cout << "& call" << endl;
}
void rvalue_call(int&& v){
  cout << "&& call" << endl;
}
void rvalue_call(const int& v){
  cout << "const & call" << endl;
}
void rvalue_call(const int&& v){
  cout << "const && call" << endl;
}

template<typename T>
void func(T&& a){
  rvalue_call(std::forward<T> (a));
}

int main(void){
  int x = 1;
  func(x);//实参为左值                                           
  int& y = x;
  func(y);//实参为左值引用                                       
  func(std::move(y));//实参为右值引用                            
  func(100);
    
  const int a = 11;
  func(a);
  func(std::move(a));
}
执行结果：发现可以调用到右值的两个函数。这就是std::forward函数在模板里的作用

& call
& call
&& call
&& call
const & call
const && call
另一个例子：

template<typename F, typename T1, typename T2>
void fcn2(F f, T1&& t1, T2&& t2){
  f(std::forward<T2>(t2), std::forward<T1>(t1));//OK
  //f(std::move(t2), std::forward<T1>(t1));//OK
  //f(t2, t1);//ERROR
}

void f1(int&& i1, int& i2){
  i1 = 10;
  i2 = 20;
}

int main(){
  int i1 = 1, i2 = 2;
  int& a = i1;
  int& b = i2;
  int&& c = 111;

  fcn2(f1, i1, 42);//因为42为右值，所以fcn2的T2为右值，如果不加forward，把T2的形参传给另一个函数时，它就变成了左值，但是函数f1的参数时右值，这时，编译就不过了。
  std::cout << i1 << ", " << i2 << std::endl;

}


##c++ 类的默认八种函数
https://www.cnblogs.com/lsgxeva/p/7668200.html

class MyClass
{
public:
    MyClass(const char * str = nullptr);  // 默认带参构造函数 // 默认构造函数指不带参数或者所有参数都有缺省值的构造函数
    ~MyClass(void);  // 默认析构函数
    MyClass(const MyClass &);  // 默认拷贝构造函数
    MyClass & operator =(const MyClass &);  // 默认重载赋值运算符函数
    MyClass * operator &();  // 默认重载取址运算符函数
    MyClass const * operator &() const;  // 默认重载取址运算符const函数
    MyClass(MyClass &&);  // 默认移动构造函数
    MyClass & operator =(MyClass &&);  // 默认重载移动赋值操作符函数

private:
    char *m_pData;
};

// 默认带参构造函数
MyClass::MyClass(const char * str)
{
    if (!str)
    {
        m_pData = nullptr;
    } 
    else
    {
        this->m_pData = new char[strlen(str) + 1];
        strcpy(this->m_pData, str);
    }
    std::cout << "默认带参构造函数" << " this addr: " << this << std::endl;
}

 // 默认析构函数
MyClass::~MyClass(void)
{
    if (this->m_pData)
    {
        delete[] this->m_pData;
        this->m_pData = nullptr;
    }
    std::cout << "默认析构函数" << " this addr: " << this << std::endl;
}

// 默认拷贝构造函数
MyClass::MyClass(const MyClass &m)
{
    if (!m.m_pData)
    {
        this->m_pData = nullptr;
    } 
    else
    {
        this->m_pData = new char[strlen(m.m_pData) + 1];
        strcpy(this->m_pData, m.m_pData);
    }
    std::cout << "默认拷贝构造函数" << " this addr: " << this << std::endl;
}

// 默认重载赋值运算符函数
MyClass & MyClass::operator =(const MyClass &m)
{
    if ( this == &m ) {
        return *this;
    }
    delete[] this->m_pData;
    if (!m.m_pData)
    {
        this->m_pData = nullptr;
    } 
    else
    {
        this->m_pData = new char[strlen(m.m_pData) + 1];
        strcpy(this->m_pData, m.m_pData);
    }
    std::cout << "默认重载赋值运算符函数" << " this addr: " << this << std::endl;
    return *this;
}

// 默认重载取址运算符函数
MyClass * MyClass::operator &()
{
    std::cout << "默认重载取址运算符函数" << " this addr: " << this << std::endl;
    return this;
}

// 默认重载取址运算符const函数
MyClass const * MyClass::operator &() const
{
    std::cout << "默认重载取址运算符const函数" << " this addr: " << this << std::endl;
    return this;
}

// 默认移动构造函数
MyClass::MyClass(MyClass && m):
    m_pData(std::move(m.m_pData))
{
    std::cout << "默认移动构造函数" << std::endl;
    m.m_pData = nullptr;
}

// 默认重载移动赋值操作符函数
MyClass & MyClass::operator =(MyClass && m)
{
    if ( this == &m ) {
        return *this;
    }
    this->m_pData = nullptr;
    this->m_pData = std::move(m.m_pData);
    m.m_pData = nullptr;
    std::cout << "默认重载移动赋值操作符函数" << " this addr: " << this << std::endl;
    return *this;
}

void funA(MyClass a)
{
    std::cout << "调用funA函数" << " param addr: " << &a << std::endl;
}

void mytest1(void)
{
    std::cout << "mytest1 >>>>" << std::endl;
    MyClass myclass1; // 等价于 MyClass myclass1 = MyClass(); // 调用默认带参构造函数
    myclass1 = MyClass(); // MyClass()为右值，需要右值引用 // 先调用默认带参构造函数，然后调用默认重载取址运算符函数，最后调用默认重载移动赋值操作符函数
    std::cout << "<<<<< mytest1" << std::endl;
    // 析构两次 1: myclass1 = MyClass()中的MyClass() 2: MyClass myclass1
}

void mytest2(void)
{
    std::cout << "mytest2 >>>>" << std::endl;
    MyClass myclass1; // 等价于 MyClass myclass1 = MyClass(); // 调用默认带参构造函数
    MyClass myclass2(myclass1);  // 调用默认拷贝构造函数
    myclass2 = myclass1; // myclass2为左值，所以此操作为赋值操作，会调用默认重载取址运算符const函数，然后调用默认重载赋值运算符函数
    funA(myclass1); // 参数传值会导致赋值操作，会调用默认拷贝构造函数，然后funA函数调用默认重载取址运算符函数取得参数
    funA(std::move(myclass1)); // funA函数的参数现为右值，会调用默认移动构造函数，然后funA函数调用默认重载取址运算符函数取得参数
    // 在移动构造函数中对于基本类型所谓移动只是把其值拷贝，对于如string这类类成员来说才会真正的所谓资源移动
    std::cout << "<<<<< mytest2" << std::endl;
}

void mytest3(void)
{
    std::cout << "mytest3 >>>>" << std::endl;
    funA(MyClass()); // 会调用默认带参构造函数，生成该类的对象，然后funA函数调用默认重载取址运算符函数取得参数
    std::cout << "<<<<< mytest3" << std::endl;
    // 析构一次 1: funA(MyClass())中的MyClass()形成的对象，是在funA函数结束调用的时候，调用默认析构函数
}

void mytest(void)
{
    std::cout << "<<<<<<<<<<<<<<<<<<<<<<<<<" << std::endl;
    mytest1();
    mytest2();
    mytest3();
    std::cout << "<<<<<<<<<<<<<<<<<<<<<<<<<" << std::endl;
}

int main(int argc, char * argv[], char * envp[])
{
    mytest();
    system("pause");
    return 0;
}

<<<<<<<<<<<<<<<<<<<<<<<<<
mytest1 >>>>
默认带参构造函数 this addr: 0x7ffca6b2eed8
默认带参构造函数 this addr: 0x7ffca6b2eed0
默认重载取址运算符函数 this addr: 0x7ffca6b2eed0
默认重载移动赋值操作符函数 this addr: 0x7ffca6b2eed8
默认析构函数 this addr: 0x7ffca6b2eed0
<<<<< mytest1
默认析构函数 this addr: 0x7ffca6b2eed8
mytest2 >>>>
默认带参构造函数 this addr: 0x7ffca6b2eed8
默认拷贝构造函数 this addr: 0x7ffca6b2eed0
默认重载取址运算符const函数 this addr: 0x7ffca6b2eed8
默认重载赋值运算符函数 this addr: 0x7ffca6b2eed0
默认拷贝构造函数 this addr: 0x7ffca6b2eeb8
调用funA函数 param addr: 默认重载取址运算符函数 this addr: 0x7ffca6b2eeb8
0x7ffca6b2eeb8
默认析构函数 this addr: 0x7ffca6b2eeb8
默认移动构造函数
调用funA函数 param addr: 默认重载取址运算符函数 this addr: 0x7ffca6b2eeb0
0x7ffca6b2eeb0
默认析构函数 this addr: 0x7ffca6b2eeb0
<<<<< mytest2
默认析构函数 this addr: 0x7ffca6b2eed0
默认析构函数 this addr: 0x7ffca6b2eed8
mytest3 >>>>
默认带参构造函数 this addr: 0x7ffca6b2eed8
调用funA函数 param addr: 默认重载取址运算符函数 this addr: 0x7ffca6b2eed8
0x7ffca6b2eed8
默认析构函数 this addr: 0x7ffca6b2eed8
<<<<< mytest3
<<<<<<<<<<<<<<<<<<<<<<<<<


##shared_ptr
https://blog.csdn.net/weixin_34128839/article/details/93301673
https://blog.csdn.net/seamanj/article/details/50507470
https://blog.csdn.net/Richelieu_/article/details/83548000
https://blog.csdn.net/OKasy/article/details/79817591
https://www.cnblogs.com/mxp-neu/articles/8580062.html

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

##C++11多线程-原子操作
C++11多线程-原子操作(1)
https://www.jianshu.com/p/c0da859a7ce0
C++11多线程-原子操作(2)
https://www.jianshu.com/p/fc6fce6543a9
C++11多线程-内存模型
https://www.jianshu.com/p/7d237771dc94

##C++11泛型-函数模板
https://www.jianshu.com/p/949afb64be86

##C++11泛型 - 类模板
https://www.jianshu.com/p/91650b5312c3

##C++性能榨汁机之switch语句
https://zhuanlan.zhihu.com/p/38139553

##使用 gcc 命令把C语言程序反汇编 
https://www.cnblogs.com/yeyeck/p/9750396.html

#《编写高质量代码改善C++程序的150个建议》简要归纳：
《编写高质量代码改善C++程序的150个建议》简要归纳：
 
##第一部分 语法篇
 
##第1章   从C继承而来的
建议0：不用让main函数返回void
         main函数的返回类型是int，不是void或其它类型。
建议1：区分0的4种面孔
         (1)、整型0；(2)、空指针NULL，指针与int类型所占空间是一样的，都是32位；(3)、字符串结束标志’\0’；(4)、逻辑FALSE/false，FALSE/TRUE是int类型，而false/true是bool类型。
建议2：避免那些由运算符引发的混乱
         不要混淆=和==、&和&&、|与||这三对运算符之间的差别，针对=和==之间的问题，可以这样做：if (0 == nValue)。
建议3：对表达式计算顺序不用想当然
         (1)、针对操作符优先级，建议多写几个括号；(2)、注意函数参数和操作数的评估求值顺序问题。
建议4：小心宏#define使用中的陷阱
         (1)、用宏定义表达式时，要使用完备的括号：如 #define ADD(a, b) ((a)+(b))；(2)、使用宏时，不允许参数发生变化；(3)、用大括号将宏所定义的多条表达式括起来。
建议5：不要忘记指针变量的初始化
         (1)、可以将其初始化为空指针0(NULL)；(2)、对于全局变量来说，在声明的同时，编译器会悄悄完成对变量的初始化。
建议6：明晰逗号分隔表达式的奇怪之处
         (1)、在使用逗号分隔表达式时，C++会确保每个表达式都被执行，而整个表达式的值则是最右边表达式的结果；(2)、在C++中，逗号分隔表达式既可以用作左值，也可以用作右值。
建议7：时刻提防内存溢出
         在调用C语言字符串金典函数(strcpy、strcat、gets等)时，要从源代码开始就提高警惕，尽量追踪传入数据的流向，向代码中的每一个假设提出质疑。在访问数据时，注意对于边界数据要特殊情况特殊处理，还要对杜绝使用未初始化指针和失败后未置NULL的“野指针”。
建议8：拒绝晦涩难懂的函数指针
         函数指针在运行时的动态调用(例如函数回调)中应用广泛。但是直接定义复杂的函数指针会由于有太多的括号而使代码的可读性下降。使用typedef可以让函数指针更直观和易维护。
建议9：防止重复包含头文件
         为了避免重复包含头文件，建议在声明每个头文件时采用“头文件卫士”加以保护，比如采用如下的形式：
1.#ifndef__SOMEFILE_H__  
2.#define__SOMEFILE_H__  
3.……//声明、定义语句  
4.#endif  
建议10：优化结构体中元素的布局
         把结构体中的变量按照类型大小从小到大依次声明，尽量减少中间的填充字节。
建议11：将强制转型减到最少
         (1)、const_cast<T*>(a)：它用于从一个类中去除以下这些属性：const、volatile和__unaligned；(2)、dynamic_cast<T*>(a)：它将a值转换成类型为T的对象指针，主要用来实现类层次结构的提升；(3)、reinterpret_cast<T*>(a)：它能够用于诸如One_class*到Unrelated_class*这样的不相关类型之间的转换，因此它是不安全的；(4)、static_cast<T*>(a)：它将a的值转换为模板中指定的类型T，但是，在运行时转换过程中，它不会进行类型检查，不能确保转换的安全性。
建议12：优先使用前缀操作符
         对于整型和长整型的操作，前缀操作和后缀操作的性能区别通常是可以忽略的。对于用户自定义类型，优先使用前缀操作符。因为与后缀操作符相比，前缀操作符因为无须构造临时对象而更具性能优势。
建议13：掌握变量定义的位置与时机
         在定义变量时，要三思而后行，掌握变量定义的时机与位置，在合适的时机于合适的位置上定义变量。尽可能推迟变量的定义，直到不得不需要该变量为止；同时，为了减少变量名污染，提高程序的可读性，尽量缩小变量的作用域。
建议14：小心typedef使用中的陷阱
         区分typedef与#define之间的不同；不要用理解宏的思维方式对待typedef，typedef声明的新名称具有一定的封装性，更易定义变量。同时还要注意它是一个无“现实意义”的存储类关键字。
建议15：尽量不要使用可变参数
         编译器对可变参数函数的原型检查不够严格，所以容易引起问题，难于查错，不利于写出高质量的代码。所以应当尽量避免使用c语言方式的可变参数设计，而用C++中更为安全的方式来完美代替之(如多态等)。
建议16：慎用goto
         过度使用goto会使代码流程错综复杂，难以理清头绪。所以，如果不熟悉goto，不要使用它；如果已经习惯使用它，试着不去使用。
建议17：提防隐式转换带来的麻烦
提防隐式转换所带来的微妙问题，尽量控制隐式转换的发生；通常采用的方式包括：
(1)、使用非C/C++关键字的具名函数，用operator as_T()替换operator T()(T为C++数据类型)。
(2)、为单参数的构造函数加上explicit关键字。
建议18：正确区分void与void*
         Void是“无类型”，所以它不是一种数据类型；void*则为“无类型指针”，即它是指向无类型数据的指针，也就是说它可以指向任何类型的数据。Void发挥的真正作用是限制程序的参数与函数返回值：
(1)、如果函数没有返回值，那么应将其声明为void类型；
(2)、如果函数无参数，那么声明函数参数为void。
对于void*，(1)、任何类型的指针都可以直接赋值给它，无须强制转型；(2)、如果函数的参数可以是任意类型指针，那么应声明其参数为void*。
 
##第2章   从C到C++，需要做出一些改变
建议19：明白在C++中如何使用C
         若想在C++中使用大量现成的C程序库，就必须把它放到extern“C” {/* code */}中，extern “C”的作用就是告诉C++链接器寻找调用函数的符号时，采用C的方式。要实现在C++代码中调用C的代码，具体方式有以下几种：(1)、修改C代码的头文件，当其中含有C++代码时，在声明中加入extern “C”；(2)、在C++代码中重新声明一下C函数，在重新声明时添加上extern “C”；(3)、在包含C头文件时，添上extern “C”。
建议20：使用memcpy()系列函数时要足够小心
  要区分哪些数据对象是POD(传统C风格的数据类型，C的所有对象都是POD，对于任何POD对象，我们都可以放心大胆地使用memset()、memcpy()、memcmp()等函数对对象的内存数据进行操作)，哪些是非POD(C++的对象可能并不是一个POD，如动多态)，由于非POD对象的存在，在C++中使用memcpy()系列函数时要保持足够的小心。
建议21：尽量用new/delete代替malloc/free
         malloc与new之间的区别：(1)、new是C++运算符，而malloc则是C标准库函数；(2)、通过new创建的东西是具有类型的，而malloc函数返回的则是void*，需要进行强制转型；(3)、new可以自动调用对象的构造函数，而malloc不会；(4)、new失败是会调用new_handler处理函数，而malloc失败则直接返回NULL。Free与delete之间的区别：(1)、delete是C++运算符，free是C标准库函数；(2)、delete可以自动调用对象的析构函数，而malloc不会。另外，new/delete必须配对使用，malloc/free也一样。
建议22：灵活地使用不同风格的注释
         C风格的注释/* */与C++风格的注释//在C++语言中同时存在。
建议23：尽量使用C++标准的iostream
         建议使用#include <iostream>
建议24：尽量采用C++风格的强制转型
建议25：尽量用const、enum、inline替换#define
         对于简单的常量,应该尽量使用const对象或枚举类型数据，避免使用#define；对于形似函数的宏，尽量使用内联函数，避免使用#define。总之一句话，尽量将工作交给编译器，而不是预处理器。
建议26：用引用代替指针
         与指针不同，引用与地址没有关联，甚至不占任何存储空间。
 
第3章   说一说“内存管理”的那点事儿
在VC中，栈空间未初始化的字符默认是-52，补码是0xCC，两个0xCC，即0xCCCC在GBK编码中就是“烫”；堆空间未初始化的字符默认是-51，两个-51在GBK编码中就是“屯”。两者都是未初始化的内存。
建议27：区分内存分配的方式
         一个程序要运行，就必须先将可执行的程序加载到计算机内存里，程序加载完毕后，就可以形成一个运行空间，按照代码区、数据区、堆区、栈区进行布局。代码区存放的是程序的执行代码；数据区存放的是全局数据、常量、静态变量等；堆区存放的则是动态内存，供程序随机申请使用；而栈区则存放着程序中所用到的局部数据。这些数据可以动态地反应程序中对函数的调用状态，通过其轨迹也可以研究其函数机制。
         在C++中，数据区又被分成自由存储区、全局/静态存储区和常量存储区，再加上堆区、栈区，也就是说，内存被分成了5个区。(1)、栈区：在执行函数时，函数内局部变量的存储单元都可以在栈上创建，函数执行结束时这些存储单元将自动被释放。栈内存分配运算内置于处理器的指令集中，效率很高，但是所分配的内存容量有限；(2)、堆区：堆就是那些由new分配的内存块，其释放编译器不会管它，而是由我们的应用程序控制它，一般一个new就对应于一个delete，如果程序员没有释放掉，那么在程序结束后，操作系统就会自动回收；(3)、自由存储区：是那些由malloc等分配的内存块，它和堆十分相似，不过它是用free来结束自己生命的；(4)、全局/静态存储区：全局变量和静态变量被分配到同一块内存中，在以前的C语言中，全局变量又分为初始化的和未初始化的，在C++里面没有作此区分，它们共同占用同一块内存区；(5)、常量存储区：这是一块比较特殊的存储区，里面存放的是常量，不允许修改。
         堆与栈的区别：(1)、管理方式不同：对于栈来讲，它是由编译器自动管理的，无须我们手工控制；对于堆来说，它的释放工作由程序员控制，容易产生memory leak；(2)、空间大小不同：一般来讲在32位系统下，堆内存可以达到4GB的空间，从这个角度来看堆内存几乎是没有什么限制的。但是对于栈来讲，一般都是有一定空间大小的；(3)、碎片问题：对于堆来讲，频繁的new/delete势必会造成内存空间的不连续，从而产生大量的碎片，使程序效率降低。对于栈来讲，则不存在这个问题，其原因还要从栈的特殊数据结构说起。栈是一个具有严明纪律的队列，其中的数据必须遵循先进后出的规则，相互之间紧密排列，绝不会留给其他数据可插入之空隙，所以永远都不可能有一个内存块从栈中间弹出，它们必须严格按照一定的顺序一一弹出；(4)、生成方向：对于堆来讲，其生长方向是向上的，也就是向着内存地址增加的方向增长；对于栈来讲，它的生长方向是向下的，是向着内存地址减小的方向增长的；(5)、分配方式：堆都是动态分配的，没有静态分配的堆。栈有两种分配方式：静态分配和动态分配。静态分配是编译器完成的，比如局部变量的分配。动态分配由alloca函数完成，但是栈的动态分配和堆是不同的，它的动态分配是由编译器进行释放的，无须我们手工实现；(6)、分配效率：栈是机器系统提供的数据结构，计算机会在底层对栈提供支持：它会分配专门的寄存器存放栈的地址，而且压栈出栈都会有专门的指令来执行，这就决定了栈的效率比较高。堆则是C/C++函数库提供的，它的机制很复杂，例如为了分配一块内存，库函数会按照一定的算法在堆内存中搜索可用的足够大小的空间，如果没有足够大小的空间(可能是由于内存碎片太多)，则可能调用系统功能去增加程序数据段的内存空间，这样就有机会分到足够大小的内存了，然后返回。显然，堆的效率比栈要低得多。
建议28：new/delete与new[]/delete[]必须配对使用
         由于内置数据类型没有构造、析构函数，所以在针对内置数据类型时，释放内存使用delete或delete[]的效果都是一样的。
建议29：区分new的三种形态
         (1)、如果是在堆上建立对象，那么应该使用new operator，它会为你提供最为周全的服务；(2)、如果仅仅是分配内存，那么应该调用operator new，但初始化不在它的工作职责之内。如果你对默认的内存分配过程不满意，想单独定制，重载operator new是不二选择；(3)、如果想在一块已经获得的内存里建立一个对象，那就应该用placement new。但是通常情况下不建议使用，除非是在某些对时间要求非常高的应用中，因为相对于其他两个步骤，选择合适的构造函数完成对象初始化是一个时间相对较长的过程。
建议30：new内存失败后的正确处理
         当使用new申请一块内存失败时，抛出异常std::bad_alloc是C++标准中规定的标准行为，所以推荐使用try{p=new int[SIZE];} catch(std::bad_alloc) {…} 的处理方式。但是在一些老旧的编译器中，却不支持该标准，它会返回NULL，此时具有C传统的Test_for_NULL代码形式便起了作用。所以，要针对不同的情形采取合理的处置方式。
建议31：了解new_handler的所作所为
         在使用operatornew申请内存失败后，编译器并不是不做任何的努力直接抛出std::alloc异常，在这之前，它会调用一个错误处理函数(这个函数被称为new-handler)，进行相应的处理。通常，一个好的new-handler函数的处理方式必须遵循以下策略之一：(1)、使更大块内存有效；(2)、装载另外的new-handler；(3)、卸载new-handler；(4)、抛出异常；(5)、无返回。
建议32：借助工具检测内存泄露问题
         内存泄露一般指的是堆内存的泄露。检测内存泄露的关键是能截获对分配内存和释放内存的函数的调用。通过截获的这两个函数，我们就能跟踪每一块内存的生命周期。每当成功分配一块内存时，就把它的指针加入一个全局的内存链中；每当释放一块内存时，再把它的指针从内存链中删除。这样当程序运行结束的时候，内存链中剩余的指针就会指向那些没有被释放的内存。这就是检测内存泄露的基本原理。
         检测内存泄露的常用方法有如下几种：(1)、MS C-Runtime Library内建的检测功能，要在非MFC程序中打开内存泄露的检测功能非常容易，只须在程序的入口处添加以下代码：
_CrtSetDbgFlag(_CrtSetDbgFlag(_CRTDBG_REPORT_FLAG)| _CRTDBG_LEAK_CHECK_DF);
(2)、外挂式的检测工具：MS下BoundsChecker或Insure++；Linux下RationalPurify或Valgrind.
建议33：小心翼翼地重载operator new/operator delete
         通过重载operator new 和 operatordelete的方法，可以自由地采用不同的分配策略，从不同的内存池中分配不同的类对象。但是是否选择重载operator new/delete 一定要深思熟虑。
建议34：用智能指针管理通过new创建的对象
         智能指针auto_ptr，要使用它，需要包含memory头文件：(1)、auto_ptr对象不可作为STL容器的元素；(2)、auto_ptr缺少对动态配置而来的数组的支持；(3)、auto_ptr在被复制的时候会发生所有权转移。
建议35：使用内存池技术提高内存申请效率与性能
         经典的内存池技术，是一种用于分配大量大小相同的小对象的技术。通过该技术可以极大地加快内存分配/释放过程。内存池技术通过批量申请内存，降低了内存申请次数，从而节省了时间。
 
##第4章   重中之重的类
建议36：明晰class与struct之间的区别
         C中的struct是一种数据类型。C++中struct被看成是一个对象，它可以包含函数，可以拥有构造函数、析构函数，同样拥有继承等能力。
C++中的class与struct差别：
(1)、class和struct如果定义了构造函数，就都不能用大括号进行初始化了；如果没有定义，struct可以用大括号初始化，而class只有在所有成员变量全是public的情况下，才可以用大括号进行初始化；
(2)、关于默认访问权限：class中默认的成员访问权限是private的，而struct中则是public的；
(3)、关于继承方式：class继承默认是private继承，而struct默认是public继承
建议37：了解C++悄悄做的那些事
         对于类，编译器会悄悄地完成很多事：隐士产生一个类的默认构造函数，拷贝构造函数，拷贝赋值运算符和析构函数。
建议38：首选初始化列表实现类成员的初始化
         类成员的初始化可采用两种形式来完成：在构造函数体重赋值完成和用初始化类成员列表完成：(1)、const成员变量只能用成员初始化列表来完成初始化，而不能在构造函数内被赋值；(2)、如果类B中含有A类型的成员变量，而类A中又禁止了赋值操作，此时要想顺利地完成B中成员变量的初始化，就必须采用初始化列表方式。即使没有禁用赋值操作，还是不推荐采用函数体内的赋值初始化方式。因为这种方式存在着两种问题。第一，比起初始化列表，此方式效率偏低；第二，留有错误隐患。
         对于初始化列表，初始化的顺序与构造函数中的赋值方式不同，初始化列表中成员变量出现的顺序并不是真正初始化的顺序，初始化的顺序取决于成员变量在类中的声明顺序。只有保证成员变量声明的顺序与初始化列表顺序一致才能真正保证其效率。
建议39：明智地拒绝对象的复制操作
         在某些需要禁止对象复制操作的情形下，可以将这个类相应的拷贝构造函数、赋值操作符operator = 声明为private，并且不要给出实现。或者采用更简单的方法：使用boost::noncopyable作为基类。
建议40：小心，自定义拷贝函数
         如果类内部出现了动态配置的资源，我们就不得不自定义实现其拷贝函数了。在自定义拷贝函数时，应该保证拷贝一个对象的All Parts:所有数据成员及所有的基类部分。
建议41：谨防因构造函数抛出异常而引发的问题
         判断构造对象成功与否，解决办法：抛出一个异常。构造函数抛出异常会引起对象的部分构造，因为不能自动调用析构函数，在异常发生之前分配的资源将得不到及时的清理，进而造成内存泄露问题。所以，如果对象中涉及了资源分配，一定要对构造之中可能抛出的异常做谨慎而细致的处理。
建议42：多态基类的析构函数应该为虚
         虚函数的最大目的就是允许派生类定制实现。所以，用基类指针删除一个派生类对象时，C++会正确地调用整个析构链，执行正确的行为，以销毁整个对象。在实际使用虚析构函数的过程中，一般要遵守以下规则：当类中包含至少一个虚函数时，才将该类的析构函数声明为虚。因为一个类要作为多态基类使用时，它一定会包含一个需要派生定制的虚函数。相反，如果一个类不包含虚函数，那就预示着这个类不能作为多态基类使用。同样，如果一个类的析构函数非虚，那你就要顶住诱惑，决不能继承它，即使它是“出身名门”。比如标准库中的string、complex、以及STL容器。
         多态基类的析构函数应该是virtual的，也必须是virtual的，因为只有这样，虚函数机制才会保证派生类对象的彻底释放；如果一个类有一个虚函数，那么它就该有一个虚析构函数；如果一个类不被设计为基类，那么这个类的析构就应该拒绝为虚。
建议43：绝不让构造函数为虚
         虚函数的工作机制：虚函数的多态机制是通过一张虚函数表来实现的。在构造函数调用返回之前，虚函数表尚未建立，不能支持虚函数机制，所以构造函数不允许设为虚。
建议44：避免在构造/析构函数中调用虚函数
         成员函数、包括虚成员函数，都可以在构造、析构的过程中被调用。当一个虚函数被构造函数(包括成员变量的初始化函数)或者析构函数直接或间接地调用时，调用对象就是正在构造或者析构的那个对象。其调用的函数是定义于自身类或者其基类的函数，而不是其派生类或者最低派生类的其他基类的重写函数。
如果在构造函数或析构函数中调用了一个类的虚函数，那它们就变成普通函数了，失去了多态的能力。
建议45：默认参数在构造函数中给你带来的喜与悲
         合理地使用默认参数可以有效地减少构造函数中的代码冗余，让代码简洁而有力。但是如果不够小心和谨慎，它也会带来构造函数的歧义，增加你的调试时间。
建议46：区分Overloading、Overriding、Hiding之间的差异
1、重载(Overloading)：是指同一作用域的不同函数使用相同的函数名，但是函数的参数个数或类型不同；
2、重写(Overriding)：是指在派生类中对基类中的虚函数重新实现，即函数名和参数都一样，只是函数的实现体不一样，派生类对基类中的操作进行个性化定制就是重写。
重写需要注意的问题：
(1)、函数的重写与访问层级(public、private、protected)无关；
(2)、const可能会使虚成员函数的重写失效；
(3)重写函数必须和原函数具有相同的返回类型；
3、隐藏(Hiding)：是指派生类中的函数屏蔽基类中具有相同名字的非虚函数。
建议47：重载operator=的标准三步走
         1、不要让编译器帮你重载赋值运算符；2、一定要检查自赋值；3、赋值运算符重载需返回*this的引用，引用之于对象的优点在于效率，为了能够更加灵活地使用赋值运算符，选择返回引用绝对是明智之举；4、赋值运算符重载函数不能被继承。如果需要给类的数据成员动态分配空间，则必须实现赋值运算符。
建议48：运算符重载，是成员函数还是友元函数
         运算符重载的四项基本原则：(1)、不可臆造运算符；(2)、运算符原有操作数的个数、优先级和结合性不能改变；(3)、操作数中至少一个是自定义类型；(4)、保持重载运算符的自然含义。
         运算符的重载可采用两种形式：成员函数形式和友元函数形式。(1)、重载为成员函数时，已经隐含了一个参数，它就是this指针；对于双目运算符，参数仅有一个；(2)、当重载友元函数时，将不存在隐含的参数this指针；如果运算符被重载为友元函数，那么它就获得一种特殊的属性，能够接受左参数和右参数的隐式转换，如果是成员函数版的重载则只允许右参数的隐式转换。一般说来，建议遵守一个不成文的规定：对双目运算符，最好将其重载为友元函数，因为这样更方便些；而对于单目运算符，则最好重载为成员函数。
建议49：有些运算符应该成对实现
         为了更好地适应使用习惯，很多运算符重载时最好成对实现，比如==与!=、<与>、<=与>=、+与+=、-与-=、*与*=、/与/=。
建议50：特殊的自增自减运算符重载
         后缀操作重载时返回值应该为一个const对象。
建议51：不要重载operator&&、operator||以及operator,
         “&&”、“||”、“,”(逗号运算符)都具有较为特殊的行为特性，重载会改变运算符的这些特性，进而影响原有的习惯，所以不要去重载这三个可以重载的运算符。
建议52：合理地使用inline函数来提高效率
         内联函数具有与宏定义相同的代码效率，但在其他方面却要优于宏定义。因为内联函数还遵循函数的类型和作用域规则。内联函数一般情况下都应该定义在头文件中。内联函数的定义分为两种方式：(1)、显示方式：在函数定义之前添加inline关键字，内联函数只有和函数体声明放在一起时inline关键字才具有效力；(2)、隐式方式：将函数定义于类的内部。一个给定的函数是否得到内联，很大程度上取决于你正在使用的编译器。
使用内联函数应该注意：
(1)、内联函数的定义必须出现在内联函数第一次被调用之前。所以，它一般会置于头文件中；
(2)、在内联函数内不允许用循环语句和开关语句，函数不能过于复杂；
(3)、依据经验，内联函数只适合于只有1~5行的小函数；
(4)、对于内存空间有限的机器而言，慎用内联。过分地使用内联会造成函数代码的过度膨胀，会占用太多空间；
(5)、不要对构造/析构函数进行内联；
(6)、大多开发环境不支持内联调试，所以为了调试方便，不要将内联优化放在调试阶段之前。
建议53：慎用私有继承
私有继承会使基类的所有东西(包括所有的成员变量与成员函数)在派生类中变成private的，也就是说基类的全部在派生类中都只能作为实现细节，而不能成为接口。私有继承意味着“只有implementation 应该被继承，interface应该被忽略”，代表着是“is-implemented-in-terms-of”的内在关系。通常情况下，这种关系可以采用组合的方式来实现，并提倡优先使用组合的方案。但是如果存在虚函数和保护成员，就会使组合方案失效，那就应使用私有继承。
建议54：抵制MI的糖衣炮弹
MI(多重继承)意味着设计的高复杂性、维护的高难度性，尽量少使用MI。
建议55：堤防对象切片
多态的实现必须依靠指向同一类族的指针或引用。否则，就可能出现著名的对象切片(Object Slicing)问题。所以，在既有继承又有虚函数的情况下，一定要提防对象切片问题。
建议56：在正确的场合使用恰当的特性
(1)、虚函数：虚函数机制的实现是通过虚函数表和指向虚函数表的指针来完成的。关键字virtual告诉编译器该函数应该实现晚绑定，编译器对每个包含虚函数的类创建虚函数表VTable，以放置类的虚函数地址。编译器密码放置了指向虚函数表的指针VPtr，当多态调用时，它会使用VPtr在VTable表中查找要执行的函数地址；(2)、多重继承：对于多重继承来说，对象内部会有多个VPrt，所以这就使偏移量计算变得复杂了，而且会使对象占用的空间和运行时开销都变大；(3)、虚基类：它与多重继承的情况类似，因为虚基类就是为了多重继承而产生的；(4)、运行时类型检测(RTTI)：是我们在程序运行时得到对象和类有关信息的保证。
建议57：将数据成员声明为private
 将数据成员声明为private是具有相当充分的理由的：
(1)、实现数据成员的访问控制；
(2)、在将来时态下设计程序，为之后的各种实现提供弹性；
(3)、保持语法的一致性。
建议58：明晰对象的构造与析构的顺序
         (1)、对象的构造都是从类的最根处开始的，由深及浅，先基类后子类，层层构造，这个顺序不能改变。如果含有多个基类，那么就按照声明顺序由前及后执行。析构函数则严格按照构造的逆序执行；(2)、成员对象构造函数的调用顺序与成员对象的声明顺序严格一致，析构顺序是构造顺序的严格逆序。这是因为类的声明是绝对唯一的，而类的构造函数可以有多个，所以按照声明才会使析构函数得到唯一的逆序；(3)、如果继承遇到成员对象，基类构造函数依然会被首先调用，然后调用成员对象的构造函数。
建议59：明了如何在主调函数启动前调用函数
         如果想在主程序main启动之前调用某些函数，调用全局对象的构造函数绝对是一个很不错的方法。因为从概念上说，全局对象是在程序开始前已经完成了构造，而在程序执行之后才会实施析构。
 
##第5章   用好模板，向着GP(泛型编程)开进
建议60：审慎地在动、静多态之间选择
虚函数机制配合继承机制，生效于运行期，属于晚绑定，是动多态；而模板将不同的行为和单个泛化记号相关联发生在编译期，属于早绑定，被称为静多态。
(1)、动多态：它的技术基础是继承机制和虚函数，它在继承体系之间通过虚函数表来表达共同的接口；
(2)、静多态：它的技术基础是模板。与动多态相比，静多态始终在和参数“较劲儿”，它适用于所有的类，与虚函数无关。从应用形式上看，静多态是发散式的，让相同的实现代码应用于不同的场合；动多态是收敛式的，让不同的实现代码应用于相同的场合。从思维方式上看，前者是泛型式编程风格，它看重的是算法的普适性；后者是对象式编程风格，它看重的是接口与实现的分离度。
两者区别：
(1)、动多态的函数需要通过指针或引用传参，而静多态则可以传值、传指针、传引用等，“适应性”更强；
(2)、在性能上，静多态优于动多态，因为静多态无间接访问的迂回代码，它是单刀直入的；
(3)、因为实现多态的先后顺序不同，所以如果出现错误，它们抛出错误的时刻也不一样，动多态会在运行时报错，而静多态则在编译时报错。
建议61：将模板的声明和定义放置在同一个头文件里
         模板类型不是一种实类型，它必须等到类型绑定后才能确定最终类型，所以在实例化一个模板时，必须要能够让编译器“看到”在哪里使用了模板，而且必须要看到模板确切的定义，而不仅仅是它的声明，否则将不能正常而顺利地产生编译代码。函数模板、类模板不同于一般的函数、类，它们不能像一般的方式那样进行声明与定义，标准要求模板的实例化与定义体必须放在同一翻译单元中。实现这一目标有三种方法(将模板的声明和定义都放置在同一个.h文件中；按照旧有的习惯性做法来处理，声明是声明，实现是实现，二者相互分离，但是需要包含头文件的地方做一些改变，如，在使用模板时，必须用#include “Temp.cpp”替换掉#include “Temp.h”；使用关键字export来定义具体的模板类对象和模板函数)，但是最优策略还是：将模板的声明和定义都放置在同一个.h文件中，虽然在某种程度上这破坏了代码的优雅性。
建议62：用模板替代参数化的宏函数
参数化的宏函数有着两个致命缺点：
(1)、缺乏类型检查；
(2)、有可能在不该进行宏替换的时候进行了替换，违背了作者的意图。模板是实现代码复用的一种工具，它可以实现类型参数化，达到让代码真正复用的目的。
建议63：区分函数模板与模板函数、类模板与模板类
         函数模板的重点在于“模板”两个字，前面的“函数”只是一个修饰词。其表示的是一个专门用来生产函数的模板。而模板函数重点在“函数”，表示的是用模板所生成的函数。函数模板的一般定义形式为：
         Template<class数据类型参数标识符>
         返回类型标识符 函数名(数据类型参数标识符 形参)
         {  //… …}
         将函数模板的模板参数实例化后会生成具体的函数，此函数就是模板函数。由函数模板所生成的模板函数的一般形式为：
         函数名<数据类型参数标识符>(数据类型参数标识符 形参)
         类模板是为类定义的一种模式，它使类中的一些数据成员和成员函数的参数或返回值可以取任意的数据类型。在类定义中，凡是采用标准数据类型的数据成员、成员函数的参数前面都要加上类型标识符，在返回类型前也要进行同样的处理。如果类中的成员函数要在类的声明之外定义，则它必须是模板函数。将类模板的模板参数实例化后生成的具体类，就是模板类。函数模板和类模板处于实例化之前，而模板函数或模板类则在实例化之后。
建议64：区分继承与模板
         模板的长处在于处理不同类型间“千篇一律”的操作。相较于类继承，这些类不必具有什么相同的性质。
 
##第6章   让神秘的异常处理不再神秘
建议65：使用exception来处理错误
         异常能：
(1)、增强程序的健壮性；
(2)、使代码变得更简洁优美、更易维护；
(3)、错误信息更灵活、丰富。
建议66：传值throw异常，传引用catch异常
         throw byvalue, catch by reference
建议67：用“throw;”来重新抛出异常
         对于异常的重新抛出，需要注意：
(1)、重新抛出的异常对象只能出现在catch块或catch调用的函数中；
(2)、如果在处理代码不执行时碰到“throw ;”语句，将会调用terminate函数。
建议68：了解异常捕获与函数参数传递之间的差异
         异常与函数参数的传递之间差异：
(1)、控制权；
(2)、对象拷贝的次数；
(3)、异常类型转换；
(4)、异常类型匹配。
建议69：熟悉异常处理的代价
         异常处理在带来便利的同时，也会带来时间和空间上的开销，使程序效率降低，体积增大，同时会加大代码调试和管理的成本。
建议70：尽量保证异常安全
         如果采用了异常机制，请尽量保证异常安全：努力实现强保证，至少实现基本保证。
 
##第7章   用好STL这个大轮子
建议71：尽量熟悉C++标准库
         C++标准库主要包含的组件：
(1)、C标准函数库；
(2)、输入/输出(input/output)；
(3)、字符串(string)；
(4)、容器(containers)；
(5)、算法(algorithms)；
(6)、迭代器(iterators)；
(7)、国际化(internationalization)；
(8)、数值(numerics)；
(9)、语言支持(languagesupport)；
(10)、诊断(diagnostics)；
(11)、通用工具(general utilities)。
字符串、容器、算法、迭代器四部分采用了模板技术，一般被统称为STL(Standard Template Library,即标准模板库)。
         在C++标准中，STL被组织成了13个头文件：
<algorithm>、<deque>、<functional>、<iterator>、<vector>、<list>、<map>、<memory>、<numeric>、<queue>、<set>、<stack>、<utility>。
建议72：熟悉STL中的有关术语
         (1)、容器：是一个对象，它将对象作为元素来存储；
(2)、泛型(Genericity)：泛型就是通用，或者说是类型独立；
(3)算法：就是对一个对象序列所采取的某些操作，例如std::sort()、std::copy()、std::remove()；
(4)、适配器(Adaptor)：是一个非常特殊的对象，它的作用就是使函数转化为函数对象，或者是将多参数的函数对象转化为少参数的函数对象；(5)、O(h)：它是一个表示算法性能的特殊符号，在STL规范中用于表示标准库算法和容器操作的最低性能极限；
(6)、迭代器：是一种可以当做通用指针来使用的对象，迭代器可以用于元素遍历、元素添加和元素删除。
建议73：删除指针的容器时避免资源泄露
         STL容器虽然智能，但尚不能担当删除它们所包含指针的这一责任。所以，在要删除指针的容器时须避免资源泄露：或者在容器销毁前手动删除容器中的每个指针，或者使用智能引用计数指针对象(比如Boost的shared_ptr)来代替普通指针。
建议74：选择合适的STL容器
         容器分为：(1)、标准STL序列容器：vector、string、deque和list；(2)、标准STL关联容器：set、multiset、map和multimap；(3)、非标准序列容器：slist(单向链表)和rope(重型字符串)；(4)、非标准关联容器：hash_set、hash_multiset、hash_map和hash_multimap；(5)、标准非STL容器：数组、bitset、valarray、stack、queue和priority_queue。
建议75：不要在STL容器中存储auto_ptr对象
         auto_ptr是C++标准中提供的智能指针，它是一个RAII对象，它在初始化时获得资源，析构时自动释放资源。C++标准中规定：STL容器元素必须能够进行拷贝构造和赋值操作。禁止在STL容器中存储auto_ptr对象原因有两个：(1)、auto_ptr拷贝操作不安全，会使原指针对象变NULL；(2)、严重影响代码的可移植性。
建议76：熟悉删除STL容器中元素的惯用法
         (1)、删除容器中具有特定值的元素：如果容器是vector、string或deque，使用erase-remove的惯用法(remove只会将不应该删除的元素前移，然后返回一个迭代器，该迭代器指向的是那个应该删除的元素，所以如果要真正删除这一元素，在调用remove之后还必须调用erase)；如果容器时list，使用list::remove；如果容器是标准关联容器，使用它的erase成员函数；(2)、删除容器中满足某些条件的所有元素：如果容器是vector、string或deque，使用erase-remove_if惯用法；如果容器是list，使用list::remove_if；如果容器是标准关联容器，使用remove_copy_if & swap组合算法，或者自己写一个遍历删除算法。
建议77：小心迭代器的失效
         迭代器是一个对象，其内存大小为12(sizeof(vector<int>::iterator)，vs2010,32bit)。引起迭代器失效的最主要操作就是插入、删除。对于序列容器(如vector和deque)，插入和删除操作可能会使容器的部分或全部迭代器失效。因为vector和deque必须使用连续分配的内存来存储元素，向容器中添加一个元素可能会导致后面邻接的内存没有可用的空闲空间而引起存储空间的重新分配。一旦这种情况发生，容器中的所有的迭代器就会全部失效。
建议78：尽量使用vector和string代替动态分配数组
         相较于内建数组，vector和string具有几方面的优点：(1)、它们能够自动管理内存；(2)、它们提供了丰富的接口；(3)、与C的内存模型兼容；(4)、集众人智慧之大成。
建议79：掌握vector和string与C语言API的通信方式
         使用vector::operator[]和string::c_str是实现STL容器与C语言API通信的最佳方式。
建议80：多用算法调用，少用手写循环
         用算法调用代替手工编写的循环，具有几方面的优点：(1)、效率更高；(2)、不易出错；(3)、可维护性更好。
 
##第二部分 编码习惯和规范篇
 
##第8章   让程序正确执行
建议81：避免无意中的内部数据裸露
         对于const成员函数，不要返回内部数据的句柄，因为它会破坏封装性，违反抽象性，造成内部数据无意中的裸露，这会出现很多“不可思议”的情形，比如const对象的非常量性。
建议82：积极使用const为函数保驾护航
         const的真正威力体现在几个方面：
(1)、修饰函数形式的参数：const只能修饰输入参数，对于内置数据类型的输入参数，不要将“值传递”的方式改为“const 引用传递”；
(2)、修饰函数返回值；
(3)、修饰成员函数：用const修饰成员函数的目的是提高程序的健壮性。const成员函数不允许对数据成员进行任何修改。
         关于const成员函数，须遵循几个规则：
(1)、const对象只能访问const成员函数，而非const对象可以访问任意的成员函数；
(2)、const对象的成员是不可修改的，然而const对象通过指针维护的对象却是可以修改的；
(3)、const成员函数不可以修改对象的数据，不管对象是否具有const性质。
建议83：不要返回局部变量的引用
         局部变量的引用是一件不太靠谱的事儿，所以尽量避免让函数返回局部变量的引用。同时也不要返回new生成对象的引用，因为这样会让代码层次混乱，让使用者苦不堪言。
建议84：切忌过度使用传引用代替传对象
         相较于传对象，传引用的优点：它减少了临时对象的构造与析构，所以更具效率。但须审慎地使用传引用替代传对象，必须传回内部对象时，就传对象，勿传引用。
建议85：了解指针参数传递内存中的玄机
         用指针参数传回一块动态申请的内存，是很常见的一种需求。然而如果不甚小心，就很容易造成严重错误：程序崩溃+内存泄露，解决之道就是用指针的指针来传递，或者换种内存传递方式，用返回值来传递。
建议86：不要讲函数参数作为工作变量
         工作变量，就是在函数实现中使用的变量。应该防止将函数参数作为工作变量，而对于那些必须改变的参数，最好先用局部变量代替之，最后再将该局部变量的内容赋给该参数，这样在一定程度上保护了数据的安全。
建议87：躲过0值比较的层层陷阱
(1)、0在不在该类型数据的取值范围内？
(2)、浮点数不存在绝对0值，所以浮点零值比较需特殊处理；
(3)区分比较操作符==与赋值操作符=，切忌混淆。
建议88：不要用reinterpret_cast去迷惑编译器
         reinterpret_cast，简单地说就是保持二进制位不变，用另一种格式来重新解释，它就是C/C++中最为暴力的类型转换，所实现的是一个类型到一个毫不相关、完全不同类型的映射。reiterpret_cast仅仅重新解释了给出对象的比特模型，它是所有类型转换中最危险的。尽量避免使用reinterpret_cast，除非是在其他转换都无效的非常情形下。
建议89：避免对动态对象指针使用static_cast
         在类层次结构中，用static_cast完成基类和子类指针(或引用)的下行转换是不安全的。所以尽量避免对动态对象指针使用static_cast，可以用dynamic_cast来代替，或者优化设计，重构代码。
建议90：尽量少应用多态性数组
         多态性数组一方面会涉及C++时代的基类指针与派生类指针之间的替代问题，同时也会涉及C时代的指针运算，而且常会因为这二者之间的不协调引发隐蔽的Bug。
建议91：不要强制去除变量的const属性
         在C++中，const_cast<T*>(a)一般用于从一个类中去除以下这些属性：const、volatile和_unaligned.强制去除变量的const属性虽然可以带来暂时的便利，但这不仅增加了错误修改变量的几率，而且还可能会引发内存故障。
 
##第9章   提高代码的可读性
建议92：尽量使代码版面整洁优雅
(1)、避免代码过长；
(2)、代码缩进和对齐；
(3)、空行分隔段落；
(4)、使用空格；
(5)、语句行。
建议93：给函数和变量起一个“能说话”的名字
(1)、名称必须直观，可望文生义，不必解码；
(2)、长度要符合“min_length && max_information”(最小名长度最大信息量)的原则；
(3)、与整体风格保持一致；
(4)、变量名称应该是一个“名词”，或者是“形容词+名词”；而函数名称应该是“动词+名词”的组合；
(5)、杜绝仅靠大小写来区分的名称标示符；
(6)、变量名之前附加前缀用来识别变量类型；
(7)、C++类或结构的成员变量附加前缀“m_”；全局变量名称附加前缀“g_”；
(8)、单字符变量只能用作循环变量；
(9)、类名采用“C+首字母大写的单词”形式来命名。
建议94：合理地添加注释
(1)、使用统一的注释方法为每个层次的代码块添加注释；
(2)、避免不必要的注释；
(3)、掌握代码注释量的一个度；
(4)、边写代码加边注释；
(5)、注释要简明而准确；
(6)、注意特有标签的特有作用。
建议95：为源代码设置一定的目录结构
         如果一个软件所涉及的文件数目比较多，通常要将其进行划分，为其设置一定的目录结构，以便于维护，如include、lib、src、doc、release、debug。
建议96：用有意义的标识代替Magic Numbers
         用宏或常量替代信息含量较低的MagicNumbers，绝对是一个好习惯，这样可提高代码的可读性与可维护性。
建议97：避免使用“聪明的技巧”
建议98：运算符重载时坚持其通用的含义
建议99：避免嵌套过深与函数过长
建议100：养成好习惯，从现在做起
 
##第10章   让代码运行得再快些
建议101：用移位实现乘除法运算
         在大部分的C/C++编译器中，用移位的方法比直接调用乘除法子程序生成代码的效率要高。只要是乘以或除以一个整数常量，均可用移位的方法得到结果，如a=a*9可以拆分成a=a*(8+1)，即a=a(a<<3)+a。移位只对整数运算起作用。
建议102：优化循环，提高效率
         应当将最长的循环放在最内层，最短的循环放在最外层，以减少CPU跨切循环层的次数，提高效率。
建议103：改造switch语句
         对于case的值，推荐按照它们发生的相对频率来排序，把最可能发生的情况放在第一位，最不可能的情况放在最后。
建议104：精简函数参数
         函数在调用时会建立堆栈来存储所需的参数值，因此函数的调用负担会随着参数列表的增长而增加。所以，参数的个数会影响进栈出栈的次数，当参数很多的时候，这样的操作就会花费很长的时间。因此，精简函数参数，减少参数个数可以提高函数调用的效率。如果精简后的参数还是比较多，那么可以把参数列表封装进一个单独的类中，并且可以通过引用进行传递。
建议105：谨慎使用内嵌汇编
         汇编语言与其他高级语言相比，更接近机器语言，效率更高，所以在应用程序中如果碰到那些对时间要求苛刻的部分，可以采用汇编语言来重写。
建议106：努力减少内存碎片
         经常性地动态分配和释放内存会造成堆碎片，尤其是应用程序分配的是很小的内存块时。避免堆碎片：(1)、尽可能少地使用动态内存，在大多数情况下，可以使用静态或自动储存，或者使用STL容器，减少对动态内存的依赖；(2)、尽量分配和重新分配大块的内存块，降低内存碎片发生的几率。内存碎片会影响程序执行的效率。
建议107：正确地使用内联函数
         内联(inline)函数既能够去除函数调用所带来的效率负担，又能够保留一般函数的优点。只有当函数非常短小的时候使用inline才能得到预想中的效果。对于编译器来说，函数内联与否取决于以下关键性的因素：(1)、函数体的大小；(2)、是否有局部对象被声明；(3)、函数的复杂性(是否存在函数调用、递归等)。
建议108：用初始化取代赋值
         以用户初始化代替赋值，可以使效率得到较大的提升，因为这样可以避免一次赋值函数operator =的调用。因此，当我们在赋值和初始化之间进行选择时，初始化应该是首选。需要注意的是，对基本的内置数据类型而言，初始化和赋值之间是没有差异的，因为内置类型没有构造和析构过程。
建议109：尽可能地减少临时对象
         临时对象产生的主要情形及避免方法：(1)、参数：采用传常量引用或指针取代传值；(2)、前缀或后缀：优先采用前缀操作；(3)、参数转换：尽量避免这种转换；(4)、返回值：遵循single-entry/single-exit原则，避免同一个函数中存在多个return语句。
建议110：最后再去优化代码
         在进行代码优化之前，需要知道：(1)、算法是否正确；(2)、如何在代码优化和可读性之间进行选择；(3)、该如何优化：代码分析(profiling)工具；(4)、如何选择优化方向：先算法，再数据结构，最后才是实现细节。
 
##第11章   零碎但重要的其他建议
建议111：采用相对路径包含头文件
         一个“点”(“.\”)代表的是当前目录所在的路径，两个“点”(“..\”)代表的是相对于当前目录的上一次目录路径。
         当写#include语句时，推荐使用相对路径；此外，要注意使用比较通用的正斜线“/”，而不要使用仅在Windows下可用的反斜线“\”。
建议112：让条件编译为开发出力
         条件编译中的预处理命令主要包括：#if、#ifndef、#ifdef、#endif和#undef等，它们的主要功能是在程序编译时进行有选择性的挑选，注释掉一些指定的代码，以达到版本控制、防止对文件重复包含等目的。
建议113：使用.inl文件让代码整洁可读
         .inl文件是内联函数的源文件，.inl文件还可用于模板的定义。.inl文件可以将头文件与内联函数的复杂定义隔离开来，使代码整洁可读，如果将其用于模板定义，这一优点更加明显。
建议114：使用断言来发现软件问题
         断言assert，只会在调试模式下生成代码，而在Release版本中则直接无视之。
1.assert(pData!= NULL && “In Function DataProcess”)  
2.assert(0&& “MSG_HANDLE_ERROR”)  

建议115：优先选择编译和链接错误
         静态检查：编译器必须检查源程序是否符合源语言规定的语法和语义要求，静态检查的主要工作就是语义分析，它是独立于数据和控制流的，可信度相对较高，而且不会增加程序的运行时开销。
         动态检查：是在运行时刻对程序的正确性、安全性等做检查，比如内存不足、溢出、数组越界、除0等，这类检查对于数据和控制流比较依赖。
         C/C++语言属于一种静态语言。一个设计较好的C++程序应该是较少地依赖动态检查，更多地依赖静态检查。
建议116：不放过任何一条编译器警告
         强烈建议：(1)、把编译器的警告级别调至最高；(2)、不要放过编译器的任何一条警告信息。
建议117：尽量减少文件之间的编译依赖
         不要在头文件中直接包含要使用的类的头文件(除了标准库)，直接包含头文件这种方式相对简单方便，但是会耗费大量的编译时间。推荐使用类的前向声明来减少文件直接的编译依赖。用对类声明的依赖替代对类定义的依赖，这是减少编译依赖的原则。
         为了加快编译进程，减少时间的浪费，我们应该尽量减少头文件依赖，其中的可选方案包括前向声明、柴郡猫技术等。
建议118：不用在头文件中使用using
         名空间是C++提供的一种机制，可以有效地避免函数名污染。然而在应用时要十分注意：任何情况下都不应在头文件中使用“using namespace XXX”这样的语句，而应该在定义时直接用全称。
建议119：划分全局名空间避免名污染
         使用自己的名空间将全局名空间合理划分，会有效地减少名污染问题，因为，不要简单地将所有的符号和名称统统扔进全局名空间里。
 
##第三部分 程序架构和思想篇
 
##第12章   面向对象的类设计
建议120：坚持“以行为为中心”的类设计
         “以数据为中心”关注类的内部数据结构，习惯将private类型的数据写在前面，而将public类型的函数写在后面。
         “以行为为中心”关注的重心放在了类的服务和接口上，习惯将public类型的函数写在前面，而将private类型的数据写在后面。
建议121：用心做好类设计
         在设计一个类时，首先，类的设计就是数据类型的设计，在数据类型的设计中，(1)、类应该如何创建和销毁呢？这会影响到类的构造函数和析构函数的设计。首先应该确定类是否需要分配资源，如果需要，还要确定这些资源又该如何释放。(2)、类是否需要一个无参构造函数？如果需要，而恰恰此时这个类已经有了构造函数，那么我们就得显示地写一个。(3)、类需要复制构造函数吗？其参数上加上了const修饰吗？它是用来定义这个类传值(pass-by-value)的具体实现的。(4)、所有的数据成员是不是都已经在构造函数中完成了初始化呢？(5)、类需要赋值操作符吗？赋值操作符能正确地将对象赋给对象本身吗？它与初始化有什么不同？其参数上加上了const修饰吗？(6)、类的析构函数需要设置为virtual吗？(7)、类中哪些值得组合是合法的？合法值的限定条件是什么？在成员函数内部是否对变量值得合法性做了检查？其次，类的设计是对现实对象进行抽象的一个过程。再次，数据抽象的过程其实是综合考虑各方面因素进行权衡的一个过程。
建议122：以指针代替嵌入对象或引用
         设计类的数据成员时，可以有三种选择：(1)、嵌入对象；(2)、使用对象引用；(3)、使用对象指针。
         如果在类数据成员中使用到了自定义数据类型，使用指针是一个较为明智的选择，它有以下几方面的优点：(1)、成员对象类型的变化不会引起包含类的重编译；(2)、支持惰性计算，不创建不使用的对象，效率更高；(3)、支持数据成员的多态行为。
建议123：努力将接口最小化且功能完善
         类接口的目标是完整且最小。精简接口函数个数，使每一个函数都具有代表性，并且使其功能恰好覆盖class的智能，同时又可以获得接口精简所带来的好处：(1)、利于理解、使用，维护成本也相对较低；(2)、可以缩小头文件长度，并缩短编译时间。
建议124：让类的数据隐藏起来
         坚持数据封装，坚持信息隐藏，杜绝公有、保护属性的存在(数据成员私有、柴郡猫技术)。
建议125：不要让成员函数破坏类的封装性
         小心类的成员函数返回属性变量的“直接句柄”，它会破坏辛辛苦苦搭建维护的封装性，一种方法，将函数的返回值加上const修饰。
建议126：理解“virtual + 访问限定符”的深层含义
         virtual关键字是C++中用于实现多态的重要机制，其核心理念就是通过基类访问派生类定义的函数。
         (1)、基类中的一个虚拟私有成员函数，表示实现细节是可以被派生类修改的；
(2)、基类中的一个虚拟保护成员函数，表示实现细节是必须被派生类修改的；
(3)、基类中的一个虚拟公有成员函数，则表示这是一个接口，不推荐，建议用protected virtual 来替换。
建议127：谨慎恰当地使用友元机制
         通常说来，类中的私有成员一般是不允许外面访问的。但是友元可以超脱这条禁令，它可以访问该类的私有成员。所带来的最大好处就是避免了类成员函数的频繁调用，节约了处理器的开销，提高了程序的效率。但是，通常，大家认为“友元破坏了类的封装性”。采用友元机制，一般是基于这样的需求：一个类的部分成员需要对个别其他类公开。
建议128：控制对象的创建方式
         栈和堆是对象的主要分布区，它们对应着两种基本的对象创建方式：
以new方式手动管理的堆创建和只需声明就可使用的栈创建。
         控制对象的创建方式：
(1)、要求在堆中建立对象：为了执行这种限制，必须找到一种方法保证调用new是建立对象的唯一手段。非堆对象是在定义它时自动构造的，而且是在生存期结束时自动释放的。将析构函数声明为private，而构造函数保持为public；
(2)、禁止在堆中建立对象：要禁止调用new来建立对象，可以通过将operator new函数声明为private来实现。
建议129：控制实例化对象的个数
         当实例化对象唯一时，采用设计模式中的单件模式；当实例化对象为N(N>0)个时，设置计数变量是一个思路。
建议130：区分继承与组合
         (1)、继承：C++的“继承”特性可以提高程序的可复用性。继承规则：若在逻辑上B是一种A，并且A的所有功能和属性对B而言都有意义，则允许B继承A的功能和属性。继承易于修改或扩展那些被复用的实现。但它的这种“白盒复用”却容易破坏封装性，因为这会将父类的实现细节暴露给子类。当父类实现更改时，子类也不得不随之更改，所以，从父类继承来的实现将不能在运行期间进行改变；(2)、组合：在逻辑上表示的是“有一个(Hase-A)”的关系，即A是B的一部分。组合属于“黑盒”复用，被包含对象的内部细节对外是不可见的。所以，它的封装性相对较好，实现上的相互依赖性比较小。并且可以通过获取指向其他的具有相同类型的对象引用，在运行期间动态地定义组合。而其缺点就是致使系统中的对象过多。Is-A关系用继承表达，Has-A关系用组合表达，优先使用(对象)组合。
建议131：不要将对象的继承关系扩展至对象容器
         A是B的基类，B是一种A，但是B的容器却不能是这种A的容器。
建议132：杜绝不良继承
         在继承体系中，派生类对象必须是可以取代基类对象的。
建议133：将RAII作为一种习惯
         RAII(ResourceAcquisition Is Initialization)，资源获取即初始化，RAII是C++语言的一种管理资源、避免泄露的惯用方法。RAII的做法是使用一个对象，在其构造时获取资源，在对象生命周期中控制对象资源的访问，使之始终保持有效，最后再对象析构时释放资源。实现这种功能的类即采用了RAII方式，这样的类被称为封装类。
建议134：学习使用设计模式
         设计模式是用来“封装变化、降低耦合”的工具，它是面向对象设计时代的产物，其本质就是充分运用面向对象的三个特性(即：封装、继承和多态)，并进行灵活的组合。
建议135：在接口继承和实现继承中做谨慎选择
         在接口继承和实现继承之间进行选择时，需要考虑的一个因素就是：基类的默认版本。对于那些无法提供默认版本的函数接口我们选择函数接口继承；而对于那些能够提供默认版本的，函数实现继承就是最佳选择。
建议136：遵循类设计的五项基本原则
         (1)、单一职责原则(SRP)：一个类，最好只做一件事。SRP可以看作是低耦合、高内聚在面向对象原则上的引申；(2)、开闭原则(OCP)：对扩展开放，对更改关闭，应该能够不用修改原有类就能扩展一个类的行为；(3)、替换原则(LSP )：子类应当可以替换父类并出现在父类能够出现的任何地方。反过来则不成立，子类可以替换基类，但是基类不一定能替换子类；(4)、依赖倒置原则(DIP)：高层模块不依赖于底层模块，而是二者都依赖于抽象，即抽象不依赖于具体，具体依赖于抽象。依赖一定会存在类与类、模块与模块之间。当两个模块之间存在紧密的耦合关系时，最好的方法就是分离接口和实现：在依赖之间定义一个抽象的接口使得高层模块调用接口，底层模块实现接口的定义，从而有效控制耦合关系，达到依赖于抽象的设计目的；(5)、接口分离原则(ISP)：使用多个小的专门的接口，而不要使用一个大的总接口。接口有效地将细节和抽象隔离开来，体现了对抽象编程的一切好处，接口隔离强调接口的单一性。分离的手段主要有两种方式：一个是利用委托分离接口，另一个是利用多重继承分离接口。
 
##第13章  返璞归真的程序设计
建议137：用表驱动取代冗长的逻辑选择
         表驱动法(Table drivenmethod)，是一种不必用很多的逻辑语句(if或case)就可以把表中信息找出来的方法。它是一种设计模式，可用来代替复杂的if/else或switch-case逻辑判断。
建议138：为应用设定特性集
         对待C++高级特性的态度一定要谨慎，是否有必要使用多重继承、异常、RTTI、模板及模板元编程，一定要做一个审慎的评估，以便为应用选择合适的特征集，避免使用过分复杂的设计和功能，否则将会使得代码难于理解和维护。
建议139：编码之前需三思
         在让电脑运行你的程序之前，先让你的大脑编译运行。
建议140：重构代码
         重构无止境，重构你的代码，精雕细琢，千锤百炼。
建议141：透过表面的语法挖掘背后的语义
建议142：在未来时态下开发C++程序
         在未来时态下开发C++程序，需要考虑代码的可重用性、可维护性、健壮性，以及可移植性。
建议143：根据你的目的决定造不造轮子
         在编程语言中这些轮子表现为大量的通用类和库。在工程实践中，不要重复造轮子；而在学习研究中，鼓励重复造轮子。
建议144：谨慎在OO与GP之间选择
         面向对象(OO)和泛型编程(GP)是C++提供给程序员的两种矛盾的思考模式。OO是我吗难以割舍的设计原则，世界是对象的，我们面向对象分析、设计、编程；而泛型编程则关注于产生通用的软件组件，让这些组件在不同的应用场合都能很容易的重用。
建议145：让内存管理理念与时俱进
         学习STL allocator，更新内存管理理念。
建议146：从大师的代码中学习编程思想与技艺
         阅读代码需要方法：刚开始不要纠结于代码的细节，将关注的重点放在代码的高层结构上，理解代码的构建过程；之后，再有重点的深入研究，理解对象的构造，明晰算法的步骤，尝试着深入理解其中的繁杂细节。
建议147：遵循自然而然的C++风格
建议148：了解C++语言的设计目标与原则
建议149：明确选择C++的理由

#c函数字符串的库源码的实现方式。
本文说明的是c函数字符串的库源码的实现方式。
其中函数声明方式：
__cdecl 是C 声明的缩写（Declaration），是C语言默认的函数调用方法：所有参数从右到左依次入栈，参数由调用者清除，即手动清栈。被调用函数不会要求调用者传递多少参数，调用者传递过多或者过少的参数，甚至完全不同的参数都不会产生编译阶段的错误。
_stdcall 是StandardCall的缩写，是C++的标准调用方式：所有参数从右到左依次入栈，如果是调用类成员的话，最后一个入栈的是this指针。这些堆栈中的参数由被调用的函数在返回清除，即自动清除，使用的指令是 retnX，X表示参数占用的字节数，CPU在ret之后自动弹出X个字节的堆栈空间。函数在编译的时候就必须确定参数个数，并且调用者必须严格的控制参数的生成，不能多或少，否则会编译出错。
##字符串比较
[cpp] view plain copy
1.int __cdecl strcmp (  
2.        const char * src,  
3.        const char * dst  
4.        )  
5.{  
6.        int ret = 0 ;  
7.        while( ! (ret = *(unsigned char *)src - *(unsigned char *)dst) && *dst)  
8.                ++src, ++dst;  
9.  
10.        if ( ret < 0 )  
11.                ret = -1 ;  
12.        else if ( ret > 0 )  
13.                ret = 1 ;  
14.  
15.        return( ret );  
16.}  

##字符串拷贝
[cpp] view plain copy
1.char * __cdecl strcpy(char * dst, const char * src)  
2.{  
3.        char * cp = dst;  
4.        while( *cp++ = *src++ ) ;               /* Copy src over dst */  
5.        return( dst );  
6.}  

##字符串拼接 
[cpp] view plain copy
1.char * __cdecl strcat (  
2.        char * dst,  
3.        const char * src  
4.        )  
5.{  
6.        char * cp = dst;  
7.        while( *cp )  
8.                cp++;                   /* find end of dst */  
9.        while( *cp++ = *src++ ) ;       /* Copy src to end of dst */  
10.        return( dst );                  /* return dst */  
11.}  



##字符串拷贝
[cpp] view plain copy
1.char * __cdecl strncpy (  
2.        char * dest,  
3.        const char * source,  
4.        size_t count  
5.        )  
6.{  
7.        char *start = dest;  
8.  
9.        while (count && (*dest++ = *source++))    /* copy string */  
10.                count--;  
11.  
12.        if (count)                              /* pad out with zeroes */  
13.                while (--count)  
14.                        *dest++ = '\0';  
15.  
16.        return(start);  
17.}  

##查找指定字符串
[cpp] view plain copy
1.char * __cdecl strstr (  
2.        const char * str1,  
3.        const char * str2  
4.        )  
5.{  
6.        char *cp = (char *) str1;  
7.        char *s1, *s2;  
8.        if ( !*str2 )  
9.            return((char *)str1);  
10.        while (*cp)  
11.        {  
12.                s1 = cp;  
13.                s2 = (char *) str2;  
14.  
15.                while ( *s1 && *s2 && !(*s1-*s2) )  
16.                        s1++, s2++;  
17.  
18.                if (!*s2)  
19.                        return(cp);  
20.  
21.                cp++;  
22.        }  
23.        return(NULL);  
24.}  

##查找字符串中最后出现的指定字符
[cpp] view plain copy
1.char * __cdecl strrchr (  
2.        const char * string,  
3.        int ch  
4.        )  
5.{  
6.        char *start = (char *)string;  
7.  
8.        while (*string++)                       /* find end of string */  
9.                ;  
10.                                                /* search towards front */  
11.        while (--string != start && *string != (char)ch)  
12.                ;  
13.        if (*string == (char)ch)                /* char found ? */  
14.                return( (char *)string );  
15.  
16.        return(NULL);  
17.}  

#使用shared_ptr 智能指针包含引用计数，管理方便。

代码如下：
[cpp] view plain copy
1.#include <iostream>  
2.#include <memory>  
3.using namespace std;  
4.  
5.class Point{  
6.public:  
7.    Point() : xval(0),yval(0){};  
8.    Point(int x, int y): xval(x), yval(y){};  
9.    Point(const Point & p)  
10.    {  
11.        if(this == &p)  
12.            return ;  
13.        this->xval = p.x();  
14.        this->yval = p.y();  
15.    }  
16.    int x() const {return xval;};  
17.    int y() const {return yval;};  
18.    Point& setXY(int xv,int yv)   
19.    {   
20.        xval = xv;   
21.        yval = yv;  
22.        return *this;  
23.    };  
24.private:  
25.    int xval, yval;  
26.};  
27.  
28.class Handle{                       //句柄类  
29.public:  
30.    Handle(): up(new Point){};  
31.    Handle(int x,int y): up(new Point(x,y)){};//按创建Point的方式构造handle，handle->UPoint->Point  
32.    Handle(const Point& p): up(new Point(p)){};//创建Point的副本  
33.    Handle(const Handle& h): up(h.up){};//此处复制的是handle，但是底层的point对象并未复制，只是引用计数加1  
34.    //Handle& operator=(const Handle& h)  
35.    //{  
36.    //  up = h.up;  
37.    //  return *this;  
38.    //};  
39.    ~Handle()  
40.    {  
41.    };  
42.    Handle& setXY(int xv,int yv)  
43.    {  
44.        up->setXY(xv,yv);   
45.        return *this;  
46.    };  
47.  
48.    int y() const{return up->y();}  
49.    int x() const{return up->x();}  
50.  
51.    int OutputU()  
52.    {  
53.        return up.use_count();  
54.    }   //输出引用个数  
55.private:  
56.    shared_ptr<Point> up;  
57.};  
58.  
59.int main()  
60.{  
61.    Handle h1(1,2);  
62.    {  
63.        Point p(8,9);  
64.        Handle h2 = h1;        //此处调用的是构造函数Handle(const Handle& h)  
65.        h2.setXY(3,4);                
66.        Handle h3(5,6);          
67.        h1 = h3;  
68.        Handle h4(p);  
69.        Handle h5(h4);  
70.        h4.setXY(7,8);  
71.        cout <<"h1(" << h1.x() <<":"<< h1.y() << "):" << h1.OutputU() <<endl;  
72.        cout <<"h2(" << h2.x() <<":"<< h2.y() << "):" << h2.OutputU() <<endl;  
73.        cout <<"h3(" << h3.x() <<":"<< h3.y() << "):" << h3.OutputU() <<endl;  
74.        cout <<"h4(" << h4.x() <<":"<< h4.y() << "):" << h4.OutputU() <<endl;  
75.        cout <<"h5(" << h5.x() <<":"<< h5.y() << "):" << h5.OutputU() <<endl;  
76.        cout<<p.x()<<" "<<p.y()<<endl;  
77.    }  
78.    cout <<"h1(" << h1.x() <<":"<< h1.y() << "):" << h1.OutputU() <<endl;  
79.    return 0;  
80.}  

运行结果：
h1(5:6):2
h2(3:4):1
h3(5:6):2
h4(7:8):2
h5(7:8):2
8 9
h1(5:6):1


#c++ STL 容器一些底层机制
##1、vector容器
vector的数据安排以及操作方式，与array非常相似。两者的唯一区别在于空间的运用的灵活性。array是静态空间，一旦配置了就不能改变。vector是动态空间，随着元素的加入，它的内部机制会自行扩充空间以容纳新元素。因此，vector的运用对于内存的合理利用与运用的灵活性有很大的帮助，我们再也不必因为害怕空间不足而一开始要求一个大块的array。

vector动态增加大小，并不是在原空间之后持续新空间（因为无法保证原空间之后尚有可供配置的空间），而是以原大小的两倍另外配置一块较大的空间，然后将原内容拷贝过来，然后才开始在原内容之后构造新元素，并释放原空间。因此，对vector的任何操作，一旦引起空间重新配置，指向原vector的所有迭代器就都失效了。

##2、list容器
相对于vector的连续空间，list就显得复杂许多，它的好处是每次插入或删除一个元素，就配置或释放一个元素空间。因此，list对于空间的运用有绝对的精准，一点也不浪费。而且，对于任何位置的元素插入或元素移除，list永远是常数时间。STL中的list是一个双向链表，而且是一个环状双向链表。

##3、deque容器
 deque 是一种双向开口的连续线性空间。所谓双向开口，意思是可以在队尾两端分别做元素的插入和删除操作。deque和vector的最大差异，一在于deque允许于常数时间内对起头端进行元素的插入或移除操作，二在于deque没有所谓容量观念，因为它是动态地以分段连续空间组合而成，随时可以增加一段新的空间并链接在一起。换句话说，像vector那样"因旧空间不足而重新配置一块更大空间，然后复制元素，再释放旧空间"这样的事情在 deque是不会发生的。
deque是由一段一段的定量连续空间构成。一旦有必要在deque的前端或尾端增加新空间，便配置一段定量连续空间，串接在整个deque的头端或尾端。deque的最大任务，便是在这些分段的定量连续空间上，维护其整体连续的假象，并提供随机存取的接口。避开了"重新配置，复制，释放"的轮回，代价则是复杂的迭代器架构。因为有分段连续线性空间，就必须有中央控制，而为了维持整体连续的假象，数据结构的设计及迭代器前进后退等操作都颇为繁琐。
deque采用一块所谓的map作为主控。这里的map是一小块连续空间，其中每个元素都是指针，指向另一段连续线性空间，称为缓冲区。缓冲区才是deque的存储空间主体。SGI STL允许我们指定缓冲区大小，默认值0表示将使用512 bytes缓冲区。

##4、stack
stack 是一种先进后出（First In Last Out , FILO）的数据结构。它只有一个出口，stack 允许新增元素，移除元素，取得最顶端元素。但除了最顶端外，没有任何其它方法可以存取stack的其它元素，stack不允许遍历行为。
以某种容器作为底部结构，将其接口改变，使之符合“先进后出”的特性，形成一个stack，是很容易做到的。deque是双向开口的数据结构，若以deque为底部结构并封闭其头端开口，便轻而易举地形成了一个stack.因此，SGI STL 便以deque作为缺省情况下的stack底部结构，由于stack 系以底部容器完成其所有工作，而具有这种"修改某物接口，形成另一种风貌"之性质者，称为adapter（配接器），因此，STL stack 往往不被归类为container(容器)，而被归类为 container adapter.

##5、 queue
queue是一种先进先出(First In First Out,FIFO) 的数据结构。它有两个出口，queue允许新增元素，移除元素，从最底端加入元素，取得最顶端元素。但除了最底端可以加入，最顶端可以取出外，没有任何其它方法可以存取queue的其它元素。
以某种容器作为底部结构，将其接口改变，使之符合“先进先出”的特性，形成一个queue，是很容易做到的。deque是双向开口的数据结构，若以 deque为底部结构并封闭其底部的出口和前端的入口，便轻而易举地形成了一个queue.因此，SGI STL 便以deque作为缺省情况下的queue底部结构，由于queue 系以底部容器完成其所有工作，而具有这种"修改某物接口，形成另一种风貌"之性质者，称为adapter（配接器），因此，STL queue往往不被归类为container(容器)，而被归类为 container adapter.

##6、heap
heap并不归属于STL容器组件，它是个幕后英雄，扮演priority queue的助手。priority queue允许用户以任何次序将任何元素推入容器中，但取出时一定按从优先权最高的元素开始取。按照元素的排列方式，heap可分为max-heap和min-heap两种，前者每个节点的键值(key)都大于或等于其子节点键值，后者的每个节点键值(key)都小于或等于其子节点键值。因此， max-heap的最大值在根节点，并总是位于底层array或vector的起头处；min-heap的最小值在根节点，亦总是位于底层array或vector起头处。STL 供应的是max-heap，用c++实现。

堆排序c++语言实现
/*堆排序 实现，算法复杂度O(nlgn)*/
/*假设节点i的左右子树都是最大堆，操作使节点i的子树变成最大堆*/
void maxHeap(int A[],int len,int i)
{
    int l = 2 * i + 1;
	int r = 2 * i + 2; 
    int large = i;
    if(l < len)
    {
        if(A[l] > A[i])
        {
            large = l;
        }
    }
    if(r < len)
    {
        if(A[r] > A[large])
        {
            large = r;
        }   
    }
    if(large != i)
    {
        std::swap(A[large],A[i]);
        maxHeap(A,len,large);
    }
}

/*建立大根堆*/
void buildMaxHeap(int A[],int len)
{
    for(int i=len/2-1;i>=0;i--)
        maxHeap(A,len,i);
}


/*堆排序*/
void maxHeapSort(int A[],int len)
{
    buildMaxHeap(A,len);
    printf("建立大跟堆\n");
    for(i=0;i<len;i++)
        printf("%d ",A[i]);
    printf("\n");
    for(i=len-1;i> 0;i--)
    {
        std::swap(A[0],A[i]);
        printf("%d  ",A[i]);
        buildMaxHeap(A,i);
    }
    printf("\n");
}

/*测试堆排序*/
int main()
{
    int i;
    int A[11]={4,1,3,2,16,9,10,14,8,7,6};
    maxHeapSort(A,11);
    for(i=0;i<11;i++)
    {
        printf("%d  ",A[i]);
    }
    printf("\n");
}


##7、priority_queue
priority_queue是一个拥有权值观念的queue,它允许加入新元素，移除旧元素，审视元素值等功能。由于这是一个queue，所以只允许在底端加入元素，并从顶端取出元素，除此之外别无其它存取元素的途径。priority_queue带有权值观念，其内的元素并非依照被推入的次序排列，而是自动依照元素的权值排列（通常权值以实值表示）。权值最高者，排在最前面。缺省情况下priority_queue系利用一个max-heap完成，后者是一个以vector表现的 complete binary tree.max-heap可以满足priority_queue所需要的"依权值高低自动递减排序"的特性。
priority_queue完全以底部容器作为根据，再加上heap处理规则，所以其实现非常简单。缺省情况下是以vector为底部容器。queue以底部容器完成其所有工作。具有这种"修改某物接口，形成另一种风貌"之性质者，称为adapter(配接器)，因此，STL priority_queue往往不被归类为container(容器)，而被归类为container adapter.

##8、set multiset
set的特性是，所有元素都会根据元素的键值自动被排序。set的元素不像map那样可以同时拥有实值(value)和键值(key), set 元素的键值就是实值，实值就是键值，set不允许两个元素有相同的值。set是通过红黑树来实现的，由于红黑树（RB-tree）是一种平衡二叉搜索树，自动排序的效果很不错，所以标准的STL的set即以RB-Tree为底层机制。又由于set所开放的各种操作接口，RB-tree也都提供了，所以几乎所有的set操作行为，都只有转调用RB-tree的操作行为而已。

multiset的特性以及用法和set完全相同，唯一的差别在于它允许键值重复，因此它的插入操作采用的是底层机制RB-tree的insert_equal()而非insert_unique().

##9、map multimap
map的特性是，所有元素都会根据元素的键值自动被排序。map的所有元素都是pair,同时拥有实值（value）和键值（key）.  pair的第一元素被视为键值，第二元素被视为实值。map不允许两个元素拥有相同的键值.由于RB-tree是一种平衡二叉搜索树，自动排序的效果很不错，所以标准的STL map即以RB-tree为底层机制。又由于map所开放的各种操作接口，RB-tree也都提供了，所以几乎所有的map操作行为，都只是转调RB-tree的操作行为。
multimap的特性以及用法与map完全相同，唯一的差别在于它允许键值重复，因此它的插入操作采用的是底层机制RB-tree的insert_equal()而非insert_unique。 



#现代 C++ 教程：高速上手 C++ 11/14/17/20
https://changkun.de/modern-cpp/zh-cn/00-preface/


#Effective STL 50条有效使用STL的经验

##容器

条款01：慎重选择容器类型
容器分类。1）序列容器（vector、string、deque和list）和关联容器（set、map、multiset、multimap）。2）或连续内存容器和基于节点的容器。
连续内存容器：vector、string、deque和非标准的rope。
基于节点的容器：list和forward_list(c++11)、所有标准的关联容器（set、map、multiset、multimap实现方式是平衡树），非标准的哈希容器（unorder_set、unorder_map c++11）。
基于节点的容器（除deque）的插入和删除操作从来不会使迭代器、指针和引用变为无效（除非它们指向一个你正在删除的元素）。而针对连续内存容器的插入和删除操作一般会使指向该容器的迭代器、指针和引用变为无效（删除点及之后的所有迭代器失效，添加可能导致重新分配内存，导致所有迭代器失效）。
当插入操作仅在容器末尾发生时，deque的迭代器可能会变为无效，但指针和引用有效。Deque是唯一的、迭代器可能会变为无效而指针和引用不会变成无效的STL标准容器。

条款02：不要试图编写独立于容器类型的代码
只有序列容器才支持push_back或push_front。
当你向序列容器中插入对象时，对象位于被插入的位置处；当你向关联容器中插入对象时，容器会按照其排序规则，将该对象移动到适当的位置上。
要求随机访问迭代器的操作：sort、stable_sort、partial_sort和nth_element。
Vector<bool>并不总是表现的像一个vector，它实际上并没有存储bool类型的对象。
在某些情况下，你意思到自己选择的容器类型不是最佳，你想使用令一种容器类型。在写容器代码之前，你需要考虑使用封装技术。
最简单的方式是通过对容器类型和其迭代器类型使用类型定义（typedef），词法层面。要想减少在替换容器类型时所需要修改的代码，你可以把容器隐藏到类中。

条款03：确保容器中的对象拷贝正确而高效
当（通过如insert或push_back之类的操作）向容器中加入对象时，存入容器的是你所指定的对象的拷贝。
Widget w[n];创建了n个widget的数组，每个数组都使用默认构造函数来创建。

条款04：调用empty而不是检查size是否为0
你应该使用empty形式，理由：empty对所有的标准容器都是O(1),而对一些list实现，size为o(n)。
List的 size（）和splice（）：有一个为O（n）和另一个为O(1)。

条款05：区间成员函数优先于与之对应的单元素成员函数
容器赋值成员函数：assign。对于所有的标准序列容器（vector、string、deque和list），它都存在。
通过利用插入迭代器的方式来限定目标区间的copy调用，几乎都应该被替换为对区间成员函数的调用。
对于标准序列容器，使用区间成员函数有更高的效率（函数调用、元素移动、内存分配、多次赋值等方面）；对于关联容器，使用区间成员函数效率不会更低。但是，区间成员函数，代码更加易懂，可维护性强。
区间成员函数：
区间创建：container::container(InputIterator begin,InputIterator end)
区间插入：
Void container::insert(iterator position, InputIterator begin,InputIterator end) 标准序列容器需要提供插入位置
Void container::insert(InputIterator begin,InputIterator end)关联容器利用比较函数来决定元素该插入何处
区间删除：
iterator container::erase(iterator begin, iterator end)
void container::erase(iterator begin, iterator end)
区间赋值：Void container::assign(InputIterator begin,InputIterator end)
选择区间成员函数理由：1）写起来容易；2）更能清晰地表达你的意图；3）它们表现出了更高的效率。

条款06：当心C++编译器最烦人的分析机制
在函数形参中，参数int（d）两边的括号是多余的，参数int d()后边的括号使d理解为（隐式）函数指针，与int (*d)()等价。
读取文件：
Ifstream datafile(“ints.dat”);
Istream_iterator<int> dataBegin(datafile);
Istream_iterator<int> dataEnd;
List<int> data(dataBegin,dataEnd);

条款07：如果容器中包含了通过new操作创建的指针，切记在容器对象析构前将指针delete
指针容器在自己被析构时会析构所含的每个元素，但指针的“析构函数”不做任何事情，当然不会调用delete，直接导致资源泄露。
为了防止资源泄露，可以把使用智能指针（std::shared_ptr）或手工删除（也可以借助for_each和DeleteObject（）函数对象）

条款08：切勿创建包含auto_ptr的容器对象
auto_ptr意味着所有权的转移。当你复制一个auto_ptr时，它所指向的对象的所有权被移交到复制的auto_ptr上，而它自身被置为NULL。
包含auto_ptr的容器对象应该被禁止。例如当对该对象sort时，就会发生意想不到的情况。

条款09：慎重选择删除元素的方法
序列容器vector、string、deque和list只提供position的erase函数；关联容器不仅提供position的erase还提供value的erase函数。
要删除容器中有特定值的所有对象：1）vector、string、deque使用erase-remove；2）list使用remove；3）关联容器使用erase。
要删除容器中满足特定判别式（条件）的所有对象：1）vector、string、deque使用erase-remove_if；2）list使用remove_if；3）关联容器使用remove_copy_if-swap或写一个循环来遍历容器中的元素，记住当把迭代器传给erase时，要对它进行后缀递增。
要在循环内部做些（除删除对象之外的）操作：1）序列容器，记住每次调用erase时，要用它的返回值更新迭代器；2）关联容器，记住当把迭代器传给erase时，要对它进行后缀递增。

条款10：了解分配子（allocator）的约定和限制
STL内存分配子（allocator）负责分配（和释放）原始内存。
STL实现可以假设同一类型的allocator是等价的。Allocator是对象，这意味着可移植的allocator不可以有任何非静态的数据成员。
Operator new和allocator<T>::allocator的区别：参数类型不同（new 字节数，allocator类型数）返回类型不同。Void* operator new(size_t bytes); pointer allocator<T>::allocator(size_type numObjects)，（尽管是pointer(T*) 但未构造）。
大多数标准容器从来没有单独调用对应的分配子（allocator<T>）。这是因为T并不一定是插入一个元素T需要分配的内存。例如list<T>在插入一个新的节点时，需要的内存为T+2*(T*)。所以，需要根据allocator<T>找到allocator<U>。
所以在allocator<T>需提供rebind模板，rebind模板内定义typedef allocator<U> other。通过Allocator<T>::rebind<ListNode>::other获得真正使用的分配子。

条款11：理解自定义分配子的合理用法
1. 在自定义allocator时，需要遵守同一类型的allocator必须是等价的。

条款12：切勿对STL容器的线程安全性有不切实际的依赖
当涉及STL容器和线程安全时，你可以指望一个STL库允许多个线程同时读一个容器，以及多个线程对不同的容器做写操作。你不能指望STL库会把你从同步控制中解脱出来，而且你不能依赖任何线程支持。
为了防止资源泄露（多线程的互斥体），使用类管理资源，构造函数中获得资源，析构函数中归还资源。可以通过创建新的代码块{}，合理选择析构资源的时机。

Vector和string

条款13：vector和string优先于动态分配的数组
如果你正在动态地分配数组，那么你可能要做更多的工作。为了减轻自己的负担，请使用vector或string。
如果你使用的string以引用计数来实现的，而你又运行在多线程环境中，并认为string的引用计数实现会影响效率。你可以：1）看看能够禁止引用计数。2）寻找或开发另一个不使用引用计数的string实现。3）使用vector<char>替代string，vector实现不允许使用引用计数。

条款14：使用reserve来避免不必要的重新分配
vector和string增长过程：
分配一块大小为当前容量的某个倍数的新内存。在大多数实现中，vector和string的容量每次以2的倍数增长，即每当容器需要扩张时，它们的容量即加倍。
把容器的所有元素从旧的内存复制到新的内存中。
析构掉旧内存中的对象。
释放旧内存。
v.size():容器v中元素的数量；v.capacity():容器v在不重新分配内存的最大容量；v.resize(container::size_type n):强制容器v包含前n个元素（不够就默认构造，多的析构）；v.reserve(container::size_type n):强迫容器的容量变为至少是n。
使用reserve可以避免不必要的重新分配。1）若确切或大体知道容器元素数量，直接reserve；2）也可以根据需要，先预留足够大的空间，再去除多余的容器。Reserve只预留内存，不执行构造。

条款15：注意string实现的多样性
string有不同的实现版本。有可能使用引用计数也可能不使用，string对象的大小可以是一个char*指针的1倍到7倍。
创建一个新的字符串可能需要0、1、2次动态分配内存。

条款16：了解如何把vector和string数据传给旧的API
有vector v，而需要得到一个指向v中数据的指针（数组），只需&v[0]，但是需要小心v为空的情况；对于string s，对应的形式是string的成员函数s.c_str()。
来自C API的元素可以直接初始化vector，因为vector和数组的内存布局兼容性。
对于string、list、deque、set等，可以通过vector的中间作用，实现旧API和STL容器间的转换。

条款17：使用“swap技巧”除去多余的容量
借助临时容器（使用现有容器初始化）和现有容器的交换：vector<type>(vec).swap(vec)。表达式Vector<type>(vec)创建一个临时的容器，它是vec的拷贝：这是由vector的拷贝构造函数来完成的。然而，vector的拷贝构造函数只为所拷贝的元素分配所需要的内存，所以这个临时向量没有多余的容量。这就是shrink to fit。C++11提供shrink_to_fit()函数。
也可以使用它清空容器：vector<type>().swap(vec) size=capacity=0。而使用clear()函数，size=0，capacity不一定为0。
在做swap的时候，不仅两个容器的内容被交换，同时他们的迭代器、指针和引用也将被交换（string除外）。在swap发生后，原先指向某容器中元素的迭代器、指针和引用依然有效，并指向同样的元素—但是这些元素已经在另一个容器了。其实，只改变了vector的数据成员。

条款18：避免使用vector<bool>
vector<bool>：1）它不是STL容器。2）它并不存储bool。
vector<bool>是一个假容器，它并不真的存储bool，相反，为了节省空间，它存储的是bool的紧凑表示。
vector<bool>是失败产物，你可以使用deque<bool>或bitset来替代它。

##关联容器

条款19：理解相等（equality）和等价（equivalence）的区别
相等是以operator==为基础的；等价是以operator<为基础的，“在已排序的区间中对象值的相对顺序。
应优先使用成员函数（像std::find）而不是与之对应的非成员函数（像find）。
标准关联容器总是保持排列顺序的，所以每个容器必须有一个比较函数（默认为less）来决定保持怎样的顺序。等价的定义正是通过该比较函数而确定。

条款20：为包含指针的关联容器指定比较类型
每当你要创建包含指针的关联容器时，一定要记住，容器将会按照指针的值进行排序。绝大多数情况下，这不会是你所希望的，所以你几乎肯定要创建自己的函数子类作为该容器的比较类型（比较函数的类型）。
Set模板的三个参数每个都是类型。Set不需要一个比较函数，它需要的是一个类型，并在内部使用它创建一个函数。

条款21：总是让比较函数在等值情况下返回false
除非你的比较函数对相等的值总是返回false，否则你会破坏所有的标准关联容器，不管它们是否存储重复的值。

条款22：切勿直接修改set或multiset中的键
对于set<T>或multiset<T>类型的对象，容器中的类型是T，而不是const T，如果你愿意，随时可以修改set或multiset<T>中的元素，只要不修改键部分（用于自动排序的部分）。
直接修改set或multiset<T>中的元素，可能会出现问题。不同STL的实现中，set<T>::iterator 的operator*可能返回一个const T&。故存在移植性问题。
可移植的做法：
1）保持一份想要修改元素的非const副本，并在该副本上修改。
2）记录元素的set（或map等关联容器）中的位置，并删除它。
3）根据位置，把副本插入到set（或map等关联容器）中。
对于map或multimap<K,V>类型的对象，键的类型是const K。

条款23：考虑用排序的vector替代关联容器
在排序的vector中存储数据可能比在标准关联容器中存储同样的数据要消耗更少的内存，而考虑到页面错误的原因，（此外还有，节点可能会散布在全部的地址空间中），通过二分搜索法来查找一个排序的vector可能比查找一个标准关联容器更快一些。
当使用vector来模仿map<K,V>，存储在vector中数据必须是pair<K,V>而不是pair<const K,V>，这是因为，vector进行排序时，它的元素的值将通过赋值操作被移动，这意味着pair的两部分都必须是可以赋值的。
当用vector替代map时，你还需要定义比较函数（函数对象）。

条款24：当效率至关重要时，请在map::operator[]与map::insert之间谨慎做出选择
map::operator []的设计目的是为了提供“添加和更新”的功能。
效率的角度：如果要更新一个已有的map，则优先选择operator[]；但如果要添加一个新的元素，那么最好还是选择insert。
Operator[]返回一个引用(m[k]=v),它指向与k相关联的值对象。然后v被赋给该引用所指的对象。

条款25：熟悉非标准的散列容器
在c++11中有了散列容器unorder_set、unorder_multiset、unorder_map 、unorder_multimap等。
元素不是以排序方式存放。

##迭代器
条款26：iterator优先于const_iterator、reverse_iterator及const_reverse_iterator
有些版本的insert和erase函数要求使用iterator。Const和revers型的迭代器不能满足这些函数的要求。
Iterator可以隐式转换为const_iterator和reverse_iterator。反之，不可能。
从reverse_iterator转换而来的iterator在使用之前可能需要相应的调整。通过base函数和一个偏移量调整。
Const_iterator的operator==（不仅仅==）作为一个成员函数而不是一个非成员函数。所以可能出现问题。一般应该为非成员函数，这样两个参数都可以隐式转换。
尽量使用iterator。

条款27：使用distance和advance将容器的const_iterator转换成iterator
使用强制类型转换（比如const_cast等）并不能将const_iterator转换成iterator。这是因为iterator和const_iterator是完全不同的类。
可以通过advance(i，distance<ConstIter>(i,ci))实现const_iterator到iterator的转换。Distance函数需要同类型的迭代器。
转换代价：随机访问的迭代器为O(1),对于双向迭代器或散列容器的迭代器为O(N)，并且需要访问const_iterator所属的容器。

条款28：正确理解由reverse_iterator和base()成员函数所产生的iterator的用法
reverse_iterator的base（）返回一个iterator，但与reverse_iterator所指有偏移量。可以通过（++ri）.base()获得相同的所指物。
reverse_iterator的递增的效果是由容器的尾部反向遍历到容器头部。

条款29：对于逐个字符的输入请考虑使用istreambuf_iterator
istream_iterator使用operator>>函数来完成实际的读操作，而默认情况下operator>>函数会跳过空白字符。改变默认情况，只需清除输入流的skipws标志即可。
istream_iterator内部使用的operator实际上会执行格式化输入。
对于非格式化的逐个字符输入过程，你总是应该考虑使用istreambuf_iterator。

##算法
条款30：确保目标区间足够大
back_inserter(container)，back_inserter返回的迭代器将使得push_back()被调用，所以back_inserter可适用于所有提供了push_back方法的容器。
reserve（n）只是增加了容器的容量，而容器的大小并未改变（也就是不会执行构造函数）；resize(n)改变了容器的大小，也可能改变容器的容量（n>当前容量，构造，容量变大；n<当前容量，析构，但容量不变）。
无论何时，如果所使用的算法需要指定一个目标区间，那么必须确保目标区间足够大，或者确保它会随着算法的运行而增大。要在算法执行过程中增大目标空间，请使用插入迭代器，比如ostream_iterator，或者由back_insert\front_inserter和inserter返回的迭代器。

条款31：了解各种与排序有关的选择
随机迭代器的排序算法：sort、stable_sort、partial_sort和nth_element(n代表的下标，实际需要-1)；双向迭代器的分类算法：partition和stable_partition。
如果你真的需要将一个区间进行排序，那么sort、stable_sort、partial_sort是非常有用的；如果你需要找到前n个元素，或者找到某个位置上的元素，那么nth_element（vec.begin(),vec.begin+n-1,vec.end()）可以满足你的需要。
Partition算法可以把满足某个特定条件的元素放在区间前部。并且提供stable_partition版本，需要双向迭代器，所以也可以用于关联容器。
List提供了sort成员函数且是稳定的。
如果想要list实现partial_sort或partition，可以通过间接途径：1）将list元素复制到vector等随机访问迭代器的容器。2）将list的迭代器复制到vector等随机访问容器。3）利用一个包含迭代器的有序容器中的信息，通过反复地调用splice成员函数，将list中的元素调整到期望的目标位置。
对关联容器中的元素进行排序并没有实际意义，因为这样的容器总是使用比较函数来维护内部元素的有序性。

条款32：如果确实需要删除元素，则需要在remove这一类算法之后调用erase
因为从容器中删除元素的唯一方法是调用该容器的成员函数，而remove并不知道它操作的元素所在的容器，所以remove不可能从容器中删除元素。
Remove移动了区间中的元素，其结果，“不用被删除”的元素移动到区间的前部（保持原来的相对顺序）。它返回的一个迭代器指向最后一个“不用被删除”的元素之后的元素。这个返回值相当于该区间“新的逻辑结尾”。
可以把remove想象成一个压缩过程，需要被删除的元素就好像是被压缩过程中需要被填充的洞。被删除的元素不一定位于容器末尾，这只是算法操作的附带结果。若元素是指针，可能存在问题。
如果你想要真正删除元素，那就必须在remove之后使用erase。STL唯一一个名为remove并且确实删除了容器中元素的函数：list的成员函数remove。如list.remove，list.unique也会真正删除元素。
除remove和erase联合使用外，还有两个属于”remove类“的算法：remove_if和unique。

条款33：对包含指针的容器使用remove这一类算法时要特别小心
当容器中存放的是指向动态分配的对象的指针的时候，你应该避免使用remove和类似的算法(remove_if和unique)。很多情况下，你会发现partition算法是个不错的选择。
如果无法避免对这种容器使用remove，那么在进行erase-remove之前，先把那些不符合特定要求的指针删除并置空，然后清除该容器中所有空指针。
最好的做好：不在容器中存放指针，而是存放引用计数的智能指针类型。

条款34：了解哪些算法要求使用排序的区间作为参数
如果你为一个算法提供了一个排序区间，而这个算法也带一个比较函数作为参数，那么你一样要保证你传递的比较函数与这个排序区间所用的比较函数有一致的行为。
用于查找的算法binary_search、lower_bound、upper_bound和equal_range要求排序的区间，因为它们用二分法查找数据并且只有接受随机访问迭代器的时候，才能保证O(n)。内部实现借助了advance（）函数。
Set_union、set_intersection、set_difference和set_symmetric_difference在排序区间提供O(n)的集合操作。
Merge和inplace_merge实际上实现了合并和排序的联合操作：它们读入两个排序的区间，然后合并成一个新的排序区间，其中包含原来两个区间中所有的元素。只有两个区间有序才能O（n）.
Unique和unique_copy只有在排序区间上，才能获取我们想要实现的功能（全区间只有一份数据）。Unique的算法行为：删除每一组连续相等的元素，仅保留其中的第一个。Unique和remove类型，也不是实际意义上的删除。
Includes可用来判断一个区间中的所有对象是否都在另一个区间中。它在区间是顺序的情况下，保证O(n)。

条款35：通过mismatch或lexicographical_compare实现简单的忽略大小写的字符串比较
mismatch将标记出两个区间中第一个对应位置不同的位置。如果两个字符串长度不同，那么我们必须把短的字符串作为第一个区间传入。
lexicographical_compare是一个泛化版本，可以比较任何类型的值的区间。lexicographical_compare可以接受一个判别式，由判别式来决定两个值是否满足一个用户自定义的准则。默认是“<”，即str1 < str2?true:false。
如果不考虑移植性，并且字符串中不会包含内嵌的空字符，你可能只需要使用strcmp、stricmp等，他们通常被优化过，比mismatch和lexicographical_compare快得多。

条款36：理解copy_if算法的正确实现
C++11中重新添加了该函数。实现：逐个复制。

条款37：使用accumulate或for_each进行区间统计
count告诉你一个区间中有多少个元素，而count_if则统计出满足某个判别式的元素个数，区间中的最小值和最大值可以通过min_element、max_element来获得。
传给accumulate的函数对象（函数）可以提供两个参数；传给for_each的函数对象（函数）只接收一个实参（即当前的区间元素）。
Accumulate直接返回我们想要的统计结果，而for_each却返回一个函数对象，我们必须从这个函数对象中提取出我们所要的统计信息。
函数子、函数子类、函数及其他

条款38：遵循按值传递的原则来设计函数子类
假设函数对象总是按值方式传递的。那么，函数对象要尽可能小，同时，函数对象必须是单态的（不是多态的），也就是说，它们不能使用虚函数。这是因为，函数对象按值传递，若实参派生类对象传递给形参基类对象，会出现剥离问题。但是，有时候我们需要函数对象附带附加信息，和使用多态。
解决办法：将所需的数据和虚函数从函数子类中分离出来，放到一个新的类中，然后在函数子类中包含一个指针（考虑使用智能指针），指向这个新类的对象（把原对象设为friend类。

条款39：确保判别式是“纯函数”
判别式：是一个返回bool类型（或者可以隐式转换为bool类型）的函数。纯函数：返回值仅仅依赖于其参数和常量的函数。判别式类：是一个函数子类，它的operator()函数是一个判别式。
一个行为正常的判别式的operator()肯定是const的，但是它还有更严格的要求。它还应是个“纯函数”。这项限制也同样使用于判别式函数。

条款40：若一个类是函数子，则应使它可配接
ptr_fun只不过完成了一些类型定义的工作，这些类型定义却是not1所需要的。
提供了这些必要的类型定义的函数对象称为可配接的函数对象，反之，则不是。可配接的函数对象能够与其他STL组件更为默契的协同工作。
这些类型定义是：argument_type、first_argument_type、second_argument_type以及result_type。
提供这些类型定义最简单的办法是让函数子类从特定的基类继承。
如果函数子类的operator()只有一个实参，那么它应该从std::unary_function继承：std::unary_function<argument_type,bool>
如果函数子类的operator()有两个实参，那么它应该从std::binary_function继承：std::binary_function<first_argument_type,second_argument_type,bool>
一般情况下，传递给unary_function、binary_function的非指针类型需要去掉const和引用（&）部分。若是，指针类型，不可去掉const和引用（&）。
对于一个函数子类定义多个operator()（重载），只有一个是可配接的。尽量避免这种情况。

条款41：理解ptr_fun、mem_fun、mem_fun_ref的来由
ptr_fun、mem_fun、mem_fun_ref主要是解决c++语言的函数调用的语法不一致问题。他们还提供一些重要的类型定义。
如果有一个函数f和一个对象x，希望在x上调用f，C++提供三种不同的语法：
1）f(x) //语法#1 f为一个非成员函数
2）x.f() //语法#2 f是成员函数，并且x是一个对象或对象的引用(mem_fun_ref)
3）p->f() //语法#3 f是成员函数，并且p是一个指向对象x的指针(mem_fun)
SGI STL默认是使用语法#1，其他形式不能通过编译。
Mem_fun函数将语法#3调整为语法#1。Mem_fun带一个指向某个成员函数的指针参数，并且返回一个mem_fun_t类型的函数对象。
Mem_fun_ref函数将语法#2调整为语法#1，并产生一个类型为mem_fun_ref_t的函数对象（函数对象配接器）。

条款42：确保less<T>与operator<具有相同的语义
less<T>默认情况下会调用operator<来完成它的工作。很多假设，使用less总是等价于使用operator<。Operator<不仅仅是less的默认实现方式，它也是程序员期望less所做的事情。
如果你希望以一种特殊的方式来排列对象（关联容器中的对象），那么最好创建一个特殊的函数子类，它的名字不能是less。

在程序中使用STL

条款43：算法调用优先于手写的循环
算法调用优于手写的循环，理由：1）效率（容器的底层实现细节等），2）正确性，3）可维护性。
Transform函数：某个函数将被应用到区间中的每一个对象上，而这些调用的结果被写到某一个地方。
Replace_if函数：区间中所有满足某个判别式条件的对象都将被修改。
Partition函数：一个区间中的对象将会被移动，所有满足某个判别式条件的对象会被组织到一起。

条款44：容器的成员函数优于同名的算法
关联容器的成员函数：count、find、lower_bound、upper_bound、equal_range，而list的成员函数remove、remove_if、unique_sort、merge和reverse。
大多数情况下，你应该使用这些成员函数，而不是相应的算法。理由：1）成员函数往往速度快。2）成员函数与容器（特别是关联容器）结合得更加紧密。原因在于，算法和成员函数虽然有相同的名称，但是他们所做的事情往往不完全相同。
STL算法以相等性来判断两个对象是否具有相同的值，而关联容器则使用等价性来进行他们的“相同性”测试。

条款45：正确区分count、find、binary_search、lower_bound、upper_bound和equal_range
binary_search、lower_bound、upper_bound和equal_range应用于排序的区间，count、find用于非排序的区间，O(N)。
从未排序的区间到排序的区间的转变也带来了另一种变化：前者利用相等性来决定两个值是否相同，而后者使用等价性作为判断依据。
Binary_search返回一个bool类型，只回答“是否存在”的问题。
Lower_bound会返回一个迭代器，该迭代器要么指向该值的第一份拷贝（如果找到），要么指向一个适合插入该值的位置（如果没有找到）。Lower_bound和upper_bound太容易退到相等性测试。
Equal_range返回一对迭代器（pair），第一个迭代器等于lower_bound返回的迭代器，第二个迭代器等于upper_bound返回的迭代器（即指向该区间中与查找的值等价的最后一个元素的下一个位置）。
以上是关于给定区间的讨论，若给的容器是list或关联容器，尽量它们的成员函数。

条款46：考虑使用函数对象而不是函数作为STL算法的参数
将函数对象传递给STL算法往往比传递实际的函数更加高效。函数对象的operator()函数可以inline，而而函数只能以函数指针的形式传递给STL算法，函数指针参数抑制了内联机制。
可能由于编译器的缺陷或STL库的原因，使得使用函数会出现问题，而函数对象可以。
函数对象优于函数的第三个理由是，这样做有助于避免一些微妙、语言本身的缺陷。

条款47：避免产生“直写型”（write-only）的代码
不要太多的嵌套。

条款48：总是包含(#inlude)正确的头文件
C++标准与C的标准不同，它没有规定标准库中的头文件之间的相互包含关系。
几乎所有的标准STL容器都被声明在与之同名的头文件中，比如vector被声明在<vector>中，等等。
除了4个STL算法以外，其他所有的算法都被声明在<algorithm>中，这是个算法accumulate、inner_product、adjacent_difference和partial_sum，被声明在头文件<numeric>中。
特殊类型的迭代器，包括istream_iterator和istreambuf_iterator，被声明在<iterator>中。
标准的函数子（如less<T>）和函数子配接器（比如not1、bind2nd）被声明在头文件<functional>中。

条款49：学会分析与STL相关的编译器诊断信息
C++中所有与string类似的类型实际上都是basic_string模板的实例。
Vector和string的迭代器通常是指针，所以当错误地使用iterator时，编译器的诊断可能会引用到指针类型。
如果你正在使用一个很常见的STL组件，比如vector、string或者for_each，但是从错误信息来看，编译器好像对此一无所知，那么可能你没有包含相应的头文件。

条款50：熟悉与STL相关的web站点
1. SGI STL站点、STLport站点、Boost站点

#runoob
https://www.runoob.com/cplusplus/cpp-tutorial.html

#cppreference
http://www.cplusplus.com/reference/atomic/atomic/exchange/
