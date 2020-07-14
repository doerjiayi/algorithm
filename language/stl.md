
##STL容器内存浅析(1)-vector内存分配策略
https://blog.csdn.net/qiuguolu1108/article/details/107146184/

##STL源码剖析-vector 
https://www.cnblogs.com/yocichen/p/10574819.html

##STL容器 迭代器失效总结
https://blog.csdn.net/qq_22238021/article/details/79591526

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