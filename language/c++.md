#c++
c++编程思想

##c++反射
https://www.cnblogs.com/lizhanwu/p/4428990.html

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

定义一个通用模板
模板特化和偏特化
模板实例化与匹配
可变参数模板
　　泛型编程是指独立与任何类型的方式编写代码。泛型编程和面向对象编程，都依赖与某种形式的多态。面向对象编程的多态性在运行时应用于存在继承关系的类，一段代码可以可以忽略基类和派生类之间的差异。在泛型编程中，编写的代码可以用作多种类型的对象。面向对象编程所依赖的多态性称为运行时多态性，泛型编程所依赖的多态性称为编译时多态性或参数式多态性。 

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


#参考
##runoob
https://www.runoob.com/cplusplus/cpp-tutorial.html

##cppreference
http://www.cplusplus.com/reference/atomic/atomic/exchange/

##现代 C++ 教程：高速上手 C++ 11/14/17/20
https://changkun.de/modern-cpp/zh-cn/00-preface/
