---
layout:     post
title:      "C++"
subtitle:   "summary"
author:     "Alan Nam"
header-img: "img/in-post/post-bg-hello.jpg"
tags:
    - c++
 


---

## Summary

#### const

**const_iterator != const iterator**

- iterator指向非const对象
- const_iterator指向const对象
- const_iterator对象本身不是const
- 可以在const_iterator对象上执行++。
- 不能写入const_iterator对象(*iter = 3)

```c++
std::vector<int> non_const_vec{1, 2, 3};
const std::vector<int> const_vec{1, 2, 3};

auto iter = non_const_vec.begin(); // non-const iter
const auto iter2 = non_const_vec.begin(); // const iter
auto iter3 = const_vec.begin(); // const_iterator

```



#### const&static

- 静态常量数据成员可以在类内初始化(即类内声明的同时初始化)，也可以在类外，即类的实现文件中初始化，不能在构造函数中初始化，也不能在构造函数的初始化列表中初始化；
- 静态非常量数据成员只能在类外，即类的实现文件中初始化，也不能在构造函数中初始化，不能在构造函数的初始化列表中初始化；
- 非静态的常量数据成员不能在类内初始化，也不能在构造函数中初始化，而只能且必须在构造函数的初始化列表中初始化；

![image-20230403163100729](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230403163100729.png)

#### reference &

```c++
//You can return references as well!
// Note that the parameter must be a non-const reference to return
// a non-const reference to one of its elements!
int& front(std::vector<int>& vec) {
 // assuming vec.size() > 0
 return vec[0];
}
int main() {
 std::vector<int> numbers{1, 2, 3};
 front(numbers) = 4; // vec = {4, 2, 3}
 return 0;
}

//Can also return const references
const int& front(std::vector<int>& vec) {
 // assuming vec.size() > 0
 return vec[0];
}

//Dangling references: references to out-of-scope vars
//Never return a reference to a local variable! They’ll go out of scope.
int& front(const std::string& file) {
 std::vector<int> vec = readFile(file);
 return vec[0];
}
int main() {
 front("text.txt") = 4; // undefined behavior
 return 0;
}
```



#### lambda

Lambda函数允许我们在运行时创建函数

##### Lambda syntax:

```c++
auto is_less_than = [limit](auto val) {
return (val < limit);
}
```

**Capture values:**

```c++
auto lambda = [capture-values](arguments) {
return expression;
}
[x](arguments) // captures x from surrounding scope by value
[x&](arguments) // captures x from surrounding scope by reference
[x, y](arguments) // captures x, y by value
[&](arguments) // captures everything by reference
[&, x](arguments) // captures everything except x by reference
[=](arguments) // captures everything by copy
```



#### lvalues & rvalues

l值和r值概括了“暂时性”的概念。A l-value can appear left or right of =.

r值是“临时的”，而l值不是。An r-value can only appear right of =.

A l-value’s lifetime is until end of scope... 

A r-value’s lifetime is until end of line, unless you artificially extend it

**官方定义:**l值有一个地址(可以做&)，而r值没有。

r-value 引用：

```c++
int main() {
change(7);
}
void change(int&& a) {...}
```

#### move

```c++
template <typename T>
void vector<T>::push_back(const T& element) {
elems[_size++] = element; // equals → copy
}
template <typename T>
void vector<T>::push_back(T&& element) {
elems[_size++] = std::move(element); // move!
}
```

**std::swap**

```c++
template <typename T>
void swap(T& a, T& b) noexcept {
T c(std::move(a)); // move constructor
a = std::move(b); // move assignment
b = std::move(c); // move assignment
} 
```



#### RAII (Resource Acquisition is  Initialization)

- 在类的构造函数中获取资源，在析构函数中释放。
- RAII类的客户端不必担心资源管理不当

```c++
void printFile () {
 ifstream input(“hamlet.txt”);
 string line;
 while (getline(input, line)) { // might throw exception
 cout << line << endl;
 }
 // no close call needed!
} // stream destructor, releases access to file



void cleanDatabase (mutex& databaseLock,
 map<int, int>& database) {
 lock_guard<mutex> lock(databaseLock);
 // other threads will not modify database
 // modify the database
 // if exception thrown, that’s fine!
 // no release call needed
} // lock always unlocked when function exits.
```

##### **Smart Pointers** (RAII for memory)

**建议使用make_unique**

```c++
//std::unique_ptr<T> up{new T};
std::unique_ptr<T> up = std::make_unique<T>();
//std::shared_ptr<T> sp{new T};
std::shared_ptr<T> sp = std::make_shared<T>();

std::weak_ptr<T> wp = sp;
//can only be copy/move constructed (or empty)!
```

**std::unique_ptr**:

- 唯一拥有它的资源，并在对象被销毁时删除它
- 不允许复制

```c++
Before 
void rawPtrFn () {
 Node* n = new Node;
 // do stuff with n…
 delete n;
}
After
void rawPtrFn () {
 std::unique_ptr<Node> n(new Node);
 // do some stuff with n
} // Freed!
       

```

**std::shared_ptr**:

- 资源可以由任意数量的std::shared_ptr存储
- 当没有人指向该资源时，该资源将被删除

```c++
{
 std::shared_ptr<int> p1(new int);
 // Use p1
 {
 std::shared_ptr<int> p2 = p1;
 // Use p1 and p2
 }
 // Use p1
}
// The integer is deallocated!
```

**std::weak_ptr**:

借助 weak_ptr 类型指针， 我们可以获取 shared_ptr 指针的一些状态信息，比如有多少指向相同的 shared_ptr 指针、shared_ptr 指针指向的堆内存是否已经被释放等等。

```c++
eg1:
std::shared_ptr<int> sp1(new int(10));
std::shared_ptr<int> sp2(sp1);
std::weak_ptr<int> wp(sp2);
//输出和 wp 同指向的 shared_ptr 类型指针的数量
cout << wp.use_count() << endl;//2
//释放 sp2
sp2.reset();
cout << wp.use_count() << endl;//1
//借助 lock() 函数，返回一个和 wp 同指向的 shared_ptr 类型指针，获取其存储的数据
cout << *(wp.lock()) << endl;//10

eg2:
std::shared_ptr<int> sp(new int(1));
std::cout << sp.use_count() << std::endl;//1
 
std::weak_ptr<int> wp(sp);
std::cout << sp.use_count() << std::endl;//1
 
std::shared_ptr<int> sp1 = wp.lock();
std::cout << sp1.use_count() << std::endl;//2
```



#### volatile

被 `volatile` 修饰的变量，在对其进行读写操作时，会引发一些**可观测的副作用**。而这些可观测的副作用，是由**程序之外的因素决定的**。

##### 应用

（1）并行设备的硬件寄存器（如状态寄存器）。 假设要对一个设备进行初始化，此设备的某一个寄存器为0xff800000。

```c++
int  *output = (unsigned  int *)0xff800000; //定义一个IO端口；  
int   init(void)  
{  
    int i;  
    for(i=0;i< 10;i++)
    {  
    *output = i;  
    }  
}
```

经过编译器优化后，编译器认为前面循环半天都是废话，对最后的结果毫无影响，因为最终只是将output这个指针赋值为 9，所以编译器最后给你编译编译的代码结果相当于：

```c++
int  init(void)  
{  
    *output = 9;  
}
```

如果你对此外部设备进行初始化的过程是必须是像上面代码一样顺序的对其赋值，显然优化过程并不能达到目的。反之如果你不是对此端口反复写操作，而是反复读操作，其结果是一样的，编译器在优化后，也许你的代码对此地址的读操作只做了一次。然而从代码角度看是没有任何问题的。这时候就该使用volatile通知编译器这个变量是一个不稳定的，在遇到此变量时候不要优化。

（2）一个中断服务子程序中访问到的变量；

```c++
static int i=0;

int main()
{
    while(1)
    {
    if(i) dosomething();
    }
}

/* Interrupt service routine */
void IRS()
{
	i=1;
}
```

上面示例程序的本意是产生中断时，由中断服务子程序IRS响应中断，变更程序变量i，使在main函数中调用dosomething函数，但是，由于编译器判断在main函数里面没有修改过i，因此可能只执行一次对从i到某寄存器的读操作，然后每次if判断都只使用这个寄存器里面的“i副本”，导致dosomething永远不会被调用。如果将变量i加上volatile修饰，则编译器保证对变量i的读写操作都不会被优化，从而保证了变量i被外部程序更改后能及时在原程序中得到感知。

（3）多线程应用中被多个任务共享的变量。 当多个线程共享某一个变量时，该变量的值会被某一个线程更改，应该用 volatile 声明。作用是防止编译器优化把变量从内存装入CPU寄存器中，当一个线程更改变量后，未及时同步到其它线程中导致程序出错。volatile的意思是让编译器每次操作该变量时一定要从内存中真正取出，而不是使用已经存在寄存器中的值。示例如下：

```c++
volatile  bool bStop=false;  //bStop 为共享全局变量  
//第一个线程
void threadFunc1()
{
    ...
    while(!bStop){...}
}
//第二个线程终止上面的线程循环
void threadFunc2()
{
    ...
    bStop = true;
}
```

要想通过第二个线程终止第一个线程循环，如果bStop不使用volatile定义，那么这个循环将是一个死循环，因为bStop已经读取到了寄存器中，寄存器中bStop的值永远不会变成FALSE，加上volatile，程序在执行时，每次均从内存中读出bStop的值，就不会死循环了。

是否了解volatile的应用场景是区分C/C++程序员和嵌入式开发程序员的有效办法，搞嵌入式的家伙们经常同硬件、中断、RTOS等等打交道，这些都要求用到volatile变量，不懂得volatile将会带来程序设计的灾难。

##### 常见问题

下面的问题可以看一下面试者是不是直正了解volatile。 

（1）一个参数既可以是const还可以是volatile吗？为什么？ 可以。一个例子是只读的状态寄存器。它是volatile因为它可能被意想不到地改变。它是const因为程序不应该试图去修改它。

（2）一个指针可以是volatile吗？为什么？ 可以。尽管这并不常见。一个例子是当一个中断服务子程序修该一个指向一个buffer的指针时。

（3）下面的函数有什么错误？

```c++
int square(volatile int *ptr) 
{ 
return *ptr * *ptr; 
} 
```

这段代码有点变态，其目的是用来返回指针ptr指向值的平方，但是，由于ptr指向一个volatile型参数，编译器将产生类似下面的代码：

```c++
int square(volatile int *ptr) 
{ 
int a,b; 
a = *ptr; 
b = *ptr; 
return a * b; 
} 
```

由于*ptr的值可能被意想不到地改变，因此a和b可能是不同的。结果，这段代码可能返回的不是你所期望的平方值！正确的代码如下：

```c++
long square(volatile int *ptr) 
{ 
int a=*ptr; 
return a * a; 
} 
```

##### 使用

- volatile 关键字是一种类型修饰符，用它声明的类型变量表示可以被某些编译器未知的因素（操作系统、硬件、其它线程等）更改。所以使用 volatile 告诉编译器不应对这样的对象进行优化。
- volatile 关键字声明的变量，每次访问时都必须从内存中取出值（没有被 volatile 修饰的变量，可能由于编译器的优化，从 CPU 寄存器中取值）
- const 可以是 volatile （如只读的状态寄存器）
- 指针可以是 volatile



#### 深入浅出C++虚函数的vptr与vtable

##### 基础理论

为了实现虚函数，C ++使用一种称为虚拟表的特殊形式的后期绑定。该虚拟表是用于解决在动态/后期绑定方式的函数调用函数的查找表。虚拟表有时会使用其他名称，例如“vtable”，“虚函数表”，“虚方法表”或“调度表”。

虚拟表实际上非常简单，虽然用文字描述有点复杂。首先，**每个使用虚函数的类（或者从使用虚函数的类派生）都有自己的虚拟表**。该表只是编译器在编译时设置的静态数组。虚拟表包含可由类的对象调用的每个虚函数的一个条目。此表中的每个条目只是一个函数指针，指向该类可访问的派生函数。

其次，编译器还会添加一个隐藏指向基类的指针，我们称之为vptr。vptr在创建类实例时自动设置，以便指向该类的虚拟表。与this指针不同，this指针实际上是编译器用来解析自引用的函数参数，vptr是一个真正的指针。

因此，它使每个类对象的分配大一个指针的大小。这也意味着vptr由派生类继承，这很重要。

##### 实现与内部结构

下面我们来看自动与手动操纵vptr来获取地址与调用虚函数！

开始看代码之前，为了方便大家理解，这里给出调用图：

![image-20230403154729485](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230403154729485.png)

代码全部遵循标准的注释风格，相信大家看了就会明白，不明白的话，可以留言！

```c++
/**
 * @file vptr1.cpp
 * @brief C++虚函数vptr和vtable
 * 编译：g++ -g -o vptr vptr1.cpp -std=c++11
 * @author 光城
 * @version v1
 * @date 2019-07-20
 */

#include <iostream>
#include <stdio.h>
using namespace std;

/**
 * @brief 函数指针
 */
typedef void (*Fun)();

/**
 * @brief 基类
 */
class Base
{
    public:
        Base(){};
        virtual void fun1()
        {
            cout << "Base::fun1()" << endl;
        }
        virtual void fun2()
        {
            cout << "Base::fun2()" << endl;
        }
        virtual void fun3(){}
        ~Base(){};
};

/**
 * @brief 派生类
 */
class Derived: public Base
{
    public:
        Derived(){};
        void fun1()
        {
            cout << "Derived::fun1()" << endl;
        }
        void fun2()
        {
            cout << "DerivedClass::fun2()" << endl;
        }
        ~Derived(){};
};
/**
 * @brief 获取vptr地址与func地址,vptr指向的是一块内存，这块内存存放的是虚函数地址，这块内存就是我们所说的虚表
 *
 * @param obj
 * @param offset
 *
 * @return 
 */
Fun getAddr(void* obj,unsigned int offset)
{
    cout<<"======================="<<endl;
    void* vptr_addr = (void *)*(unsigned long *)obj;  //64位操作系统，占8字节，通过*(unsigned long *)obj取出前8字节，即vptr指针
    printf("vptr_addr:%p\n",vptr_addr);
    
    /**
     * @brief 通过vptr指针访问virtual table，因为虚表中每个元素(虚函数指针)在64位编译器下是8个字节，因此通过*(unsigned long *)vptr_addr取出前8字节，
     * 后面加上偏移量就是每个函数的地址！
     */
    void* func_addr = (void *)*((unsigned long *)vptr_addr+offset);
    printf("func_addr:%p\n",func_addr);
    return (Fun)func_addr;
}
int main(void)
{
    Base ptr;
    Derived d;
    Base *pt = new Derived(); // 基类指针指向派生类实例
    Base &pp = ptr; // 基类引用指向基类实例
    Base &p = d; // 基类引用指向派生类实例
    cout<<"基类对象直接调用"<<endl;
    ptr.fun1();
    cout<<"基类对象调用基类实例"<<endl;
    pp.fun1(); 
    cout<<"基类指针指向基类实例并调用虚函数"<<endl;
    pt->fun1();
    cout<<"基类引用指向基类实例并调用虚函数"<<endl;
    p.fun1();
    
    // 手动查找vptr 和 vtable
    Fun f1 = getAddr(pt, 0);
    (*f1)();
    Fun f2 = getAddr(pt, 1);
    (*f2)();
    delete pt;
    return 0;
}
```

运行结果：

```
基类对象直接调用
Base::fun1()
基类引用指向派生类实例
Base::fun1()
基类指针指向派生类实例并调用虚函数
Derived::fun1()
基类引用指向基类实例并调用虚函数
Derived::fun1()
=======================
vptr_addr:0x401130
func_addr:0x400ea8
Derived::fun1()
=======================
vptr_addr:0x401130
func_addr:0x400ed4
DerivedClass::fun2()
```

我们发现C++的动态多态性是通过虚函数来实现的。简单的说，通过virtual函数，指向子类的基类指针可以调用子类的函数。例如，上述通过基类指针指向派生类实例，并调用虚函数，将上述代码简化为：

```c++
Base *pt = new Derived(); // 基类指针指向派生类实例
cout<<"基类指针指向派生类实例并调用虚函数"<<endl;
pt->fun1();
```

其过程为：首先程序识别出fun1()是个虚函数，其次程序使用pt->vptr来获取Derived的虚拟表。第三，它查找Derived虚拟表中调用哪个版本的fun1()。这里就可以发现调用的是Derived::fun1()。因此pt->fun1()被解析为Derived::fun1()!

### c++并发

#### std::format

cout方式效率低：每次cout << endl，都会刷新一遍输出缓冲区。

cout方式线程不安全：

```c++
std::cout << "Hello " << "World "; 
```

上面代码其实等同于：

```c++
std::operator<<(std::operator<<(std::cout, "Hello "), "World "); 
```

相当于调用了两次operator<<，不保证调用两次operator<<是线程安全的。

cout配合format(c++20)。

```c++
std::cout << std::format("{} + {} = {} \n", a, b, c); 
```

或存到字符串里。

#### std::thread  

1、默认构造函数，创建⼀个空的 thread 执⾏对象。  

2、初始化构造函数，创建⼀个 thread对象，该 thread对象可被 joinable，新产⽣的线程会 调⽤ fn 函数，该函数的参数由 args 给出。  

3、拷贝构造函数(被禁⽤)，意味着 thread 不可被拷贝构造。

 4、move 构造函数，move 构造函数，调⽤成功之后 x 不代表任何 thread 执⾏对象。  

注意： 可被 joinable 的 thread 对象必须在他们销毁之前被主线程 join 或者将其设置为 detached.  

std::thread在使⽤上容易出错，即std::thread对象在线程函数运⾏期间必须是有效的。什么意思呢？

```c++
#include <iostream>
#include <thread>
void threadproc() {
 while(true) {
 std::cout << "I am New Thread!" << std::endl;
 }
}
void func() {
 std::thread t(threadproc);
    //t.detach();
}
int main() {
 func();
 while(true) {} //让主线程不要退出
 return 0;
}
以上代码再main函数中调⽤了func函数，在func函数中创建了⼀个线程，乍⼀看好像没有什么问题，但在实际运⾏是会崩溃。
崩溃的原因：
在func函数调⽤结束后，func中局部变量t（线程对象）被销毁，⽽此时线程函数仍在运⾏。所以在使⽤std::thread类时，必须保证线程函数运⾏期间其线程对象有效。
std::thread对象提供了⼀个detach⽅法，通过这个⽅法可以让线程对象与线程函数脱离关系，这样即使线程对象被销毁，也不影响线程函数的运⾏。
只需要在func函数中调⽤detach⽅法即可。
```

#### lock_guard

lock_guard是⼀个互斥量包装程序，它提供了⼀种⽅便的RAII风格的机制来在作⽤域块的持续时间内拥有⼀个互斥量。

创建lockguard对象时，它将尝试获取提供给它的互斥锁的所有权。当控制流离开lockguard对象的作⽤域时，lock_guard析构并释放互斥量。 

它的特点如下： 

1. 创建即加锁，作⽤域结束⾃动析构并解锁，⽆需⼿⼯解锁 
2. 不能中途解锁，必须等作⽤域结束才解锁 
3. 不能复制 

#### unique_lock 

unique_lock是⼀个通⽤的互斥量锁定包装器，它允许延迟锁定，限时深度锁定，递归锁定，锁定所有权的转移以及与条件变量⼀起使⽤。  

简单地讲，uniquelock 是 lockguard 的升级加强版，它具有lock_guard 的所有功能，同时 又具有其他很多⽅法，使⽤起来更强灵活⽅便，能够应对更复杂的锁定需要。  

特点如下： 

1. 创建时可以不锁定（通过指定第⼆个参数为std::defer_lock），⽽在需要时再锁定  
2. 可以随时加锁解锁  
3. 作⽤域规则同 lock_grard，析构时⾃动释放锁  
4. 不可复制，可移动 
5.  条件变量需要该类型的锁作为参数（此时必须使⽤unique_lock） 

#### std::scoped_lock

类 `scoped_lock` 是提供便利 RAII 风格机制的互斥包装器，它在作用域块的存在期间占有一或多个互斥。

创建 `scoped_lock` 对象时，它试图取得给定互斥的所有权。控制离开创建 `scoped_lock` 对象的作用域时，析构 `scoped_lock` 并释放互斥。若给出数个互斥，则使用免死锁算法，如同以 [std::lock](https://zh.cppreference.com/w/cpp/thread/lock) 。

`scoped_lock` 类不可复制。

#### std::atomic 

a++ 和 int a = b 在C++中不是是线程安全的

C++11新标准发布后改变了这种困境，新标准提供了对整形变量原⼦操作的相 关库，即std::atomic，这是⼀个模板类型：

我们可以传⼊具体的整型类型对模板进⾏实例化，实际上stl库也提供了这些实例化的模板类型 

```c++
// 初始化1 
std::atomic value; 
value = 99; 
// 初始化2 
// 下⾯代码在Linux平台上⽆法编译通过（指在gcc编译器） 
std::atomic<int> value = 99;
// 出错的原因是这⾏代码调⽤的是std::atomic的拷⻉构造函数 
// ⽽根据C++11语⾔规范，std::atomic的拷⻉构造函数使⽤=delete标记禁⽌编译器⾃动⽣成 
// g++在这条规则上遵循了C++11语⾔规范。
```

#### 内存模型

##### **计算机中的乱序执⾏**

1、⼀定会按正常顺序执⾏的情况 

 	1. 对同⼀块内存进⾏访问，此时访问的顺序不会被编译器修改
 	2.   新定义的变量的值依赖于之前定义的变量，此时两个变量定义的顺序不会被编译器 修改 

2、其他情况计算机会进⾏乱序执⾏ 

单线程的情况下允许，但是多线程情况下就会产⽣问题

##### **C++中的库中提供了六种内存模型** 

⽤于在多线程的情况下防⽌编译器的乱序执⾏ 

（1）memory_order_relaxed 最放松的 

（2）memory_order_consume 当客户使⽤，搭配release使⽤，被release进⾏赋值的变量y，获取的时候如果写成 consume，那么所有与y有关的变量的赋值⼀定会被按顺序进⾏ 

（3）memory_order_acquire ⽤于获取资源  

（4）memory_order_release ⼀般⽤于⽣产者，当给⼀个变量y进⾏赋值的时候，只有⾃⼰将这个变量释放了，别⼈才可 以去读，读的时候如果使⽤acquire来读，编译器会保证在y之前被赋值的变量的赋值都在y 之前被执⾏，相当于设置了内存屏障 

（5）memory_order_acq_rel（acquire/release）  

（6）memory_order_seq_cst（squentially consistent）  

好处：不需要编译器设置内存屏障，morden c++开始就会有底层汇编的能⼒

#### 副作⽤ 

1、⽆副作⽤编程 

存在⼀个函数，传⼀个参数x进去，⾥⾯进⾏⼀系列的运算，返回⼀个y。中间的所有过程 都是在栈中进⾏修改 

 2、有副作⽤编程

 ⽐如在⼀个函数运⾏的过程中对全局变量进⾏了修改或在屏幕上输出了⼀些东西。此函数还 有可能是类的成员⽅法，在此⽅法中如果对成员变量进⾏了修改，类的状态就会发⽣改变 

 3、在多线程情况下的有副作⽤编程

在线程1运⾏的时候对成员变量进⾏了修改，此时如果再继续运⾏线程2，此时线程2拥有的 就不是这个类的初始状态，运⾏出来的结果会收到线程1的影响 

解决办法：将成员⽅法设为const，此时就可以放⼼进⾏调⽤ 

#### 信号量 c++20

**1、binary_semaphore**  

定义： 可以当事件来⽤，只有有信号和⽆信号两种状态，⼀次只能被⼀个线程所持有。  

使⽤步骤：  

（1）初始创建信号量，并且⼀开始将其置位成⽆信号状态 std::binary_semaphore sem(0) 

（2）线程使⽤acquire()⽅法等待被唤醒  

（3）主线程中使⽤release()⽅法，将信号量变成有信号状态  

**2、counting_semaphore** 

定义： ⼀次可以被很多线程所持有，线程的数量由⾃⼰指定 

使⽤步骤： 

（1）创建信号量 

指定⼀次可以进⼊的线程的最⼤数量，并在最开始将其置位成⽆信号状态： std::counting_semaphore<8> sem(0);  

（2）主线程中创建10个线程 

并且这些线程全部调⽤acquire()⽅法等待被唤醒。但是主线程使⽤release(6)⽅法就只能随 机启⽤6个线程。

#### future库

⽤于任务链（即任务A的执⾏必须依赖于任务B的返回值） 

**1、例⼦：⽣产者消费者问题** 

（1）⼦线程作为消费者 

参数是⼀个future，⽤这个future等待⼀个int型的产品：std::future& fut  

（2）⼦线程中使⽤get()⽅法等待⼀个未来的future，返回⼀个result  

（3）主线程作为⽣产者,做出⼀个承诺：std::promise prom  

（4）⽤此承诺中的get_future()⽅法获取⼀个future  

（5）主线程中将⼦线程创建出来,并将刚刚获取到的future作为参数传⼊  

（6）主线程做⼀些列的⽣产⼯作,最后⽣产完后使⽤承诺中的set_value()⽅法，参数为刚刚 ⽣产出的产品  

（7）此时产品就会被传到⼦线程中,⼦线程就可以使⽤此产品做⼀系列动作  

（8）最后使⽤join()⽅法等待⼦线程停⽌,但是join只适⽤于等待没有返回值的线程的情况  

**2、如果线程有返回值** 

（1）使⽤async⽅法可以进⾏异步执⾏  

参数⼀： 可以选择是马上执⾏还是等⼀会执⾏（即当消费者线程调⽤get()⽅法时才开始执 ⾏）  

参数⼆： 执⾏的内容（可以放⼀个函数对象或lambda表达式）  

（2）⽣产者使⽤async⽅法做⽣产⼯作并返回⼀个future  

（3）消费者使⽤future中的get（）⽅法可以获取产品



```c++
#include <vector>
#include <thread>
#include <future>
#include <numeric>
#include <iostream>
#include <chrono>
 
void accumulate(std::vector<int>::iterator first,
                std::vector<int>::iterator last,
                std::promise<int> accumulate_promise)
{
    int sum = std::accumulate(first, last, 0);
    accumulate_promise.set_value(sum);  // 提醒 future
}
 
void do_work(std::promise<void> barrier)
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
    barrier.set_value();
}
 
int main()
{
    // 演示用 promise<int> 在线程间传递结果。
    std::vector<int> numbers = { 1, 2, 3, 4, 5, 6 };
    std::promise<int> accumulate_promise;
    std::future<int> accumulate_future = accumulate_promise.get_future();
    std::thread work_thread(accumulate, numbers.begin(), numbers.end(),
                            std::move(accumulate_promise));
 
    // future::get() 将等待直至该 future 拥有合法结果并取得它
    // 无需在 get() 前调用 wait()
    //accumulate_future.wait();  // 等待结果
    std::cout << "result=" << accumulate_future.get() << '\n';
    work_thread.join();  // wait for thread completion
 
    // 演示用 promise<void> 在线程间对状态发信号
    std::promise<void> barrier;
    std::future<void> barrier_future = barrier.get_future();
    std::thread new_work_thread(do_work, std::move(barrier));
    barrier_future.wait();
    new_work_thread.join();
}
```

### New features

#### Structured binding

```c++
Before:
auto p = std::make_pair(“s”, 5);
string a = s.first;
int b = s.second;


After:
auto p = std::make_pair(“s”, 5);
auto [a, b] = p;
// a is of type string
// b is of type int

// auto [a, b] = std::make_pair(...);


//quadratic
#include <cmath>
#include <iostream>
std::pair<bool, std::pair<double, double>> quadratic(int a, int b, int c) {
  double inside = b * b - 4 * a * c;
  std::pair<double, double> blank;
  if (inside < 0) return std::make_pair(false, blank);
  auto answer =
      std::make_pair((-b + sqrt(inside)) / 2, (-b - sqrt(inside)) / 2);
  return std::make_pair(true, answer);
}

int main() {
  int a, b, c;
  std::cin >> a >> b >> c;
  auto [found, solutions] = quadratic(a, b, c);  // Structured binding
  if (found) {
    auto [x1, x2] = solutions;
    std::cout << x1 << " " << x2 << std::endl;
  } else {
    std::cout << "No solutions found !"<< std::endl;
  }
  
}
```

#### Uniform Initialization

```c++
eg1:
Before 
std::pair<bool, int> some_pair = 
std::make_pair(false, 6);
Student s;
s.name = “Ethan”;
s.state = “CA”;
s.age = 20;

After
std::pair<bool, int> some_pair{false, 6};
Student s{"Ethan", "CA", 20};

eg2:
struct Course {
 std::string code;
 std::pair<Time, Time> time;
 std::vector<std::string> instructors;
};
struct Time {
 int hour, minute;
};
...
Course now{“CS106L”, { {14, 30}, {15, 50} }, {“Raghuraman”, “Chi”} };
```

注意：

```c++
int n = 3;
int k = 5;
//This initialization uses a constructor
std::vector<int> v(n, k); // {5, 5, 5}

std::vector<int> v2{n, k}; // {3, 5} -- not the same!!
```



#### Variadic templates

```c++
template<typename T>
T adder(T v) {
 return v;
}
template<typename T, typename... Args>
T adder(T first, Args... args) {
 return first + adder(args...);
}
adder(5, 6, 7, 8) // 26
```



#### Spaceship operator

```c++
Before
struct IntWrapper {
 int value;
 IntWrapper(int value): value{value} { }
 bool operator==(const IntWrapper& rhs) const { return value == rhs.value; }
 bool operator!=(const IntWrapper& rhs) const { return !(*this == rhs); }
 bool operator<(const IntWrapper& rhs) const { return value < rhs.value; }
 bool operator<=(const IntWrapper& rhs) const { return !(rhs < *this); }
 bool operator>(const IntWrapper& rhs) const { return rhs < *this; }
 bool operator>=(const IntWrapper& rhs) const { return !(*this < rhs); }
};
After
struct IntWrapper {
 int value;
 IntWrapper(int value): value(value) { }
 auto operator<=>(const int& rhs) auto { 
 return value <=> rhs;
 }
};
or
struct IntWrapper {
 int value;
 IntWrapper(int value): value(value) { }
 auto operator<=>(const int& rhs) auto { 
 if (value < rhs) return -1;
 else if (value == rhs) return 0;
 else return 1;
 }
};
IntWrapper(5) < IntWrapper(7) // returns true
```



#### Designated initializers

```c++
struct A {
 int x;
 int y;
 int z = 123;
};
A a {.x = 1, .z = 2}; // a.x == 1, a.y == 0, a.z == 2
```



#### [[likely]]

Use the [[likely]] operator to mark things that probably will run...

```c++
int random = get_random_number_between_x_and_y(0, 100);
[[likely]] if (random > 0) {
 // body of if statement; efficiency will be prioritized
}
[[unlikely]] if (random == 0) {
 // body of if statement; efficiency will not be prioritized
}
```





## 面试问题

#### 指针与引用

指针存放某个对象的地址，其本⾝就是变量（命了名的对象），本⾝就有地址，所以可以有 指向指针的指针；可变，包括其所指向的地址的改变和其指向的地址中所存放的数据的改变 

引⽤就是变量的别名，从⼀⽽终，不可变，必须初始化 

不存在指向空值的引⽤，但是存在指向空值的指针

#### define 和 typedef 、inline、const的区别

| 宏定义 #define         | const 常量     |
| ---------------------- | -------------- |
| 宏定义，相当于字符替换 | 常量声明       |
| 预处理器处理           | 编译器处理     |
| 无类型安全检查         | 有类型安全检查 |
| 不分配内存             | 要分配内存     |
| 存储在代码段           | 存储在数据段   |
| 可通过 `#undef` 取消   | 不可取消       |

 **define** 

1. 只是简单的字符串替换，没有类型检查 
2. 是在编译的预处理阶段起作⽤ 
3. 可以⽤来防⽌头⽂件重复引⽤
4. 不分配内存，给出的是⽴即数，有多少次使⽤就进⾏多少次替换

**typedef** 

1. 有对应的数据类型，是要进⾏判断的 
2. 是在编译、运⾏的时候起作⽤
3. 在静态存储区中分配空间，在程序运⾏过程中内存中只有⼀个拷贝 

**inline**

 inline是先将内联函数编译完成⽣成了函数体直接插⼊被调⽤的地⽅，减少了压栈，跳转和 返回的操作。没有普通函数调⽤时的额外开销； 

内联函数是⼀种特殊的函数，会进⾏类型检查； 

对编译器的⼀种请求，编译器有可能拒绝这种请求；  

C++中inline编译限制： 

1. 不能存在任何形式的循环语句
2. 不能存在过多的条件判断语句 
3. 函数体不能过于庞⼤ 
4. 内联函数声明必须在调⽤语句之前

#### override 和 overload 

##### **override是重写（覆盖）了⼀个⽅法** 

以实现不同的功能，⼀般是⽤于⼦类在继承⽗类时，重写⽗类⽅法。

虚函数可以让我们在子类中重写方法。 

规则： 

 	1. 重写⽅法的参数列表，返回值，所抛出的异常与被重写⽅法⼀致 
 	2. 被重写的⽅法不能为private 
 	3. 静态⽅法不能被重写为⾮静态的⽅法
 	4.  重写⽅法的访问修饰符⼀定要⼤于被重写⽅法的访问修饰符 （public>protected>default>private） 

##### overload是重载

这些⽅法的名称相同⽽参数形式不同,⼀个⽅法有不同的版本，存在于⼀个类中。 

规则： 

1. 不能通过访问权限、返回类型、抛出的异常进⾏重载 
2. 不同的参数类型可以是不同的参数类型，不同的参数个数，不同的参数顺序（参数 类型必须不⼀样） 
3. ⽅法的异常类型和数⽬不会对重载造成影响 

使⽤多态是为了避免在⽗类⾥⼤量重载引起代码臃肿且难于维护。 

重写与重载的本质区别是,加⼊了override的修饰符的⽅法,此⽅法始终只有⼀个被你使⽤的⽅法。 

#### 多态

- 多态，即多种状态（形态）。简单来说，我们可以将多态定义为消息以多种形式显示的能力。

- 多态是以封装和继承为基础的。

- C++ 多态分类及实现：

  1. 重载多态（Ad-hoc Polymorphism，编译期）：函数重载、运算符重载

  2. 子类型多态（Subtype Polymorphism，运行期）：虚函数

     ```
     允许将⼦类类型的指针赋值给⽗类类型的指针。
     
     虚函数依赖虚函数表⼯作，表来保存虚函数地址，当我们⽤基类指针指向派⽣类时，虚表指针指向派⽣类的虚函数表.
     
     这个机制可以保证派⽣类中的虚函数被调⽤到.
     ```

     

  3. 参数多态性（Parametric Polymorphism，编译期）：类模板、函数模板

  4. 强制多态（Coercion Polymorphism，编译期/运行期）：基本类型转换、自定义类型转换

#### new 和 malloc 

1、new内存分配失败时，会抛出bac_alloc异常，它不会返回NULL；malloc分配内存失败时 返回NULL。  

2、使⽤new操作符申请内存分配时⽆须指定内存块的⼤⼩，⽽malloc则需要显式地指出所 需内存的尺⼨。  

3、opeartor new /operator delete可以被重载，⽽malloc/free并不允许重载。  

4、new/delete会调⽤对象的构造函数/析构函数以完成对象的构造/析构。⽽malloc则不会  

5、malloc与free是C++/C语⾔的标准库函数,new/delete是C++的运算符  

6、new操作符从⾃由存储区上为对象动态分配内存空间，⽽malloc函数从堆上动态分配内 存。

![image-20230403160107203](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230403160107203.png)

#### delete this 合法吗

合法，但：

1. 必须保证 this 对象是通过 `new`（不是 `new[]`、不是 placement new、不是栈上、不是全局、不是其他对象成员）分配的
2. 必须保证调用 `delete this` 的成员函数是最后一个调用 this 的成员函数
3. 必须保证成员函数的 `delete this ` 后面没有调用 this 了
4. 必须保证 `delete this` 后没有人使用了

#### 前置++与后置++ 

```c++
self &operator++() { 
    node = (linktype)((node).next); 
    return *this; 
} 
const self operator++(int) { 
    self tmp = *this; 
    ++*this; 
    return tmp; 
} 
```

为了区分前后置，重载函数是以参数类型来区分，在调⽤的时候，编译器默默给int指定为 ⼀个0

1、为什么后置返回对象，⽽不是引⽤ 

因为后置为了返回旧值创建了⼀个临时对象，在函数结束的时候这个对象就会被销毁，如果 返回引⽤，那么我请问你？你的对象对象都被销毁了，你引⽤啥呢？ 

2、为什么后置前⾯也要加const 

其实也可以不加，但是为了防⽌你使⽤i++++,连续两次的调⽤后置++重载符，为什么呢? 

原因： 它与内置类型⾏为不⼀致；你⽆法活得你所期望的结果，因为第⼀次返回的是旧值，⽽不是 原对象，你调⽤两次后置++，结果只累加了⼀次，所以我们必须⼿动禁⽌其合法化，就要 在前⾯加上const。 

3、处理⽤户的⾃定义类型 

最好使⽤前置++，因为他不会创建临时对象，进⽽不会带来构造和析构⽽造成的格外开销。 

#### 如何定义一个只能在堆上（栈上）生成对象的类

##### 只能在堆上

方法：将析构函数设置为私有

原因：C++ 是静态绑定语言，编译器管理栈上对象的生命周期，编译器在为类对象分配栈空间时，会先检查类的析构函数的访问性。若析构函数不可访问，则不能在栈上创建对象。

##### 只能在栈上

方法：将 new 和 delete 重载为私有

原因：在堆上生成对象，使用 new 关键词操作，其过程分为两阶段：第一阶段，使用 new 在堆上寻找可用内存，分配给对象；第二阶段，调用构造函数生成对象。将 new 操作设置为私有，那么第一阶段就无法完成，就不能够在堆上生成对象。

#### 为何空类的⼤⼩不是0 

为了确保两个不同对象的地址不同，必须如此。 

类的实例化是在内存中分配⼀块地址，每个实例在内存中都有独⼀⽆⼆的⼆地址。  

同样，空类也会实例化，所以编译器会给空类隐含的添加⼀个字节，这样空类实例化后就有 独⼀⽆⼆的地址了。  

所以，空类的sizeof为1，⽽不是0。

```c++
class A{ virtual void f(){} }; 
class B:public A{} 
此时，类A和类B都不是空类，其sizeof都是4，因为它们都具有虚函数表的地址。 

class A{}; 
class B:public virtual A{};
此时，A是空类，其⼤⼩为1；B不是空类，其⼤⼩为4.因为含有指向虚基类的指针。
    
多重继承的空类的⼤⼩也是1。
class Father1{}; class Father2{};
class Child:Father1, Father2{};
它们的sizeof都是1. 
何时共享虚函数地址表： 
如果派⽣类继承的第⼀个是基类，且该基类定义了虚函数地址表，则派⽣类就共享该表⾸址占⽤的存储单元。 
    对于除前述情形以外的其他任何情形，派⽣类在处理完所有基类或虚基类后，根据派⽣类是
否建⽴了虚函数地址表，确定是否为该表⾸址分配存储单元。
class X{}; //sizeof(X):1
class Y : public virtual X {}; //sizeof(Y):4
class Z : public virtual X {}; //sizeof(Z):4
class A : public virtual Y {}; //sizeof(A):8
class B : public Y, public Z{}; //sizeof(B):8
class C : public virtual Y, public virtual Z {}; //sizeof(C):12
class D : public virtual C{}; //sizeof(D):1
```

#### 内存泄漏

**什么是内存泄露?** 

内存泄漏(memory leak)是指由于疏忽或错误造成了程序未能释放掉不再使⽤的内存的情 况。内存泄漏并⾮指内存在物理上的消失，⽽是应⽤程序分配某段内存后，由于设计错误， 失去了对该段内存的控制，因⽽造成了内存的浪费。 

可以使⽤Valgrind, mtrace进⾏内存泄漏检查。 

**内存泄漏的分类** 

（1）堆内存泄漏 （Heap leak）  

对内存指的是程序运⾏中根据需要分配通过malloc,realloc new等从堆中分配的⼀块内存， 再是完成后必须通过调⽤对应的 free或者 delete 删掉。如果程序的设计的错误导致这部分 内存没有被释放，那么此后这块内存将不会被使⽤，就会产⽣ Heap Leak.  

（2）系统资源泄露（Resource Leak）  

主要指程序使⽤系统分配的资源⽐如 Bitmap,handle ,SOCKET 等没有使⽤相应的函数释放 掉，导致系统资源的浪费，严重可导致系统效能降低，系统运⾏不稳定。  

（3）没有将基类的析构函数定义为虚函数  

当基类指针指向⼦类对象时，如果基类的析构函数不是 virtual，那么⼦类的析构函数将不 会被调⽤，⼦类的资源没有正确是释放，因此造成内存泄露。

**什么操作会导致内存泄露?** 

指针指向改变，未释放动态分配内存。 

**如何防⽌内存泄露?** 

将内存的分配封装在类中，构造函数分配内存，析构函数释放内存；使⽤智能指针 。RAII思想。

**智能指针有了解哪些?** 

智能指针是为了解决动态分配内存导致内存泄露和多次释放同⼀内存所提出的，C11标准中 放在< memory>头⽂件。包括:共享指针，独占指针，弱指针  

**构造函数，析构函数要设为虚函数吗，为什么？** 

（1）析构函数 析构函数需要。当派⽣类对象中有内存需要回收时，如果析构函数不是虚函数，不会触发动 态绑定，只会调⽤基类析构函数，导致派⽣类资源⽆法释放，造成内存泄漏。 

（2）构造函数 构造函数不需要，没有意义。虚函数调⽤是在部分信息下完成⼯作的机制，允许我们只知道 接⼜⽽不知道对象的确切类型。 要创建⼀个对象，你需要知道对象的完整信息。 特别是， 你需要知道你想要创建的确切类型。 因此，构造函数不应该被定义为虚函数。

**野指针**

使用`free`函数释放内存后，该指针的值不会被自动置为`NULL`，它仍然指向释放掉的那块内存，这种指向已经释放的内存的指针被称为“野指针”。

### 手撕系列

#### 单例模式

```c++
#include <iostream>

using namespace std;

class singleton {
private:
    singleton() {}
public:
    static singleton &instance();
};

singleton &singleton::instance() {
    static singleton p;
    return p;
}
- 单线程下，正确。
- C++11及以后的版本（如C++14）的多线程下，正确。
- C++11之前的多线程下，不一定正确。
原因在于在C++11之前的标准中并没有规定local static变量的内存模型。于是乎它就是不是线程安全的了。但是在C++11却是线程安全的，这是因为新的C++标准规定了当一个线程正在初始化一个变量的时候，其他线程必须得等到该初始化完成以后才能访问它。
```



#### 实现内存拷贝函数

**假如考虑dst和src内存重叠的情况，strcpy该怎么实现**

所谓重叠，就是src未处理的部分已经被dst给覆盖了，只有⼀种情 况：src<=dst<=src+strlen(src)

```c++
char* strcpy(char *dst,const char *src) {
 assert(dst != NULL && src != NULL);
 char *ret = dst;
 while ((*dst++=*src++)!='\0');
 //或者my_memcpy(dst, src, strlen(src)+1);
 //C函数 memcpy ⾃带内存重叠检测功能
 return ret;
}

/* my_memcpy的实现如下 */
char *my_memcpy(char *dst, const char* src, int cnt)
{
 assert(dst != NULL && src != NULL);
 char *ret = dst;
 /*内存重叠，从⾼地址开始复制*/
 if (dst >= src && dst <= src+cnt-1)
 {
 	dst = dst+cnt-1;
 	src = src+cnt-1;
 	while (cnt--)
 	{
 		*dst-- = *src--;
 	}
 }
 else //正常情况，从低地址开始复制
 {
 	while (cnt--)
 	{
 		*dst++ = *src++;
 	}
 }
 return ret;
}

```

已知String的原型为：

```c++
class String
{
public:
 String(const char *str = NULL);
 String(const String &other);
 ~ String(void);
 String & operate =(const String &other);
private:
 char *m_data;
};
```

请编写上述四个函数

```c++
// 构造函数
String::String(const char *str)
{
 if(str==NULL)
 {
 m_data = new char[1]; //对空字符串⾃动申请存放结束标志'\0'
 *m_data = '\0';
 } 
 else
 {
 int length = strlen(str);
 m_data = new char[length + 1];
 strcpy(m_data, str);
 }
}
// 析构函数
String::~String(void)
{
 delete [] m_data; // 或delete m_data;
}
//拷⻉构造函数
String::String(const String &other)
{
 int length = strlen(other.m_data);
 m_data = new char[length + 1];
 strcpy(m_data, other.m_data);
}
//赋值函数
String &String::operate =(const String &other)
{ 
 if(this == &other)
 {
 return *this; // 检查⾃赋值
 } 
 delete []m_data; // 释放原有的内存资源
 int length = strlen(other.m_data);
 m_data = new char[length + 1]; //对m_data加NULL判断
 strcpy(m_data, other.m_data); 
 return *this; //返回本对象的引⽤
}

```

### 算法

#### 排序

| 排序算法                                                     | 平均时间复杂度 | 最差时间复杂度 | 空间复杂度 | 数据对象稳定性       |
| ------------------------------------------------------------ | -------------- | -------------- | ---------- | -------------------- |
| 冒泡排序                                                     | O(n2)          | O(n2)          | O(1)       | 稳定                 |
| 选择排序                                                     | O(n2)          | O(n2)          | O(1)       | 数组不稳定、链表稳定 |
| [插入排序](https://interview.huihut.com/#/Algorithm/InsertSort.h) | O(n2)          | O(n2)          | O(1)       | 稳定                 |
| [快速排序](https://interview.huihut.com/#/Algorithm/QuickSort.h) | O(n*log2n)     | O(n2)          | O(log2n)   | 不稳定               |
| [堆排序](https://interview.huihut.com/#/Algorithm/HeapSort.cpp) | O(n*log2n)     | O(n*log2n)     | O(1)       | 不稳定               |
| [归并排序](https://interview.huihut.com/#/Algorithm/MergeSort.h) | O(n*log2n)     | O(n*log2n)     | O(n)       | 稳定                 |
| [希尔排序](https://interview.huihut.com/#/Algorithm/ShellSort.h) | O(n*log2n)     | O(n2)          | O(1)       | 不稳定               |
| [计数排序](https://interview.huihut.com/#/Algorithm/CountSort.cpp) | O(n+m)         | O(n+m)         | O(n+m)     | 稳定                 |
| [桶排序](https://interview.huihut.com/#/Algorithm/BucketSort.cpp) | O(n)           | O(n)           | O(m)       | 稳定                 |
| [基数排序](https://interview.huihut.com/#/Algorithm/RadixSort.h) | O(k*n)         | O(n2)          |            | 稳定                 |

> - 均按从小到大排列
> - k：代表数值中的 “数位” 个数
> - n：代表数据规模
> - m：代表数据的最大值减最小值
> - 来自：[wikipedia . 排序算法](https://zh.wikipedia.org/wiki/排序算法)

#### 查找

| 查找算法                                                     | 平均时间复杂度   | 空间复杂度 | 查找条件   |
| ------------------------------------------------------------ | ---------------- | ---------- | ---------- |
| [顺序查找](https://interview.huihut.com/#/Algorithm/SequentialSearch.h) | O(n)             | O(1)       | 无序或有序 |
| [二分查找（折半查找）](https://interview.huihut.com/#/Algorithm/BinarySearch.h) | O(log2n)         | O(1)       | 有序       |
| [插值查找](https://interview.huihut.com/#/Algorithm/InsertionSearch.h) | O(log2(log2n))   | O(1)       | 有序       |
| [斐波那契查找](https://interview.huihut.com/#/Algorithm/FibonacciSearch.cpp) | O(log2n)         | O(1)       | 有序       |
| [哈希查找](https://interview.huihut.com/#/DataStructure/HashTable.cpp) | O(1)             | O(n)       | 无序或有序 |
| [二叉查找树（二叉搜索树查找）](https://interview.huihut.com/#/Algorithm/BSTSearch.h) | O(log2n)         |            |            |
| [红黑树](https://interview.huihut.com/#/DataStructure/RedBlackTree.cpp) | O(log2n)         |            |            |
| 2-3树                                                        | O(log2n - log3n) |            |            |
| B树/B+树                                                     | O(log2n)         |            |            |

#### 图搜索算法

| 图搜索算法                                                   | 数据结构          | 遍历时间复杂度           | 空间复杂度               |
| ------------------------------------------------------------ | ----------------- | ------------------------ | ------------------------ |
| [BFS广度优先搜索](https://zh.wikipedia.org/wiki/广度优先搜索) | 邻接矩阵 邻接链表 | O(\|v\|2) O(\|v\|+\|E\|) | O(\|v\|2) O(\|v\|+\|E\|) |
| [DFS深度优先搜索](https://zh.wikipedia.org/wiki/深度优先搜索) | 邻接矩阵 邻接链表 | O(\|v\|2) O(\|v\|+\|E\|) | O(\|v\|2) O(\|v\|+\|E\|) |

#### 其他算法

| 算法                                               | 思想                                                         | 应用                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [分治法](https://zh.wikipedia.org/wiki/分治法)     | 把一个复杂的问题分成两个或更多的相同或相似的子问题，直到最后子问题可以简单的直接求解，原问题的解即子问题的解的合并 | [循环赛日程安排问题](https://github.com/huihut/interview/tree/master/Problems/RoundRobinProblem)、排序算法（快速排序、归并排序） |
| [动态规划](https://zh.wikipedia.org/wiki/动态规划) | 通过把原问题分解为相对简单的子问题的方式求解复杂问题的方法，适用于有重叠子问题和最优子结构性质的问题 | [背包问题](https://github.com/huihut/interview/tree/master/Problems/KnapsackProblem)、斐波那契数列 |
| [贪心法](https://zh.wikipedia.org/wiki/贪心法)     | 一种在每一步选择中都采取在当前状态下最好或最优（即最有利）的选择，从而希望导致结果是最好或最优的算法 | 旅行推销员问题（最短路径问题）、最小生成树、哈夫曼编码       |

# 语法知识点

## **Static**

把内部变量改变为静态内部变量后，改变了它的生存期，但作用域未变。

把外部变量改变为静态外部变量后，改变了它的作用域（不能被其他文件访问），但生存期未变。

其它源文件中的函数，引用非静态外部变量时，需要在引用函数所在的源文件中（通常在文件开头）进行说明：extern 数据类型 全局变量表；

## C++中的局部静态（Local Static）

> 在局部作用域中可以使用static来声明一个变量，这和前两种有所不同。这一种情况需要考虑变量的生命周期和作用域。
> 生命周期：变量实际存在的时间；作用域：指可以访问变量的范围。
> 静态局部（local static）变量允许我们声明一个变量，它的生命周期基本相当于整个程序的生命周期，然而它的作用范围被限制在这个作用域内。

例如：

```cpp
#include <iostream>

void Function（）
{   //这句的意思是当我第一次调用这个函数时它的值被初始化为0,后续调用不会再创建一个新的变量
 static int i = 0；
 i++；  //如果上一行没有static结果会是每次调用这个函数i的值被设为0，然后i自增1向控制台输出1
 std::cout << i << std::endl；
}

int main()
{
 Function（）；
 Function（）；
 Function（）；
 std:：cin.get（）
}
```

输出 1 2 3

这其实就如同在函数外声明一个全局变量：

```cpp
#include <iostream>
int i = 0；//声明一个全局变量
void Function（）
{ 
 i++； 
 std::cout << i << std::endl；
}

int main()
{
 Function（）；
 Function（）；
 Function（）； //输出 1 2 3
 std:：cin.get（）
}
```

但是这种问题是：可以**在任何地方**访问到变量 i

```cpp
int main()
{
 Function（）；
 i = 10;     // 可以在函数之间改变i的值
 Function（）；
 Function（）； //输出 1 11 12
 std::cin.get（）
}
```

因此，如果想达到这种效果，但又不想让其他人访问到该变量，就可以Local Static；

## 枚举

**枚举量的声明**

- ENUM是enumeration的缩写。基本上它就是一个数值集合。不管怎么说，这里面的**数值只能是整数。**
- 定义枚举类型的主要目的：**增加程序的可读性**
- 枚举变的名字一般以**大写字母**开头（非必需）
- 默认情况下，编译器设置第一个 枚举变量值为 0，下一个为 1，以此类推（也可以手动给每个枚举量赋值），且 **未被初始化的枚举值的值默认将比其前面的枚举值大1。** ）
- 枚举量的值可以相同
- 枚举类型所使用的类型默认为int类型，也可指定其他类型 ，如 unsigned char

例如：

```cpp
enum example {   //声明example为新的数据类型，称为枚举(enumeration);
 Aa, Bb, Cc   //声明Aa, Bb, Cc等为符号常量，通常称之为枚举量，其值默认分别为0，1，2
};


enum example {
 Aa = 1, Bb, Cc = 1，Dd, Ee //1 2 1 2 3 未被初始化的枚举值的值默认将比其前面的枚举值大1。
};

enum example : unsigned char //将类型指定成unsigned char，枚举变量变成了8位整型，减少内存使用。
{
 Aa, Bb = 10, Cc
};

enum example : float  //ERROR！枚举量必须是一个整数，float不是整数（double也不行）。
{
 Aa, Bb = 10, Cc
};
```

要意识到，enum语句示例实际上并没有创建任何变量，只是定义数据类型。当以后创建这个数据类型的变量时，它们看起来就是整数，并且这些整数的值被限制在与枚举集合中的符号名称相关联的整数上。

```cpp
enum example
{
 Aa = 0, Bb, Cc
};
example ex;
```

**枚举量的定义**:

可利用新的枚举类型**example**声明这种类型的变量 example Dd，可以在定义枚举类型时定义枚举变量：

```cpp
enum  example 
{
     Aa, Bb, Cc
}Dd;
```

> 与基本变量类型不同的地方是，**在不进行强制转换的前提下**，只能将定义的**枚举量**赋值给该种枚举的变量(非绝对的，可用强制类型转换将其他类型值赋给**枚举变量**)

```cpp
Dd = Bb; //ok
Dd = Cc; //ok

Dd = 5; //Error!因为5不是枚举量
```

> 枚举量可赋给非枚举变量

```cpp
int a = Aa; //ok.枚举量是符号常量，赋值时编译器会自动把枚举量转换为int类型。
```

> 对于枚举，**只定义了赋值运算符，没有为枚举定义算术运算** ，但**能参与其他类型变量的运算**

```cpp
Aa++;          //非法！
Dd = Aa + Cc   //非法！
int a = 1 + Aa //Ok,编译器会自动把枚举量转换为int类型。
```

> 可以通过**强制转换**将其他类型值赋给枚举变量

```cpp
Dd = example(2);
//等同于
Dd = Cc
//若试图将一个超出枚举取值范围的值通过强制转换赋给枚举变量
Dd = example(10); //结果将是不确定的，这么做不会出错，但得不到想要的结果
```

**枚举的取值范围**:

枚举的上限：大于【最大枚举量】的【最小的2的幂】，减去1；

枚举的下限：

- 枚举量的最小值不小于0，则枚举下限取0；
- 枚举量的最小值小于0，则枚举下限是: 小于【最小枚举量】的【最大的2的幂】，加上1。

例如定义enumType枚举类型：

```cpp
enum enumType {
    First=-5，Second=14，Third=10
};
```

则枚举的上限是16-1=15（16大于最大枚举量 14，且为2的幂）； 枚举的下限是-8+1=-7（-8小于最小枚举量-5，且为2的幂）；

**枚举应用**:

**枚举**和**switch**是最好的搭档：

```cpp
enum enumType{
    Step0, Step1, Step2
}Step=Step0;      //注意这里在声明枚举的时候直接定义了枚举变量Step,并初始化为Step0 

switch (Step)

{

case Step0:{…;break;}

case Step1:{…;break;}

case Step2:{…;break;}

default:break;

}
```

### C++11 的枚举类

C++11 标准中引入了“枚举类”(enum class)。

- 新的enum的作用域不在是全局的
- 不能隐式转换成其他类型

```c++
/**
 * @brief C++11的枚举类
 * 下面等价于enum class Color2:int
 */
enum class Color2
{
    RED=2,
    YELLOW,
    BLUE
};
Color2 c2 = Color2::RED;
cout << static_cast<int>(c2) << endl; //必须转！
```

- 可以指定用特定的类型来存储enum

```c++
enum class Color3:char;  // 前向声明

// 定义
enum class Color3:char 
{
    RED='r',
    BLUE
};
char c3 = static_cast<char>(Color3::RED);
```

## C++的类

默认私有

```cpp
class player {
 int x, y;
 int speed;
 void move(int a, int b){
     x += a * speed;
     y += b * speed;
 }
};
```

访问控制（private、public）

```cpp
class player {
public:
 int x, y;
 int speed;
private:
 void move(int a, int b){
     x += a * speed;
     y += b * speed;
 }
};
```

### 类与结构体对比

区别：

> 作用上：class默认private，struct默认public。
> 使用上：引入struct是为了让C++向后兼容C。

推荐选用：

> 若只包含一些变量结构或POD(plain old data)时，选用struct。例如数学中的向量类。

```cpp
struct Vec2{
 float x, y;
 void Add(const Vec2& other){
     x += other.x;
     y += other.y;
 }
};
```

若要实现很多功能的类，则选用class



### 类和结构体外的静态（static）

static关键字两种用法

- 在类或结构体外部使用static关键字

> **这意味着你定义的函数和变量只对它的声明所在的cpp文件（编译单元）是“可见”的**。换句话说此时static修饰的符号,（在link的时候）它只对定义它的翻译单元(.obj)可见（internal linkage）。

- 在类或结构体内部使用static关键字

> 此时表示这部分内存（static变量）是这个类的所有实例共享的。即：该静态变量在类中创建的所有实例中，静态变量只有一个实例。**一个改变就改变所有。**

类中的静态方法也一样，静态方法中没有该实例的指针（this）。在类中没有实例会传递给该方法。

这一节着重讲在类和结构体外部的静态。

如果不用static定义全局变量，在别的翻译单元可以用extern int a这样的形式，这被称为 external linkage或external linking。

重点是，要让函数和变量标记为静态的，除非你真的需要它们跨翻译单元链接。



**两个全局变量的名字不能一样**

```cpp
//a.cpp中
int s_var = 5;

//main.cpp中
int s_var = 10;
int main(){
    std::cout << s_var << std::endl;
}
```

这样将会linker失败！

```text
error: ld returned 1 exit status
```

解决方法：

> 1.static声明

```cpp
//a.cpp中
static int s_var = 5;

//main.cpp中
int s_var = 10;
int main(){
    std::cout << s_var << std::endl;
}

//输出10
```

> 2.extern声明（将其变成另一个变量的引用）

```cpp
//a.cpp中
int s_var = 5;

//main.cpp中
extern int s_var;   //注意这里没有了赋值
int main(){
    std::cout << s_var << std::endl;
}

//输出5
//这被称为 external linkage或external linking。
```

### 类和结构体中的静态（static）

- **静态方法**不能访问**非静态变量**
- **静态方法没有类实例**
- 本质上你在类里写的每个**非静态方法**都会获得当前的类实例作为参数（this指针）
- 静态成员变量在编译时存储在静态存储区，即**定义过程应该在编译时完成**，因此**一定要在类外进行定义**，但可以不初始化。 **静态成员变量是所有实例共享的**，但是其**只是在类中进行了声明，并未定义或初始化**（分配内存），类或者类实例就无法访问静态成员变量，这显然是不对的，**所以必须先在类外部定义**，也就是分配内存。
  - 因为静态成员属于整个类，而不属于某个对象，如果在类内初始化，会导致每个对象都包含该静态成员，这是矛盾的。

> 在几乎所有面向对象的语言里，static在一个类中意味着特定的东西。**如果是static变量，这意味着在类的所有实例中，这个变量只有一个实例。**比如一个entity类，有很多个entity实例，若其中一个实例更改了这个static变量，它会在所有实例中反映这个变化。这是因为即使创建了很多实例，static的变量仍然只有一个。正因如此，**通过类实例来引用静态变量是没有意义的**。因为这就像类的全局实例。
> 静态方法也是一样，无法访问类的实例。静态方法可以被调用，不需要通过类的实例。而在静态方法内部，你不能写引用到类实例的代码，因为你不能引用到类的实例。

比如一段最简单的测试代码：

```cpp
#include <iostream>
using namespace std;

struct Entity
{
    static int x;

    void print()
    {
        cout << x << endl;
    }
};

int main()
{
    Entity e1;
    e1.x = 1;
    cin.get();
}
```

虽然编译不会报错，但是链接会报错：

```cpp
source.obj : error LNK2001: unresolved external symbol "public: static int Entity::x"
```

于是我们需要给出定义，让链接器可以链接到合适的变量

```cpp
#include <iostream>
using namespace std;

struct Entity
{
    static int x;

    void print()
    {
        cout << x << endl;
    }
};

int Entity::x;

int main()
{
    Entity e1;
    e1.x = 1;
    cin.get();
}
```

此时再创建一个实例对象。

```cpp
#include <iostream>
using namespace std;

struct Entity
{
    static int x;

    void print()
    {
        cout << x << endl;
    }
};

int Entity::x;

int main()
{
    Entity e1;
    e1.x = 1;

    Entity e2;
    e2.x = 2;

    e1.print();
    e2.print();

    cin.get();
}
```

输出结果为两个2，这是因为两个实例化对象共享的是同一个变量，正因如此，**通过类实例来引用静态变量是没有意义的**，最好写为

```cpp
int main()
{
    Entity e1;
    Entity::x = 1;

    Entity e2;
    Entity::x= 2;

    e1.print();
    e2.print();

    cin.get();
}
```

如果包print()函数改为static，仍然正常，**因为它引用的x，y也是静态的变量**，同样的，正确代调用方式：

```cpp
#include <iostream>
using namespace std;

struct Entity
{
    static int x;

    static void print()
    {
        cout << x << endl;
    }
};

int Entity::x;
int main()
{
    Entity e1;
    Entity::x = 1;

    Entity e2;
    Entity::x= 2;

    Entity::print();
    Entity::print();

    cin.get();
}
```

这样，在这个例子里我们都用不到类的实例，因为这些全是静态的，故代码改为：

```cpp
int main()
{
    Entity::x = 1;

    Entity::x= 2;

    Entity::print();
    Entity::print();

    cin.get();
}
```

**但如果把x改为非静态的，则会报错，因为静态方法不能访问非静态变量，原因就是静态方法没有类实例，我们在编写类的时候，本质上我们在类里写的每个非静态方法都会获得当前的类实例作为参数（this指针）。**

因此静态方法和在类外部编写的方法是一样的。

如果在类外面写一个print()函数，则就会报错，这就能为什么说明不能访问到x；

```cpp
#include <iostream>
using namespace std;

struct Entity
{
    int x;

    static void print()
    {
        cout << x << endl;  // 报错，不能访问到非静态变量x
    }
};
//在类外面写一个print()函数
static void print()
{
 cout << x << endl;  // 报错，x是什么？没被定义。
}

int main()
{
    Entity e1;
    e1.x = 1;

    Entity e2;
    e2.x = 2;

    e1.print();
    e2.print();

    cin.get();
}
```

但是如果这样就可以

```cpp
//在类外面写一个print()函数
static void print(Entity e)
{
 cout << e.x << endl;  // 成功运行
}
```

我们刚刚写的方法，**本质上就是一个类的非静态方法在编译时的真正样子**

但如果我把**Entity实例去掉**，就是**把static关键字加到类方法时所做的**

```cpp
//在类外面写一个print()函数
static void print()       //把Entity实例去掉
{
 cout << e.x << endl;  // 报错
}
```

这就是问什么会错，他不知道要怎么访问Entity的x，y因为你没有给他一个Entity的引用。

#### 单例模式

一个例子：有一个单例的类（即:这个类只有一个实例存在）

如果我想不适用静态局部作用域（local static scope）来创建一个单例类,我得创建某种静态的单例实例

如果我用静态局部作用域（local static scope）

> 通过static静态，将生存期延长到永远。这意味着，我们第一次调用get的时候，它实际上会构造一个单例实例，在接下来的时间它只会返回这个已经存在的实例。

```c++
// 在C++11标准下，《Effective C++》提出了一种更优雅的单例模式实现，使用函数内的 local static 对象。
// 这种方法也被称为Meyers' Singleton。


#include <iostream>

using namespace std;

class singleton {
private:
    singleton() {}
public:
    static singleton &instance();
};

singleton &singleton::instance() {
    static singleton p;
    return p;
}

```

- 单线程下，正确。
- C++11及以后的版本（如C++14）的多线程下，正确。
- C++11之前的多线程下，不一定正确。

原因在于在C++11之前的标准中并没有规定local static变量的内存模型。于是乎它就是不是线程安全的了。但是在C++11却是线程安全的，这是因为新的C++标准规定了当一个线程正在初始化一个变量的时候，其他线程必须得等到该初始化完成以后才能访问它。

### 构造函数

- 当创建对象的时候，构造函数被调用
- 构造函数最重要的作用就是初始化类

```cpp
class Entity {
public:
  int x, y;
  Entity(){}  //不带参数
  Entity(int x, int y) : x(x), y(y) {}  //带参数，用来初始化x和y

  void print()
  {
    std::cout << x << ',' << y << std::endl;
  }
};
```

- 构造函数没有返回类型
- **构造函数的命名必须和类名一样**
- 如果你不指定构造函数，你仍然有一个构造函数，这叫做默认构造函数（default constructor），是默认就有的。但是，我们仍然可以删除该默认构造函数：

```cpp
class Log{
public:
    Log() = delete;  //删除默认构造函数
    ......
}
```

- 构造函数不会在你没有实例化对象的时候运行，所以如果你只是使用类的静态方法，构造函数是不会执行的。
- 当你用new关键字创建对象实例的时候也会调用构造函数。

### 析构函数

- 析构函数是在你销毁一个对象的时候运行。
- 析构函数同时适用于栈和堆分配的内存。

> 因此如果你用new关键字创建一个对象（存在于堆上），然后你调用delete，析构函数就会被调用。
> 如果你只有基于栈的对象，当跳出作用于的时候这个对象会被删除，所以这时侯析构函数也会被调用。

- 构造函数和析构函数在声明和定义的唯一区别就是放在析构函数前面的波形符（~）
- 因为这是找分配的，我们会看到当main函数执行完的时候析构函数就会被调用
- **析构函数没有参数，不能被重载**，因此一个类只能有一个析构函数。
- 不显式的定义[析构函数](https://link.zhihu.com/?target=https%3A//so.csdn.net/so/search%3Fq%3D%E6%9E%90%E6%9E%84%E5%87%BD%E6%95%B0%26spm%3D1001.2101.3001.7020)系统会调用默认析构函数

例子：

```cpp
//Student.h
#ifndef PROJECT1_STUDENT_H
#define PROJECT1_STUDENT_H

#include <string>
using namespace std;

class Student {
private:
    int num;
    string name;
    char gender;
public:
    Student();
    Student(int num, string name, char gender);
    ~Student();
    void display();
};

#endif //PROJECT1_STUDENT_H

-------------------------------------------------------------------------------

//Student.cpp:
#include "Student.h"
#include <iostream>
using namespace std;

// 无参构造
Student::Student() : num(-1), name("None"), gender('N') {}

Student::Student(int n, string p, char g) : num(n), name(p), gender(g) {
    cout << "执行构造函数: " << "Welcome, " << name << endl;
}

void Student::display() {
    cout << "num: " << num << endl;
    cout << "name: " << name << endl;
    cout << "gender: " << gender << endl;
    cout << "===============" << endl;
}

Student::~Student() {
    cout << "执行析构函数: " << "Bye bye, " << name << endl;
}

------------------------------------------------------------------------------

//main.cpp
#include "Student.h"
#include <iostream>
using namespace std;

int main() {

    Student student1(1, "Little white", 'f');
    Student student2(2, "Big white", 'f');

    student1.display();
    student2.display();

    return 0;
}

//输出结果
执行构造函数: Welcome, Little white
执行构造函数: Welcome, Big white
num: 1
name: Little white
gender: f
===============
num: 2
name: Big white
gender: f
===============
执行析构函数: Bye bye, Big white
执行析构函数: Bye bye, Little white
```





### C++继承

- 当你创建了一个子类，它会包含父类的一切。
- 继承给我们提供了这样的一种方式：把一系列类的所有通用的代码（功能）放到基类
- 在定义一个新的类 B 时，如果该类与某个已有的类 A 相似（指的是 B 拥有 A 的全部特点），那么就可以把 A 作为一个基类，而把B作为基类的一个派生类（也称子类）。
- 派生类是通过对基类进行修改和扩充得到的，在派生类中，可以扩充新的成员变量和成员函数。
- 派生类拥有基类的全部成员函数和成员变量，不论是private、protected、public。需要注意的是：在派生类的各个成员函数中，不能访问基类的private成员。

**继承的格式**

```cpp
class 派生类名：public 基类名
{
};
```

例子如下，分析：

- 这个Player类不再仅仅只是Player类型，它也是Entity类型，就是说它**同时是这两种类型**。意思是我们可以在**任何想要用Entity的地方使用Player**
- Player总是Entity的一个超集，它拥有Entity的所有内容。
- 因为对于Player来说，在Entity中任何**不是私有**的（private）成员，Player都可以**访问**到

```cpp
//基类
class Entity
{
public:
    float x,y;
 void move(float xa, float ya)
 {
     x += xa;
     y += ya;
 }
};
//派生类
class Player : public Entity
{
public:
 const char* Name;
//  float x,y;    //因为是派生类，所以这些是重复代码，只保留新代码即可
//  void() move(float xa, float ya)
//  {
//      x += xa;
//     y += ya;
// }
 void printName()   //在派生类中，可以扩充新的成员变量和成员函数
 {
     std::cout << Name << std::endl;
 }
};
```

### C++虚函数

- 虚函数可以让我们在子类中重写方法。
- 格式

```cpp
claee 父类名{
   //virtual + 函数
   virtual void GetName(){
       .....
   }
}
```

- 虚函数的例子，通常有三步。

> 第一步，定义基类，声明基类函数为 `virtual` 的。
> 第二步，定义派生类(继承基类)，派生类实现了定义在基类的 `virtual` 函数。
> 第三步，声明基类指针，并指向派生类，调用`virtual`函数，此时虽然是基类指针，但调用的是派生类实现的基类`virtual` 函数。

例子：

```cpp
//基类
class Entity
{
public:
    std::string GetName() {return "Entity";}
};

//派生类
class Player : public Entity
{
private: 
 std::string m_Name; 
public: 
 Player(const std::string& name):m_Name (name) {}  //构造函数
 std::string GetName() {return m_Name;}
};

int main(){
 Entity* e = new Entity();//不用手动删除，因为程序会终止（对象被自动删除）
 std::cout << e->GetName() << std::endl; 
 Player* p = new Player("cherno"); 
 std::cout << p->GetName() << std::endl;
}
---------------------------------
//输出
Entity
cherno
```

上述代码虽能够成功运行，**但是如果使用多态的概念来看，目前为止我们写的这些都是不合格的**

如果我引用这个Player并把它当成Entity类型，就会出现问题，例如：

```cpp
int main(){
 Entity* e = new Entity();//不用手动删除，因为程序会终止（对象被自动删除）
 std::cout << e->GetName() << std::endl; 
 Player* p = new Player("cherno"); 
 std::cout << p->GetName() << std::endl;
 //声明基类指针，并指向派生类
 Entity* entity = p; //p是一个Player类型的指针，它是一个Player，但是我把它指向了一个Entity
 std::cout << entity->GetName() << std::endl; 
}
```

输出：

```cpp
Entity
cherno
Entity //运行代码，你会看见打印出了"Entity"，但是我们希望的是打印Player
```

我们希望的是打印Player,因为虽然我们指向的是个Entity*，但是它实际上是一个Player，它是一个Player类的实例，但实际上运行代码，你会看见打印出了"Entity"，why?

```cpp
Entity* entity = p;
```

举一个更清晰的例子：

```cpp
//基类
class Entity
{
public:
    std::string GetName() {return "Entity";} 
};

//派生类
class Player : public Entity
{
private: 
 std::string m_Name; 
public: 
 Player(const std::string& name):m_Name (name) {}  //构造函数
 std::string GetName() {return m_Name;}
};

void printName(Entity* entity){
 std::cout << entity -> GetName() << std::endl;
}

int main(){
 Entity* e = new Entity();
 printName(e); //我们这儿做的就是调用entity的GetName函数，我们希望这个GetName作用于Entity
 Player* p = new Player("cherno"); 
 printName(p); //printName(Entity* entity)，没有报错是因为Player也是 Entity类型。同样我们希望这个GetName作用于Player
}
```

输出：

```cpp
Entity
Entity
```

两次输出都是Entity，**原因在于如果我们在类中正常声明函数或方法，当调用这个方法的时候，它总是会去调用属于这个类型的方法**，而`void printName(Entity* entity);`参数类型是`Entity*`,意味着它会调用Entity内部的GetName函数，**它只会在Entity的内部寻找和调用GetName**.

但是我们希望C++能意识到,在这里我们传入的其实是一个Player，所以请调用Player的GetName。**此时就需要使用虚函数了。**

- 虚函数引入了一种要**动态分配**的东西，一般通过虚表（vtable）来实现编译。虚表就是一个包含类中所有虚函数映射的列表，通过虚表我们就可以在运行时找到正确的被重写的函数。
- 简单来说，你需要知道的就是**如果你想重写一个函数，你么你必须要把基类中的原函数设置为虚函数**

```cpp
//基类
class Entity
{
public:
    virtual std::string GetName() {return "Entity";} //第一步，定义基类，声明基类函数为 virtual的。
};

//派生类
class Player : public Entity
{
private: 
    std::string m_Name; 
public: 
    Player(const std::string& name):m_Name (name) {} 
    //第二步，定义派生类(继承基类)，派生类实现了定义在基类的 virtual 函数。
    std::string GetName() override {return m_Name;}  //C++11新标准允许给被重写的函数用"override"关键字标记,增强代码可读性。
};

void printName(Entity* entity){
    std::cout << entity -> GetName() << std::endl;
}

int main(){
    Entity* e = new Entity();
    printName(e); 
    //第三步，声明基类指针，并指向派生类，调用`virtual`函数，此时虽然是基类指针，但调用的是派生类实现的基类virtual函数。Entity* p = new Player("cherno");也可以
    Player* p = new Player("cherno"); 
    printName(p); 
}
```

#### 注意：

- 普通函数（非类成员函数）不能是虚函数
- 静态函数（static）不能是虚函数 （static属于class⾃⼰的类相关，必须有实体； static成员没有this指针。virtual函数⼀定要通过对象来调⽤，有隐藏的this指针，实例相关。）
- 构造函数不能是虚函数（因为在调用构造函数时，虚表指针并没有在对象的内存空间中，必须要构造函数调用完成后才会形成虚表指针）
- 析构函数可以是虚函数，⽽且，在⼀个复杂类结构中，这往往是必须的。
- 内联函数不能是表现多态性时的虚函数
- 内联是在编译期建议编译器内联，而虚函数的多态性在运行期，编译器无法知道运行期调用哪个代码，因此虚函数表现为多态性时（运行期）不可以内联。
- 析构函数可以是纯虚的 但纯虚析构函数必须有定义体，因为析构函数的调⽤是在⼦类中隐含的。 
- 当基类指针指向⼦类对象时，如果基类的析构函数不是 virtual，那么⼦类的析构函数将不 会被调⽤，⼦类的资源没有正确是释放，因此造成内存泄露。
- 派⽣类的override虚函数定义必须和⽗类完全⼀致。  除了⼀个特例，如果⽗类中返回值是⼀个指针或引⽤，⼦类override时可以返回这个指针 （或引⽤）的派⽣。

### 虚继承

为什么需要虚继承 

1、为了解决多继承时的命名冲突和冗余数据问题 

C++ 提出了虚继承，使得在派⽣类中只保留⼀份间接基类的成员。其中多继承（Multiple Inheritance）是指从多个直接基类中产⽣派⽣类的能⼒，多继承的派⽣类继承了所有⽗类 的成员。  

2、虚继承的⽬的是让某个类做出声明，承诺愿意共享它的基类



虚继承用于解决多继承条件下的菱形继承问题（浪费存储空间、存在二义性）。

底层实现原理与编译器相关，一般通过**虚基类指针**和**虚基类表**实现，每个虚继承的子类都有一个虚基类指针（占用一个指针的存储空间，4字节）和虚基类表（不占用类对象的存储空间）（需要强调的是，虚基类依旧会在子类里面存在拷贝，只是仅仅最多存在一份而已，并不是不在子类里面了）；当虚继承的子类被当做父类继承时，虚基类指针也会被继承。

实际上，vbptr 指的是虚基类表指针（virtual base table pointer），该指针指向了一个虚基类表（virtual table），虚表中记录了虚基类与本类的偏移地址；通过偏移地址，这样就找到了虚基类成员，而虚继承也不用像普通多继承那样维持着公共基类（虚基类）的两份同样的拷贝，节省了存储空间。

C++标准库中的 iostream 类就是⼀个虚继承的实际应⽤案例。  

iostream 从 istream 和 ostream 直接继承⽽来，⽽ istream 和 ostream 又都继承⾃⼀个共 同的名为 baseios 的类，是典型的菱形继承。  

此时 istream 和 ostream 必须采⽤虚继承，否则将导致 iostream 类中保留两份 baseios 类 的成员。 



使⽤多继承经常出现⼆义性，必须⼗分⼩⼼；  ⼀般只有在⽐较简单和不易出现⼆义性或者实在必要情况下才使⽤多继承，能⽤单⼀继承解 决问题就不要⽤多继承。

#### 虚继承、虚函数

- 相同之处：都利用了虚指针（均占用类的存储空间）和虚表（均不占用类的存储空间）
- 不同之处：
  - 虚继承
    - 虚基类依旧存在继承类中，只占用存储空间
    - 虚基类表存储的是虚基类相对直接继承类的偏移
  - 虚函数
    - 虚函数不占用存储空间
    - 虚函数表存储的是虚函数地址

### C++接口（纯虚函数）

- **纯虚函数优点**

> 防止派生类忘记实现虚函数，**纯虚函数使得派生类必须实现基类的虚函数**。
> 在某些场景下，创建基类对象是不合理的，含有纯虚拟函数的类称为**抽象类**，它**不能直接生成对象。**

- **声明方法**: 在基类中纯虚函数的方法的后面加 **=0**

```cpp
virtual void funtion()=0;
virtual std::string GetName() = 0;
```

- C++中的纯虚函数本质上与其他语言（bi如Java或C#）中的抽象方法或接口相同。
- 纯虚函数与虚函数的区别在于，纯虚函数的基类中的`virtual`函数，只定义了，但不实现。实现交给派生类来做。
- **只能实例化一个实现了所有纯虚函数的类**。**纯虚函数必须被实现**，然后我们才能创建这个类的实例。
- 纯虚函数允许我们在基类中定义一个没有实现的函数，然后**强制子类**去实现该函数。
- 实际上，其他语言有interface关键字而不是叫class，但C++没有。接口只是C++的类而已。
- 在面向对象程序设计中，创建一个只包含未实现方法然后交由子类去实际实现功能的类是非常普遍的,这通常被称为接口。**接口就是一个只包含未实现的方法并作为一个模板的类**。并且由于此**接口类**实际上不包含方法实现，所以我们**无法实例化**这个类。

例子：

```cpp
//基类
class Entity
{
public:
    virtual std::string GetName() = 0; //声明为纯虚函数。请记住，这仍然被定义为虚函数，但是=0实际上将它变成了一个纯虚函数，这意味着如果你想实例化这个类，那么这个函数必须在子类中实现
};

//派生类
class Player : public Entity
{
private: 
 std::string m_Name; 
public: 
 Player(const std::string& name):m_Name (name) {} 
 std::string GetName() override {return m_Name;} //实现纯虚函数
};

void printName(Entity* entity){
 std::cout << entity -> GetName() << std::endl;
}

int main(){
// Entity* e = new Entity();  //会报错，在这里我们必须给它一个实际上实现了这个函数的子类
 Entity* e = new Player("");  //ok
 printName(e); 

 Player* p = new Player("cherno"); 
 printName(p); 
}
```

> 再看一个更好的例子
> 假设我们想编写一个打印这儿所有类名的函数。

```cpp
class Printable{   //接口。其实就是个类。之所以称它为接口，只是因为它有一个纯虚函数，仅此而已。
public:
    virtual std::string GetClassName()= 0;
};
//基类
class Entity : public Printable
{
public:
    virtual std::string GetName() {return "Entity";}
 std::string GetClassName() override {return "Entity";} //实现接口的纯虚函数
};

//派生类
class Player : public Entity //因为Player已经是Entity（已有接口），所以Player不用去实现Printable接口
{
private: 
 std::string m_Name; 
public: 
 Player(const std::string& name):m_Name (name) {} 
 std::string GetName() override {return m_Name;} 
 std::string GetClassName() override {return "Player";} //实现接口的纯虚函数
};

void  Print(Printable* obj){  //我们需要某种类型能提供GetClassName函数，这就是接口。所有的打印都来自于这个Print函数，它接受printable对象，它不关心实际是什么类
    std::cout <<obj ->GetClassName() << std::endl;
}

int main(){
 Entity* e = new Entity();
 Player* p = new Player("cherno"); 
 Print(e);
 Print(p);
}
//输出：
Entity
Player
```

上例中，如果Player不是Entity的话，就要添加接口，如下

```cpp
class Player : public OtherClass,Printable  //加逗号，添加接口Printable
{
 ....
 std::string GetName() override {return m_Name;} 
};
```

### C++可见性

- 可见性是一个属于面向对象编程的概念，它指的是类的某些成员或方法实际上是否可见。可见性是指：谁能看到它们，谁能调用它们，谁能使用它们，所有这些东西。
- 可见性是对程序实际运行方式、程序性能或类似的东西没影响。它只单纯的是语言层面的概念，让你能够写出更好的代码或者帮助你组织代码。
- C++中有三个基础的可见修饰符（访问修饰符）：**private、protected、public**

> **private**：只有**自己的类和它的友元**才能访问（继承的子类也不行，友元的意思就是可以允许你访问这个类的私有成员）。
> **protected**：这个**类以及它的所有派生类**都可以访问到这些成员。（但在main函数中new一个类就不可见，这其实是因为main**函数不是类的函数**，对main函数是不可访问的）
> **public：**谁都可见。

- 可见性是让代码更加容易维护，容易理解，不管是阅读代码还是扩展代码。这与性能无关，也不会产生完全不同的代码。可见性不是CPU需要理解的东西，这不是你的电脑需要知道的东西。它只是人类发明的东西，为了帮助其他人和自己。

## C++数组

- C++数组就是表示一堆的变量组成的集合，一般是一行相同给类型的变量。
- 内存访问违规（Memory access violation）：在debug模式下，你会得到一个程序崩溃的错误消息，来帮助你调试那些问题；然而在release模式下，你可能不会得到报错信息。这意味着你已经写入了不属于你的内存。

```cpp
//定义一个含5个整数的数组  
    int a[5];
    //访问
    a[5],a[-1]; //内存访问违规（Memory access violation）
```

- 循环的时候涉及到性能问题，我们一般是小于比较而不是小于等于（少一次等于的判断）。

```cpp
int main()
{
    int example[5];
    int* ptr = example;
    for (int i = 0; i< 5;i++) //5个元素全部设置为2
        example[i] = 2; 

    example[2] = 5; //第三个元素设置为5
    *(ptr + 2) = 6; //第三个元素设置为6。因为它会根据数据类型来计算实际的字节数，所以在这里因为这个指针是整形指针所以会是加上2乘以4，因为每个整形是4字节
    *(int*)((char*)ptr + 8) = 7; //第三个元素设置为5。因为每个char只占一个字节
    std::cin.get();
}
```

### 栈数组和堆数组

- 不能把栈上分配的数组（字符串）作为返回值，**除非**你传入的参数是一个内存地址。
- 如果你想返回的是在函数内新创建的数组，那你就要用new关键字来创建。
- 栈数组`int example[5];` 堆数组`int* another = new int[5];`

```cpp
int main()
{
 int example[5]; //这个是创建在栈上的，它会在跳出这个作用域时被销毁
 for (int i = 0; i< 5;i++) //5个元素全部设置为2
    example[i] = 2; 
    int* another = new int[5];//这行代码和之前的是同一个意思，但是它们的生存期是不同的.因为这个是创建在堆上的,实际上它会一直存活到直到我们把它销毁或者程序结束。所以你需要用delete关键字来删除它。
 for (int i = 0; i< 5;i++) //5个元素全部设置为2
    another[i] = 2; 
 delete[] another;
 std::cin.get();
}
```

> 上述的两个数组在内存上看都是一样的，元素都是5个2；
> 那为什么要使用new关键字来动态分配，而不是在栈上创建它们呢？**最大的原因是因为生存期**,因为new分配的内存，会一直存在，直到你手动删除它。
> 如果你有个函数要**返回新创建的数组**，那么你**必须要使用new来分配**，**除非**你传入的参数是一个内存地址。

### memory indirection（内存间接寻址）

> 意思是，有个指针，指针指向另一个保存着我们实际数组的内存块（p-> p-> array），这会产生一些内存碎片和缓存丢失。

```cpp
class Entity
{
public:
    int example[5]; //栈数组
    Entity()  //创建一个构造函数，用来初始化所有值为2
    {
        for (int i = 0; i< 5;i++) 
             example[i] = 2;   
    }
};

int main()
{
    Entity e; //实例化一个对象。如果我们查看Entity e的内存地址，可以看到Entity的内存上实际就是一行，包含了数组中所有的2，所有的数据都在这儿

    std::cin.get();
}
```

> 但是如果我在这里使用new关键字在堆上创建，运行相同的代码
> 这时我们查看Entity e的内存地址，可以看到这里的内存根本没有2。我看到另一个内存地址，其实这就是个指针。我可以复制该地址然后粘贴查找（因为endian（字节序）的原因我必须要反转它们），然后就可以看到我真正的数据。这就是**memory indirection（内存间接寻址）**。

```cpp
class Entity
{
public:
    int* example = new int[5];  //堆数组
    Entity()  
    {
        for (int i = 0; i< 5;i++) 
             example[i] = 2;   
    }
};

int main()
{
    Entity e; //这时我们查看Entity e的内存地址，可以看到这里的内存根本没有2。我看到另一个内存地址，其实这就是个指针。我可以复制该地址然后粘贴查找（因为endian（字节序）的原因我必须要反转它们），然后就可以看到我真正的数据。这就是memory indirection（内存间接寻址）

    std::cin.get();
}
```

我们在e地址上实际上得到的另一个地址，这个地址指向我们实际的数组。这意味着如果我们要访问这个数组，我们基本上要在代码里跳来跳去，先找到Entity，然后再找到数组。

如果可以你应该在**栈**上**创建数组来避免这种情况**。因为像这样在内存里跳跃肯定会影响性能

### C++11中的std:array

> 这是一个内置数据结构，定义在C++11的标准库中。很多人喜欢用它来代替这里的原生数组，因为他有很多优点，它有边界检查，有记录数组的大小
> 实际上我们没有办法计算原生数组的大小，但可以通过一些办法知道大小（例如因为当你删除这个数组时，编译器要知道实际上需要释放多少内存）。

**计算原生数组大小**

> 方法一：通过依赖编译器，它有可能会在数组中存储一个负索引（-1），但应该永远都不要在数组内存中访问数组的大小，这很危险。

创建一个**栈数组**,你不知道他的实际大小，因为它是在栈上分配的，也就是说这是**栈上的地址加上偏移量**

```cpp
int a[5]; //栈数组
```

所以如果你写:

```cpp
sizeof(a); //20bytes
```

如果你想知道有多少个元素，可以用sizeof(a)除以数据类型int的大小，得到5

```cpp
int count = sizeof(a) / sizeof(int); //5
```

但是如果你用**堆数组**example做同样的事：

```cpp
int* example = new int[5];  //堆数组
int count = sizeof(example) / sizeof(int); //1
```

你再这里得到的实际上是一个整形指针的大小（int* example），就是4字节，4/4就是1。

所以**只能在栈分配的数组上用这个技巧**，但是你真的**不能相信这个方法**！当你把它放在函数或者它变成了指针，那你完蛋了（因为**“栈上的地址加上偏移量”**）。所以你要做的就是自己维护数组的大小。

如何维护呢？方法有两个

方法一：

```cpp
class Entity
{
public:
 static constexpr const int size = 5;//在栈中为数组申请内存时,它必须是一个编译时就需要知道的常量。constexpr可省略，但类中的常量表达式必须时静态的
 int example[size];  //此时为栈数组,
 Entity()  
 {
     for (int i = 0; i<size;i++) 
          example[i] = 2;   
 }
};
```

方法二：std:array

```cpp
include <array> //添加头文件
class Entity
{
public:
    std::array<int,5> another; //使用std::array
 Entity()  
 {
     for (int i = 0; i< another.size();i++) //调用another.size()
          example[i] = 2;   
 }
};
```

这个方法会安全一些。



## C++字符串

**字符串/字符数组的工作原理**

> 1.字符串实际上是字符数组
> 2.C++中有一种数据类型叫做char，是Character的缩写，这是一个字节的内存。它很有用因为它能把指针转换为char型指针，所以你可以用字节来做指针运算。它对于分配内存缓冲区也很有用，比如分配1024个char即1KB空间。它对字符串和文本也很有用，因为C++对待字符的默认方式是通过Ascii字符进行文本编码。我们在C++中处理字符是一个字符一个字节。Ascii可以扩展比如UTF-8、UTF-16、UTF-32，我们有wide string（宽字符串）等。我们有两个字节的字符、三个字节、四个字节的等等。
> 3.C++中默认的双引号就是一个字符数组**const char\*，并且\**末尾会补’\0’ （**空终止符**）， 而**cout会输出直到’\0’就终止。**

```cpp
const char* name = "cherno"; //c风格字符串
char* name = "cherno" //报错，因为C++中默认的双引号就是一个字符数组const char*
char name[3] = {'l','i','u'};//报错，缺少空终止符
char name[3] = {'l','i','u'，0};//正确
char name[3] = {'l','i','u'，'\0'};//正确,因为ascii码'\0'就是null
```

**C++字符串（std::string）的使用**

- C++标准库里有个类叫string，实际上还有一个模板类basic_string。**std::string** 本质上就是这个basic_string的char作为模板参数的**模板类实例**。叫**模板特化**（template specialization），就是把char作为模板类basic string的模板参数，意味着char就是每个字符背后的的数据类型。
- 在C++中使用字符串时你应该使用std::string。
- string有个接受参数为**char指针或者const char指针**的**构造函数**。在C++中用双引号来定义字符串一个或者多个单词时，它其实就是**const char数组**，而不是char数组。
- std::string本质上它就是一个char数组，一个char的数组和一些内置函数

使用：

```cpp
#include <iostream>
  #include <string>

  int main()
  {
    std::string name = "Cherno";
   mname.size();
    std::cout << name << std::endl;

    std::cin.get();
  }
```

追加字符串

```cpp
  std::string name = "Cherno" + "hello!";//ERROR!
```

原因是在将两个const char数组相加，因为，**双引号里包含的内容是const char数组，它不是真正的string**；它不是字符串，你不能将两个指针或者两个数组加在一起，它不是这么工作的。

所以如果你想这么做，要么就是把它们分成多行，然后name+="hello！"

```cpp
  std::string name = "Cherno"";
  name += "hello!  //OK
```

这样做是在将一个指针加到了字符串name上了，然后+=这个操作符在string类中被重载了，所以可以支持这么操作。

或者经常做的是**显式地调用string构造函数**将**其中一个传入string构造函数中**,相当于你在创建一个字符串，然后附加这个给他。

```cpp
  std::string name = std::string("Cherno") + "hello!";//OK
  bool contains = name.find("no") != std::string::npos;//用find去判断是否包含字符“no”
```

## C++字符串字面量

- 字符串字面量就是双引号中的内容。
- 字符串字面量是存储在**内存**的**只读部分**的，不可对只读内存进行写操作。
- C++11以后，默认为`const char*`,否则会报错。

```cpp
char* name = "cherno";//Error!
  name[2] = 'a'; //ERROR! 未定义行为；是因为你实际上是在用一个指针指向那个字符串字面量的内存位置，
                 //但字符串字面量是存储在内存的只读部分的，而你正在试图对只读内存进行写操作
  --------------------------------------
  const char* name = "cherno"; //Ok!
  name[2] = 'a'; //ERROR!const不可修改
  //如果你真的想要修改这个字符串，你只需要把类型定义为一个数组而不是指针
  char name[] = "cherno"; //Ok!
  name[2] = 'a'; //ok
```

- 从C++11开始，有些编译器比如Clang，实际上只允许你编译`const char*`, 如果你想从一个字符串字面量编译char,你必须手动将他转换成`char*`

```cpp
char* name = (char*)"cherno"; //Ok!
name[2] = 'a'; //OK
```

- 别的一些字符串

> 基本上，char是一个字节的字符，char16_t是两个字节的16个比特的字符（utf16），char32_t是32比特4字节的字符（utf32），const char就是utf8. 那么wchar_t也是两个字节，和char16_t的区别是什么呢？事实上宽字符的大小，实际上是由编译器决定的，可能是一个字节也可能是两个字节也可能是4个字节，实际应用中通常不是2个就是4个（Windows是2个字节，Linux是4个字节），所以这是一个变动的值。如果要两个字节就用char16_t，它总是16个比特的。

```cpp
   const char* name = "lk";
   const wchar_t* name2 = L"lk";
   const char16_t* name3 = u"lk";
   const char32_t* name4 = U"lk";
   const char* name5 = u8"lk";
```

**string_literals**

```cpp
#include <iostream>
#include <string>

int main()
{
    using namespace std::string_literals;

    std::string name0 = "hbh"s + " hello";

    std::cin.get();
}
```

string_literals中定义了很多方便的东西，这里字符串字面量末尾加s，可以看到实际上是一个操作符函数，它返回标准字符串对象（std::string）

然后我们就还能方便地这样写等等：

```cpp
std::wstring name0 = L"hbh"s + L" hello";
```

string_literals也可以忽略转义字符

```cpp
#include <iostream>
#include <string>

int main()
{
    using namespace std::string_literals;

    const char* example =R"(line1
    line2
    line3
    line4)"

    std::cin.get();
}
```

## C++中CONST

### 作用

- 类型检查

  - const常量与`#define`宏定义常量的区别：

  > ~~**const常量具有类型，编译器可以进行安全检查；#define宏定义没有数据类型，只是简单的字符串替换，不能进行安全检查。**~~此处的错误在于 `#define` 定义的宏常量（假设它不是花括号初始化器列表）同样存在类型。
  > 例如若写 `#define FOURTY_TWO 42` ，则 `FOURTY_TWO` 的类型是 `int` 。具体的类型和各种字面量（整数、浮点、用户定义等）和运算符的结果类型有关。

  - const定义的变量只有类型为整数或枚举，且以常量表达式初始化时才能作为常量表达式。其他情况下它只是一个 `const` 限定的变量，不要将与常量混淆。

- 防止修改，起保护作用，增加程序健壮性

- 可以节省空间，避免不必要的内存分配

  - const定义常量从汇编的角度来看，只是给出了对应的内存地址，而不是像`#define`一样给出的是立即数。
  - const定义的常量在程序运行过程中只有一份拷贝，而`#define`定义的常量在内存中有若干个拷贝。

  > 此处的错误在于，若用 `const` 定义常量（类型为整数或枚举，必须以常量表达式初始化），则这种常量在非 odr 式使用（粗略来说是只使用其值）时不需要依赖其身为变量的身份，一定场合下甚至可以不需要定义（譬如作为类的 `static` 成员对象）。
  > 编译器在作为常量处理它时，不会依赖“一份定义”，而是像是立即数一样使用它，它本身可能在机器码中被“拷贝”到多个地方，和 `#define` 定义的宏常量的结果相同。
  > 另一方面， `const` 定义的常量由于是整数或枚举，所以直接变成机器码上的立即数往往性能更好。



### 用法

- **const首先作用于左边的东西；如果左边没东西，就做用于右边的东西**
- const被cherno称为伪关键字，因为它在改变生成代码方面做不了什么。
- const是一个承诺，承诺一些东西是不变的，你是否遵守诺言取决于你自己。我们要保持const是因为这个承诺实际上可以简化很多代码。
- 绕开const的方法：**(但不建议这么做)**

> **强制转换**

```cpp
  const int Age =90;
  int* a = new int;
  *a = 2;
  a = &Age //error!
  a =(int*)&Age //ok
```

#### 函数参数

**参数为引用，为了增加效率同时防止修改。**

```cpp
void func(const A &a)
```

对于非内部数据类型的参数而言，像void func(A a) 这样声明的函数注定效率比较低。因为函数体内将产生A 类型的临时对象用于复制参数a，而临时对象的构造、复制、析构过程都将消耗时间。

为了提高效率，可以将函数声明改为void func(A &a)，因为“引用传递”仅借用一下参数的别名而已，不需要产生临 时对象。

> 但是函数void func(A &a) 存在一个缺点：
>
> “引用传递”有可能改变参数a，这是我们不期望的。解决这个问题很容易，加const修饰即可，因此函数最终成为 void func(const A &a)。

以此类推，是否应将void func(int x) 改写为void func(const int &x)，以便提高效率？完全没有必要，因为内部数据类型的参数不存在构造、析构的过程，而复制也非常快，“值传递”和“引用传递”的效率几乎相当。

> 小结：
> 1.**对于非内部数据类型的输入参数，应该将“值传递”的方式改为“const 引用传递”，目的是提高效率。例如将void func(A a) 改为void func(const A &a)。**
>
> 2.**对于内部数据类型的输入参数，不要将“值传递”的方式改为“const 引用传递”。否则既达不到提高效率的目的，又降低了函数的可理解性。例如void func(int x) 不应该改为void func(const int &x)。**

#### **const指针的用法**

> 适用于指针和指针指向的内容

```cpp
int* a = new int;
*a = 2;
a =(int*)&Age //ok
```

上述代码我们可以做两件事，一可以**改变这个指针的内容**，就是指针指向的内存内容；二可以**改变指针指向的内存地址**。

> **const int**（同int const）
> **可以改变指针指向的地址,不能再去修改指针指向的内容了**

```cpp
const int* a = new int;
*a = 2; //error! 不能再去修改指针指向的内容了。
a =(int*)&Age  //可以改变指针指向的地址
```

> int* const
> **可以改变指针指向的内容,不能再去修改指针指向的地址**

```text
int* const a = new int;
*a = 2; //ok
a =(int*)&Age  //error
```

> const int* const
> **既不可以改变指针指向的内容,也不能再去修改指针指向的地址**



#### **在类和方法中的const**

> const的第三种用法，他和变量没有关系，而是用在方法名的后面( **只有类才有这样的写法** )
> 这意味这这个方法不会修改任何实际的类，因此这里你可以看到我们不能修改类的成员变量

```cpp
class Entity
{
private:
    int m_x,m_y;
public:
    int Getx() const  //const的第三种用法，他和变量没有关系，而是用在方法名的后面
    {
        return m_x; //不能修改类的成员变量
        m_x = 2; //ERROR!
    }
    void Setx(int a)
    {
        m_x = a; //ok
    }
};

void PrintEntity(const Entity& e)  //const Entity调用const函数
{
    std::cout << e.Getx() << std::endl;
}

int main()
{
    Entity e;
}
```

然后有时我们就会写两个Getx版本，一个有const一个没有，然后上面面这个传const Enity&的方法就会调用const的GetX版本。

所以，我们把成员方法标记为const是因为**如果我们真的有一些const Entity对象，我们可以调用const方法**。如果没有const方法，那const Entity&对象就掉用不了该方法。

- 如果实际上没有修改类或者它们不应该修改类，**总是**标记你的方法为const，否则在有常量引用或类似的情况下就用不了你的方法。
- 在**const函数中**， 如果要修改别的变量，可以用**关键字mutable**：

把类成员标记为mutable，意味着类中的const方法可以修改这个成员。

```cpp
class Entity
  {
  private:
      int m_x,m_y;
      mutable var;
  public:
      int Getx() const 
      {   
          var = 2; //ok mutable var
          return m_x; //不能修改类的成员变量
          m_x = 2; //ERROR!
      }
  };
```

### 注意

#### const对象默认为文件局部变量

注意：非const变量默认为extern。要使const变量能够在其他文件中访问，必须在文件中显式地指定它为extern。

> 未被const修饰的变量在不同文件的访问

```cpp
// file1.cpp
int ext;
// file2.cpp
#include<iostream>

extern int ext;
int main(){
    std::cout<<(ext+10)<<std::endl;
}
```

> const常量在不同文件的访问

```cpp
//extern_file1.cpp
extern const int ext=12;
//extern_file2.cpp
#include<iostream>
extern const int ext;
int main(){
    std::cout<<ext<<std::endl;
}
```

> 小结：
> 可以发现未被const修饰的变量不需要extern显式声明！而const常量需要显式声明extern，并且需要做初始化！因为常量在定义后就不能被修改，所以定义时必须初始化。



## C++constexpr

关键字 **`constexpr`** 是在 C++11 中引入的，并在 C++14 中进行了改进。 它表示 constant（常数）表达式。 与 **`const`** 一样，它可以应用于变量：如果任何代码试图 modify（修改）该值，将引发编译器错误。 与 **`const`** 不同，**`constexpr`** 也可以应用于函数和类 constructor（构造函数）。 **`constexpr`** 指示值或返回值是 constant（常数），如果可能，将在编译时进行计算。

每当需要 const 整数时（如在模板自变量和数组声明中），都可以使用 **`constexpr`** 整数值。 如果在编译时（而非运行时）计算某个值，它可以使程序运行速度更快、占用内存更少。

### const与constexpr区别

**`const`** 与 **`constexpr`** 变量之间的主要 difference（区别）是，**`const`** 变量的初始化可以推迟到运行时进行。 **`constexpr`** 变量必须在编译时进行初始化。 所有的 **`constexpr`** 变量都是 **`const`**。

- 如果一个变量具有文本类型并且已初始化，则可以使用 **`constexpr`** 声明该变量。 如果初始化是由 constructor（构造函数）performed（执行）的，则必须将 constructor（构造函数）声明为 **`constexpr`**。
- 当满足以下两个条件时，引用可以被声明为 **`constexpr`**：引用的对象是由 constant（常数）常数表达式初始化的，初始化期间调用的任何隐式转换也是 constant（常数）表达式。
- **`constexpr`** 变量或函数的所有声明都必须具有 **`constexpr`** specifier（说明符）。

```cpp
constexpr float x = 42.0;
constexpr float y{108};
constexpr float z = exp(5, 3);
constexpr int i; // Error! Not initialized
int j = 0;
constexpr int k = j + 1; //Error! j not a constant expression
```



```c++
// 对于修饰Object来说:
// const并未区分出编译期常量和运行期常量
// constexpr限定在了编译期常量
// constexpr作用：优化!效率!
// constexpr修饰的函数，返回值不一定是编译期常量。It is not a bug, it is a feature.
// const修饰的是类型，constexpr修饰的是用来算出值的那段代码。
/**
 * constexpr修饰的函数，简单的来说，如果其传入的参数可以在编译时期计算出来，那么这个函数就会产生编译时期的值。
 * 但是，传入的参数如果不能在编译时期计算出来，那么constexpr修饰的函数就和普通函数一样了。
 * @param i
 * @return
 */
constexpr int foo(int i)
{
    return i+10;
}

int main()
{
    int i = 10;
    std::array<int, foo(2)> arr; // OK

    foo(i); // Call is Ok

    std::array<int, foo(i)> arr1; // error: the value of ‘i’ is not usable in a constant expression

}

```



## C++mutable关键字

mutable有两种不同的用途：

> 1.与const一起用(最主要的用法，见上一篇)
> 2.lambda表达式，或者同时包含这两种情况

```cpp
//引用传递，没什么问题
#include <iostream>
int main()
{
    int x = 8;  
    auto f = [&]()
    {
        x++;  //如果时值传递，则会报错。
        std::cout << y << std::endl;
    };
    f();
}
-----------------------------------------------
//值传递的正确做法
#include <iostream>
int main()
{
    int x = 8;
    auto f = [=]()
    {
        int y = x;  //有些臃肿
        y++;
        std::cout << y << std::endl;
    };
    f();
}
```

添加mutable关键字,会干净许多（但本质是一样的）

```cpp
#include <iostream>
int main()
{
    int x = 8;
    auto f = [=]() mutable
    {
        x++;
        std::cout << x << std::endl;
    };
    f();
}
```

## C++的成员初始化列表

- 注意：在成员初始化列表里需要按成员变量定义的顺序写。这很重要，因为不管你怎么写初始化列表，它都会按照定义类的顺序进行初始化。
- 使用成员初始化列表的原因：

> 代码风格简洁 避免性能浪费

有两种方法可以**在构造函数中初始化类成员**

方法一：

```cpp
class Entity
  {
  private:
    std::string m_Name;
  public:
    Entity()   //默认构造函数
    {    
        m_Name = "Unknow";
    }
    Entity(const std::string& name)
    {
        m_Name = name;
    }
  };
```

方法二：初始化成员列表

```cpp
  #include <iostream>
  
  class Entity
  {
  private:
  	std::string m_Name;
  	int m_Score;
  public:
  	Entity() : m_Name("Unknow"), m_Score(0)
  	{
  	}
  	Entity(const std::string& name,int n) :m_Name(name), m_Score(100)
  	{
  	}
  	const std::string& GetName() const { return m_Name; };
  	const int& GetScore() const { return m_Score; };
  };
  int main()
  {
  	Entity e0;
  	Entity e1("lk",50);
  	std::cout << e0.GetName() <<e0.GetScore() << std::endl;
  	std::cout << e1.GetName() <<e1.GetScore()<<std::endl;
  }
```

## C++三元运算符

格式:

```cpp
条件表达式 ? 表达式1 : 表达式2;
语义：如果“条件表达式”为true，则整个表达式的值就是表达式1，忽略表达式2；如果“条件表达式”为false，则整个表达式的值就是表达式2,等价于if/else语句。
```

> 实际上只是if的语法糖。

作用: - 代码更简洁 - 速度更快一点

- 尽量不对三元操作符进行嵌套

## 创建并初始化C++对象

- 基本上，当我们编写了一个类并且到了我们实际开始使用该类的时候，就需要实例化它(除非它是完全静态的类)
- 实例化类有两种选择，这两种选择的区别是内存来自哪里，我们的对象实际上会创建在哪里。
- 应用程序会把内存分为两个主要部分：堆和栈。还有其他部分，比如源代码部分，此时它是机器码。

### 栈分配

格式:

```cpp
// 栈中创建
Entity entity;
Entity entity("lk");
```

- 什么时候栈分配？几乎任何时候，因为在C++中这是初始化对象最快的方式和最受管控的方式。
- 什么时候不栈分配？ 如果创建的**对象太大**，或是需要显示地控制对象的**生存期**，那就需要堆上创建 。

### 堆分配

格式:

```cpp
// 堆中创建
Entity* entity = new Entity("lk");
delete entity； //清除
```

- 当我们调用new Entity时，实际发生的就是我们在堆上分配了内存，我们调用了构造函数，然后这个new Entity实际上会返回一个Entity指针，它返回了这个entity在堆上被分配的内存地址，这就是为什么我们要声明成Entity*类型。
- 如果你使用了new关键字，那你就要用delete来进行清理。

#### C++ new关键字

- new的主要目的是分配内存，具体来说就是在堆上分配内存。
- 如果你用new和**[]**来分配数组，那么也用**delete[]**。
- new主要就是找到一个满足我们需求的足够大的内存块，然后**返回一个指向那个内存地址的指针**。

```cpp
int* a = new int; //这就是一个在堆上分配的4字节的整数,这个b存储的就是他的内存地址.
  int* b = new int[50];//在堆上需要200字节的内存。
  delete a;
  delete[] b;
  //在堆上分配Entity类
  Entity* e = new Entity();
  Entity* e = new Entity;//或者这我们不需要使用括号，因为他有默认构造函数。
  Entity* e0 = new Entity[50]; //如果我们想要一个Entity数组，我们可以这样加上方括号,在这个数组里，你会在内存中得到50个连续的Entity
  delete e;
  delete[] e0;
```

- 在new类时，该关键字做了两件事

> 分配内存 调用构造函数

```cpp
Entity* e = new Entity();//1.分配内存 2.调用构造函数
Entity* e = (Entity*)malloc(sizeof(Entity);//仅仅只是分配内存**然后给我们一个指向那个内存的指针
//这两行代码之间仅有的区别就是第一行代码new调用了Entity的构造函数
delete e;//new了，必须要手动清除
```

- new 是一个**操作符**，就像加、减、等于一样。它是一个操作符，这意味着你可以重载这个操作符，并改变它的行为。
- 通常调用new会调用隐藏在里面的C函数malloc，但是**malloc仅仅只是分配内存**然后给我们一个指向那个内存的指针，而**new不但分配内存，还会调用构造函数**。同样，**delete则会调用destructor析构函数。**
- new支持一种叫**placement new**的用法，这决定了他的内存来自哪里, 所以你并没有真正的分配内存。在这种情况下，你只需要调用构造函数，并在一个特定的内存地址中初始化你的Entity，可以通过些new()然后指定内存地址，例如：

```cpp
int* b = new int[50]; 
Entity* entity = new(b) Entity();
```

## C++隐式转换与explicit关键字

**隐式转换**

> 隐式转换**只能进行一次**。

```cpp
#include <iostream>

class Entity
{
private:
    std::string m_Name;
    int m_Age;
public:
    Entity(const std::string& name)
        : m_Name(name), m_Age(-1) {}

    Entity(int age)
        : m_Name("Unknown"), m_Age(age) {}
};

int main()
{
    Entity test1("lk");
    Entity test2(23); 
    Entity test3 = "lk"; //error!只能进行一次隐式转换
    Entity test4 = std::string("lk");
    Entity test5 = 23; //发生隐式转换


    std::cin.get();
}
```

如上，在test5中，int型的23就被隐式转换为一个Entity对象，这是**因为Entity类中有一个Entity(int age)构造函数，因此可以调用这个构造函数，然后把23作为他的唯一参数，就可以创建一个Entity对象。**

同时我们也能看到，对于语句`Entity test3 = "lk";`会报错，原因是**只能进行一次隐式转换**，`"lk"`是`const char`数组，这里需要先转换为`std::string`，再从string转换为Entity变量，两次隐式转换是不行的，所以会报错。但是写为`Entity test4 = std::string("lk");`就可以进行隐式转换。

最好不写`Entity test5 = 23;`这样的函数，应尽量避免隐式转换。因为`Entity test2(23);`更清晰。



**explicit 关键字**

- explicit是用来当你想要显示地调用构造函数，而不是让C++编译器隐式地把任何整形转换成Entity
- 我有时会在数学运算库的地方用到explicit，因为我不想把数字和向量来比较。一般explicit很少用到。
- 如果你在构造函数前面加上explicit，这意味着这个构造函数不会进行隐式转换
- 如果你想用一个整数构造一个Entity对象，那你就必须显示的调用这个构造函数，**explicit会禁用隐式转换**，explicit关键字放在构造函数前面

```cpp
#include <iostream>
  class Entity
  {
  private:
    std::string m_Name;
    int m_Age;
  public:
    Entity(const std::string& name)
        : m_Name(name), m_Age(-1) {}

    explicit Entity(int age)  //声明为explicit
        : m_Name("Unknown"), m_Age(age) {}
  };

  int main()
  {
    Entity test1("lk");
    Entity test2(23); 
      Entity test3 = "lk"; 
    Entity test4 = std::string("lk");
    Entity test5 = 23; //error！禁用隐式转换


    std::cin.get();
  }
```

加了explicit后还想隐式转换，则可以：

```cpp
Entity test5 = (Entity)23; //ok
```

## C++运算符(操作符)及其重载

- **操作符就是函数。**
- 运算符是给我们使用的一种符号，通常代替一个函数来执行一些事情。比如加减乘除、dereference运算符、箭头运算符、+=运算符、&运算符、左移运算符、new和delete、逗号、圆括号、方括号等等 。
- 运算符重载允许你在程序中定义或者更改一个操作符的行为。
- 应该相当少地使用操作符重载，只在他非常有意义的时候使用。

### “+”和“*”操作符重载

重载和无重载的区别：

> **无重载**（例如JAVA里就没有操作符重载）

```cpp
#include <iostream>
struct Vector2
{
    float x, y;
    Vector2(float x,float y) 
        :x(x),y(y){}
    Vector2 Add(const Vector2& other) const
    {
        return Vector2(x + other.x, y + other.y);
    }
    Vector2 Multiply(const Vector2& other) const
    {
        return Vector2(x * other.x, y * other.y);
    }

};

int main()
{
    Vector2 position(4.0f, 4.0f);
    Vector2 speed(0.5f, 1.5f);
    Vector2 powerup(1.1f, 1.1f); //改变speed
    Vector2 result1 = position.Add(speed.Multiply(powerup)); //无重载方式
    std::cin.get();
}
```

> **使用重载** 需要要【**定义操作符**】比如上述代码中的`“+”`和`*`要定义重载操作

```cpp
#include <iostream>
struct Vector2
{
    float x, y;
    Vector2(float x,float y) 
        :x(x),y(y){}
    Vector2 Add(const Vector2& other) const
    {
        return Vector2(x + other.x, y + other.y);
    }

    Vector2 operator+(const Vector2& other) const  //定义+操作符
    {
        return Add(other);
    }

    Vector2 Multiply(const Vector2& other) const
    {
        return Vector2(x * other.x, y * other.y);
    }
    Vector2 operator*(const Vector2& other) const  //定义*操作符
    {
        return Multiply(other);
    }

};

int main()
{
    Vector2 position(4.0f, 4.0f);
    Vector2 speed(0.5f, 1.5f);
    Vector2 powerup(1.1f, 1.1f); //改变speed
    Vector2 result1 = position.Add(speed.Multiply(powerup)); //无重载方式
    Vector2 result2 = position + speed * powerup; //重载方式

    std::cin.get();
}
```

result2的写法比rusult1看起来好的多。

### 左移操作符的重载

左移操作符的重载

> 如上，现在我们有了这个Vector2，然后我们想要把它打印到控制台

```cpp
Vector2 result2 = position + speed * powerup; //重载方式
std::cout << result2 << std::endl; //C++ 没有与这些操作数匹配的运算符 操作数类型为:  std::ostream << Vector2
```

> 报错的原因在于**"<<"操作符还没有被重载**，他接受两个参数，一个是输出流，也就是cout，然后另一个就是Vector2 （**操作数类型为: std::ostream << Vector2** ）
> 我们可以**在Vector2类外面**对它进行重载，因为她其实和Vector2其实没什么关系

```cpp
#include <iostream>
struct Vector2
{
      ......

};
//定义左移操作符的重载
std::ostream& operator<<(std::ostream& stream, const Vector2& other)
{
    stream << other.x << "," << other.y;  //这里的stream已经知道要如何打印浮点数.所以我们不需要再对浮点数进行重载
    return stream;
}

int main()
{
    .....
    Vector2 result2 = position + speed * powerup; 
    std::cout << result2 << std::endl; //需要重载<<
    std::cin.get();
}
```

### bool操作符的重载

> == 和 =!

```cpp
#include <iostream>
struct Vector2
{
    ....
    bool operator==(const Vector2& other) const  //定义操作符的重载,如果！=，这里做相应修改即可
    {
        return x == other.x && y == other.y;
    }

    bool operator！=(const Vector2& other) const  //如果！=，这里做相应修改即可
    {
        return ！(*this == other);
    }
};

    ....

int main()
{
    ....
    Vector2 result1 = position.Add(speed.Multiply(powerup));
    Vector2 result2 = position + speed * powerup; 

    if (result1 == result2)   //需要对==进行重载操作 （！=同理）
    {
      ....
    }
    std::cin.get();
}
```

## C++this关键字

- C++中有this关键字，通过他我们可以访问成员函数，成员函数就是属于某个类的函数或方法。
- this在一个**const函数**中，this是一个`const Entity* const`或者是`const Entity*`,在一个**非const函数**中，那么它就是一个`Entity*`类型的
- 在函数内部，我们可以引用this，**this是指向这个函数所属的当前对象实例的指针**

> 所以我们可以在C++中写一个**非静态**的方法，为了去调用这个方法，我们需要先**实例化**一个对象，然后再去调用这个方法，所以这个方法必须由一个**有效对象**来调用，而**this关键字**就是指向那个对象的**指针**

```cpp
  class Entity
  {
    public:
      int x,y;
      Entity(int x, inty)
      {
          
      }
  };
```

对于上面Entity类中的代码，构造函数中如果过要对**成员变量x,y**进行赋值我可以用成员初始化的方式来进行赋值。

但是假如我不想这么做，我想要**在函数内部**做这个操作，那可能就会遇到点问题了，因为**成员变量**的名字和**传入参数**的名字**相同**，所以，如果：

```cpp
Entity(int x, inty)
 {
        x = x;     
 }
```

这其实只是在用它自己给这个x参数进行赋值操作，这相当于啥都没干。

我其实真正要做的是引用属于**这个类的x和y**，这个类的成员。而**this关键字**可以让我们做到这点，因为**this是指向当前对象的指针**。

```cpp
Entity(int x, inty)
 {
     Entity* e = this;
        e->x = x;     
 }
```

更清晰一点

```cpp
Entity(int x, inty)
 {
        this->x = x;  //同(*this).x = x;   
        this->y = y; 
 }
```

如果我们要写一个返回其中一个变量的函数的话，比如:

```cpp
class Entity
{
public:
 int x,y;
 Entity(int x, inty)
 {
        this->x = x;   
        this->y = y; 
 }
 int GetX() const  //在函数后面加上const是很常见的，因为他不会修改这个类
 {
 `  Entity* e = this;//ERROR!
        const Entity* e= this;//ok
     `e->x = 5；//ERROR！
 }
};
```

在这个**const函数**(`int GetX() const`)里，我们**不能写**`Entity* e= this`，而应该是`const Entity* e= this`。

因为**函数后面加上const**就意味着我们不能修改这个类，所以**this必须是const**的。

所以也不能写`e->x = 5;`如果没有这个const倒是可以这么写。

```cpp
int GetX() const  
 {
    const Entity* e= this;//ok
    e->x = 5；//ERROR！
        //所以也不能写`e->x = 5;`如果没有这个const倒是可以这么写。
    Entity* e= this;//ERROR！
    e->x = 5；//ok
 }
```

> 另一个用到的场景就是，如果我们想要调用这个Entity类外面的函数，他不是Entity的方法，但是我们想在这个类内部调用一个外部的函数，然后这个函数接受一个Entity类型作为参数，这时候就可以使用**this**

~~~cpp
class Entity;  //前置声明。
void PrintEntity(Entity* e); //在这里声明
class Entity  
{
  public:
    int x,y;
    Entity(int x, inty)
    {
        // Entity* e = this;
        this->x = x;   
        this->y = y; 
        PrintEntity(this); //我们希望能在这个类里调用PrintEntity,就可以传入this，这就会传入我已经设置了x和y的当前实例
    }
}; 
void PrintEntity(Entity* e) //在这里定义
{
    //print something
}
如果我想**传入一个常量引用**，我要做的就是在这里进行**解引用this**

  ```cpp
  void PrintEntity(const Entity& e);
  class Entity  
  {
    public:
      int x,y;
      Entity(int x, inty)
      {
          // Entity* e = this;
            this->x = x;   
            this->y = y; 
          PrintEntity(*this); //
      }
  }; 
  void PrintEntity(const Entity& e) 
  {
      //print something
  }
~~~

在**非const函数**里通过**解引用this**，我们就可**赋值给Entity&**，如果是在**const方法**中，我们会得到一个**const引用**

```cpp
  void PrintEntity(const Entity& e);
  class Entity  
  {
    public:
      int x,y;
      Entity(int x, inty)
      {
          // Entity* e = this;
     		this->x = x;   
     		this->y = y; 
          Entity& e = *this;  //在非const函数里通过解引用this，我们就可赋值给Entity&
          PrintEntity(*this); //解引用this
      }
      int GetX() const  
      {
  		const Entity& e = *this; //在const方法中，我们会得到一个const引用
      }
  }; 
  void PrintEntity(const Entity& e) 
  {
      //print something
  }
```

## C++的对象生存期（栈作用域生存期）

1.基于栈的变量生存周期是什么意思

> 这些分为两部分：一个是你必须要明白对象是如何生存在栈上的，这样你才会写出能正常工作不会崩溃的代码

2.作用域可以是任何东西，比如说函数作用域，还有像if语句作用域，或者for和while循环作用域，或者空作用域、类作用域。

3.每当我们在C++中进入一个作用域，我们是在push栈帧。它不一定就是一个栈帧。

> 当我在push数据时，你可以想象成把一本书放到书堆上，在这个作用域声明的变量一就是你再这本书里写的内容，一旦这个作用域结束，你就把这本书从书堆上拿出来，他就结束了，每个基于栈的变量，就是你再那本书里创建的对象就都结束了

4.**基于栈的变量在我们离开作用域的时候就会被摧毁，内存被释放**。在堆上创建的，当程序结束后才会被系统摧毁。

### 局部作用域创建数组的经典错误

例如：返回一个在作用域内创建的数组

> 如下代码，因为我们没有使用new关键字，所以他不是在堆上分配的，我们只是在栈上分配了这个数组，当我们返回一个指向他的指针时(`return array`)，也就是返回了一个指向栈内存的指针，旦离开这个作用域（CreateArray函数的作用域），这个栈内存就会被回收

```cpp
int CreateArray()
{
    int array[50];  //在栈上创建的
    return array;
}
int main()
{
    int* a = CreateArray(); //不能正常工作
}
```

如果你想要像这样写一个函数，那你一般有**两个选择**

> 1.在堆上分配这个数组，这样他就会一直存在

```cpp
int CreateArray()
{
    int* array = new int[50];  //在堆上创建的
    return array;
}
```

> 2.将创建的数组赋值给一个在这个作用域外的变量

比如说，我在这里创建一个大小为50的数组，然后把这个数组作为一个参数传给这个函数，当然在这个CreateArray函数里就不需要再创建数组了，但是我们可以对传入的数组进行操作，比如，填充数组，因为我们只是闯入了一个指针，所以不会做分配的操作。

```cpp
int CreateArray(int* array)
{
//填充数组
}
int main()
{
   int array[50];
    CreateArray(array); //不能正常工作
}
```

### 基于栈的变量的好处

1.可以帮助我们自动化代码。 比如类的作用域，比如像智能指针smart_ptr，或是unique_ptr，这是一个作用域指针，或者像作用域锁（scoped_lock）。

2.最简单的例子可能是作用域指针，它基本上是一个类，它是一个指针的包装器，在构造时用堆分配指针，然后在析构时删除指针，所以我们可以自动化这个new和delete。

> 创建Entity对象时，我还是想在堆上分配它，但是我想要在跳出作用域时自动删除它，这样能做到吗？我们可以使用标准库中的作用域指针unique_ptr实现。
> 如下，ScopedPtr就是我们写的一个最基本的作用域指针，由于其是在栈上分配的，然后作用域结束的时候，ScopedPtr这个类就被析构，析构中我们又调用delete把堆上的指针删除内存。

```cpp
#include <iostream>

class Entity
{
private:

public:
    Entity()
    {
        std::cout << "Create!" << std::endl;
    }
    ~Entity()
    {
        std::cout << "Destroy!" << std::endl;
    }
};

class ScopedPtr
{
private:
    Entity* m_Ptr;
public:
    ScopedPtr(Entity* ptr)
        : m_Ptr(ptr)
    {
    }

    ~ScopedPtr()
    {
        delete m_Ptr;
    }
};

int main()
{
    {
        ScopedPtr test = new Entity();  //发生隐式转换。虽然这里是new创建的，但是不同的是一旦超出这个作用域，他就会被销毁。因为这个ScopedPtr类的对象是在栈上分配的
    }

    std::cin.get();
}
```

## 智能指针

- 智能指针本质上是原始指针的包装。当你创建一个智能指针，它会调用new并为你分配内存，然后基于你使用的智能指针，这些内存会在某一时刻自动释放。
- 优先使用unique_ptr，其次考虑shared_ptr。

> 尽量使用unique_ptr因为它有一个较低的开销，但如果你需要在对象之间共享，不能使用unique_ptr的时候，就使用shared_ptr

### 作用域指针unique_ptr的使用

- 要访问所有这些智能指针，你首先要做的是包含**memory头文件**
- unique_ptr是作用域指针，意味着超出作用域时，它会被销毁然后调用delete。
- unique_ptr是**唯一的**，不可复制，不可分享。

> 如果复制一个unique_ptr，会有两个指针，两个unique_ptr指向同一个内存块，如果其中一个死了，它会释放那段内存，也就是说，指向同一块内存的第二个unique_ptr指向了已经被释放的内存。

- unique_ptr构造函数实际上**是explicit的**，没有构造函数的隐式转换,需要**显式**调用构造函数。
- 最好使用`std::unique_ptr<Entity> entity = std::make_unique<Entity>();` 因为如果构造函数碰巧抛出异常，不会得到一个没有引用的悬空指针从而造成内存泄露,它会稍微安全一些。
- `std::make_unique<>()`是在**C++14**引入的，C++11不支持。

```cpp
#include <iostream>
   #include <memory>

   class Entity
   {
   private:

   public:
    Entity()
    {
        std::cout << "Create!" << std::endl;
    }

    ~Entity()
    {
        std::cout << "Destroy!" << std::endl;
    }
       void Print(){}
   };

   int main()
   {
    {
           //使用unique_ptr的一种方式
        std::unique_ptr<Entity> entity = new Entity(); // error! unique_ptr不能隐式转换
        std::unique_ptr<Entity> entity(new Entity());//ok,可以但不建议
           std::unique_ptr<Entity> entity = std::make_unique<Entity>(); //推荐使用std::make_unique。因为如果构造函数碰巧抛出异常，它会稍微安全一些。std::make_unique<>()是在C++14引入的，C++11不支持。
           entity->Print();  //像一般原始指针使用

    }

    std::cin.get();
   }
```

### 共享指针shared_ptr 的使用

1.shared_ptr的工作方式是**通过引用计数**。

> 引用计数基本上是一种方法，可以跟踪你的指针有多少个引用，一旦引用计数达到零，他就被删除了。
> 例如：我创建了一个共享指针shared_ptr，我又创建了另一个shared_ptr来复制它，我的引用计数是2，第一个和第二个，共2个。当第一个死的时候，我的引用计数器现在减少1，然后当最后一个shared_ptr死了，我的引用计数回到零，内存就被释放。

2.shared_ptr需要分配另一块内存，叫做控制块，用来存储引用计数，如果您首先创建一个new Entity，然后将其传递给shared_ptr构造函数，它必须分配，做2次内存分配。先做一次new Entity的分配，然后是shared_ptr的控制内存块的分配。然而如果你用make_shared你能把它们组合起来，这样更有效率。

```cpp
std::shared_ptr<Entity> sharedEntity = sharedEntity(new Entity());//不推荐！
std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();//ok
```

3.使用格式： `std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();`

```cpp
#include <iostream>
#include <memory>

class Entity
{
private:

public:
    Entity()
    {
        std::cout << "Create!" << std::endl;
    }
    ~Entity()
    {
        std::cout << "Destroy!" << std::endl;
    }
};

int main()
{
    {
        std::shared_ptr<Entity> e0;
        {
        //  std::shared_ptr<Entity> sharedEntity = sharedEntity(new Entity());//不推荐！
            std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();//ok
            e0 = sharedEntity; //可以复制
        } //此时sharedEntity已经“死了”,但没有调用析构，因为e0仍然是活的，并且持有对该Entity的引用，此时计数由2-》1
    } //析构被调用，因为所有的引用都消失了，计数由2-》0，内存被释放

    std::cin.get();
}
```

shared_ptr的实现机制是在拷贝构造时使⽤同⼀份引⽤计数 

（1）⼀个模板指针T* ptr 指向实际的对象  

（2）⼀个引⽤次数 必须new出来的，不然会多个shared_ptr⾥⾯会有不同的引⽤次数⽽导致多次delete 

（3）重载operator*和operator-> 使得能像指针⼀样使⽤shared_ptr 

（4）重载copy constructor 使其引⽤次数加⼀（拷贝构造函数) 

（5）重载operator=（赋值运算符） 如果原来的shared_ptr已经有对象，则让其引⽤次数减⼀并判断引⽤是否为零(是否调⽤ delete)，然后将新的对象引⽤次数加⼀ 

（6）重载析构函数 使引⽤次数减⼀并判断引⽤是否为零; (是否调⽤delete) 

线程安全问题 

（1）同⼀个shared_ptr被多个线程“读”是安全的; 

（2）同⼀个shared_ptr被多个线程“写”是不安全的; 

 证明：在多个线程中同时对⼀个shared_ptr循环执⾏两遍swap。 shared_ptr的swap函数的 作⽤就是和另外⼀个shared_ptr交换引⽤对象和引⽤计数，是写操作。执⾏两遍swap之后, shared_ptr引⽤的对象的值应该不变） 

（3）共享引⽤计数的不同的shared_ptr被多个线程”写“ 是安全的。

### 弱指针weak_ptr

1.可以和共享指针shared_ptr一起使用。

2.weak_ptr可以被复制，但是同时**不会增加额外的控制块来控制计数**，仅仅声明这个指针还活着。

> 当你将一个shared_ptr赋值给另外一个shared_ptr，引用计数++，而若是**把一个shared_ptr赋值给一个weak_ptr时，它不会增加引用计数**。这很好，如果你不想要Entity的所有权，就像你可能在排序一个Entity列表，你不关心它们是否有效，你只需要存储它们的一个引用就可以了。

```cpp
{
    std::weak_ptr<Entity> e0;
    {
        std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>();
        e0 = sharedEntity;
    } //此时，此析构被调用，内存被释放
}
```

### auto_ptr

被 c++11 弃用，原因是缺乏语言特性如 “针对构造和赋值” 的 `std::move` 语义，以及其他瑕疵。

#### auto_ptr 与 unique_ptr 比较

- auto_ptr 可以赋值拷贝，复制拷贝后所有权转移；unqiue_ptr 无拷贝赋值语义，但实现了`move` 语义；
- auto_ptr 对象不能管理数组（析构调用 `delete`），unique_ptr 可以管理数组（析构调用 `delete[]` ）；

## C++的拷贝与拷贝构造函数

1.拷贝构造函数的格式：

```cpp
//声明：
T(const T& var);
//定义
T(const T& var){
    //函数体，进行深拷贝 分配空间放副本
}
//不使用拷贝函数，禁止赋值
T(const T& var) = delete;
```

2.每当你编写一个变量被赋值另一个变量的代码时，你**总是**在复制。在指针的情况下，你在复制指针，也就是内存地址，内存地址的数字，就是数字而已，而不是指针指向的实际内存。

3.“成员树”不包括指针和引用时，浅拷贝和深拷贝没区别。

4.浅拷贝只拷贝基本数据类型（非指针变量）

> 浅拷贝的问题是如果对象中变量带有指针（或引用）,则会发生错误.因为**两个指针指向同一个内存**,**一个对象修改,另一个对象的值也被更改了.** 当在**析构**的时候,会**发生两次free (double free)**同一个内存，造成错误.

```cpp
class String
{
private:
    char* m_Buffer;
    unsigned m_Size;
public:
    String(const char* string)
    {
        m_Size = strlen(string); //计算字符串的长度，这样就可以把这个字符串的数据复制到缓冲区中
        m_Buffer = new char[m_Size+1]; //需要一个空终止符，所以+1.(也可以使用strcpy函数（拷贝时，包含了空终止字符）)
        memcpy(m_Buffer,string,m_Size); //是把这个指针复制到我们实际的缓冲区中，这样缓冲区就会被我们的字符填充
        m_Buffer[m_Size] = 0;  //手动在最后添加自己的空终止符。不在上一行代码中写m_Size+1的原因是为了避免：char* string这个字符串不能正常的通过空终止符结束而造成错误。

    }
    ~String()
    {
        delete [] m_Buffer;
    }
    char& operator[](const unsigned index) //[]操作符重载
    {
        return m_Buffer[index];
    }
    friend std::ostream& operator<<(std::ostream& stream, const String& string);//把<<操作符重载函数声明为String类的友元，这样就可以在该重载函数中访问m_Buffer
};
std::ostream& operator<<(std::ostream& stream, const String& string) //<<操作符重载，用来打印我创建的字符串
{
    stream<<string.m_Buffer;
    return stream;
}
```

基于以上的代码，复制一个字符串并输出：

```cpp
int main()
{   
    String string = "Cherno";
    String second = string;
    std::cout << string << std::endl;
    std::cout << second << std::endl;
    std::cin.get();
}
//输出
Cherno
Cherno
------------------------------------------
int main()
{   
    String string = "Cherno";
    String second = string;
    second[2] = 'a';
    std::cout << string << std::endl;
    std::cout << second << std::endl;
    std::cin.get();
}
//输出,两个都被改变了
Charno
Charno
```

正常输出后，运行到`std::cin.get();`此时敲击回车键，**程序崩溃！**

造成该错误的原因时：String类中含有一个指针变量和一个整形变量，复制字符串是，对这两个变量也进行了赋值，但对这个指针只复制了他的内存地址，于时此时由两个指针，**这两个指针指向同一个内存,一个对象修改,另一个对象的值也被更改了.**并且，当在**析构**的时候,会**发生两次free (double free)**同一个内存，造成错误.。

为了解决这个问题，据需要使用**拷贝构造函数**

> 拷贝构造函数是一个构造函数，当你复制第二个字符串时，它会被调用，当你把一个字符串赋值给一个对象时，这个对象也是一个字符串。当你试图创建一个新的变量并给它分配另一个变量时，它（这个变量）和你正在创建的变量有相同的类型。你复制这个变量，也就是所谓的拷贝构造函数

```cpp
class String
{
private:
    char* m_Buffer;
    unsigned m_Size;
public:
    String(const char* string)
    {
        m_Size = strlen(string); 
        m_Buffer = new char[m_Size+1];
        memcpy(m_Buffer,string,m_Size);
        m_Buffer[m_Size] = 0;  

    }
    String(const String& other):m_Size(other.m_Size)  //创建拷贝构造函数
    {
        m_Buffer = new char[m_Size+1];  //分配一个新的缓冲区
        memcpy(m_Buffer,other.m_Buffer,m_Size+1); //知道other的大小了，other字符串已经有了一个空终止字符，因为它是一个字符串，必须有空终止字符。
    } // 此函数为拷贝构造函数，new出一块内存，复制原来的数组
    ~String()
    {
        delete [] m_Buffer;
    }
    char& operator[](const unsigned index) 
    {
        return m_Buffer[index];
    }
    friend std::ostream& operator<<(std::ostream& stream, const String& string);
};
std::ostream& operator<<(std::ostream& stream, const String& string) 
{
    stream<<string.m_Buffer;
    return stream;
}
```

## C++的箭头操作符

**1.特点：**

- 箭头运算符必须是类的成员。
- 一般将箭头运算符定义成了const成员，这是因为与递增和递减运算符不一样，获取一个元素并不会改变类对象的状态。

**2.对箭头运算符返回值的限定**

> 箭头运算符的重载**永远不能丢掉成员访问**这个最基本的含义。当我们重载箭头时，可以改变的是箭头从哪个对象当中获取成员，而箭头获取成员这一事实则永远不变。 对于形如point->mem的表达式来说，point必须是指向类对象的指针或者是一个重载了operator->的类的对象。根据point类型的不同，point->mem分别等价于

```cpp
(*point).mem； //point 是一个内置的指针类型
point.operator()->mem； //point是类的一个对象
```

重载的箭头运算符**必须返回类的指针或者自定义了箭头运算符的某个类的对象。**

**3.三种应用场景**

1)可用于指针调用成员：p->x 等价于 (*p).x

2)重载箭头操作符

```cpp
#include <iostream>
class Entity
{
private:
    int x;
public:
    void Print() 
    {
        std::cout << "Hello!" << std::endl;
    }
};
class ScopedPtr
{
private:
    Entity* m_Ptr;
public:
    ScopedPtr(Entity* ptr)
        : m_Ptr(ptr)
    {
    }
    ~ScopedPtr()
    {
        delete m_Ptr;
    }
    Entity* operator->()  //重载操作符
    {
        return m_Ptr;
    }
};

int main()
{
    {
        ScopedPtr entity = new Entity();
        entity->Print();
    }
    std::cin.get();
}
```

进一步,可以写为const版本的：

```cpp
#include <iostream>
class Entity
{
private:
    int x;
public:
    void Print() const   //添加const
    {
        std::cout << "hello!" << std::endl;
    }
};

class ScopedPtr
{
private:
    Entity* m_Ptr;
public:
    ScopedPtr(Entity* ptr)
        : m_Ptr(ptr)
    {
    }
    ~ScopedPtr()
    {
        delete m_Ptr;
    }
    Entity* operator->()
    {
        return m_Ptr;
    }
    const Entity* operator->() const //添加const
    {
        return m_Ptr;
    }
};

int main()
{
    {
        const ScopedPtr entity = new Entity(); //如果是const，则上面代码要改为const版本的。
        entity->Print();
    }
    std::cin.get();
}
```

3)可用于计算成员变量的offset：

> 引自B站评论：
> 因为"指针->属性"访问属性的方法实际上是通过把指针的值和属性的偏移量相加，得到属性的内存地址进而实现访问。 而把指针设为nullptr(0)，然后->属性就等于0+属性偏移量。编译器能知道你指定属性的偏移量是因为你把nullptr转换为类指针，而这个类的结构你已经写出来了(float x,y,z)，float4字节，所以它在编译的时候就知道偏移量(0,4,8)，所以无关对象是否创建

```cpp
struct vec2
{
    int x,y;
    float pos,v;
};
int main()
{   
    int offset = (int)&((vec2*)nullptr)->x; // x,y,pos,v的offset分别为0,4,8,12
    std::cout<<offset<<std::endl;
    std::cin.get();
}
```

## C++的动态数组(std::vector)

1.vector本质上是一个动态数组,是内存连续的数组

2.它的使用需要包含头文件`#include <vector>`

3.使用格式：

类型尽量**使用对象**而非指针。

```cpp
std::vector<T> a；//T是一种模板类型，尽量使用对象而非指针
```

4.添加元素

```cpp
a.push_back(element); // 后面插入
    //定义一个类
    struct Vertex
    {
        float x, y, x;
    }

    std::vector<Vertex> vertices; //定义一个Vertex类型的动态数组
    vertices.push_back({ 1, 2, 3 });//列表初始化（结构体或者类，可以按成员声明的顺序用列表构造）
    vertices.push_back({ 4, 5, 6 });//同：vertices.push_back(Vertex(4, 5, 6)
    vertices.push_back({ 7, 8, 9 });
```

5.初始化

遍历

> for遍历

```cpp
for(int i =0; i < vertices.size();i++)
{
    std::cout << vertices[i] << std::endl;
}
```

> 范围for循环遍历

```cpp
for(Vertex& v : vertices)  //引用，避免复制浪费。
{
    std::cout << v << std::endl;
}
```

6.清除数组列表

```cpp
vertices.clear();
```

7.清除指定元素

> 例如：清除第二个元素

```cpp
vertices.erase(vertices.begin()+1) //参数是迭代器类型
```

8.参数传递时，如果不对数组进行修改，请使用引用类型传参。

```cpp
void Function(const std::vector<T>& vec){};
```

## C++的std::vector使用优化

vecctor的优化策略：

> **问题1：**当向vector数组中**添加新元素**时，为了扩充容量，**当前的vector的内容会从内存中的旧位置复制到内存中的新位置**(产生一次复制)，然后删除旧位置的内存。 简单说，push_back时，容量不够，会自动调整大小，重新分配内存。这就是将代码拖慢的原因之一。 **解决办法：** vertices.reserve(n) ，直接指定容量大小，避免重复分配产生的复制浪费。
> **问题2：**在非vector内存中创建对象进行初始化时，即push_back() 向容器尾部添加元素时，首先会创建一个临时容器对象（不在已经分配好内存的vector中）并对其追加元素，然后再将这个对象拷贝或者移动到【我们真正想添加元素的容器】中 。这其中，就造成了一次复制浪费。 **解决办法：** **emplace_back**，直接在容器尾部创建元素，即直接在已经分配好内存的那个容器中直接添加元素，不创建临时对象。

简单的说：

> **reserve提前申请内存**，避免动态申请开销 
>
> **emplace_back直接在容器尾部创建元素**，省略拷贝或移动过程

```cpp
#include <iostream>
#include <vector>

struct Vertex
{
    float x, y, z;

    Vertex(float x, float y, float z)
        : x(x), y(y), z(z)
    {
    }

    Vertex(const Vertex& vertex)
        : x(vertex.x), y(vertex.y), z(vertex.z)
    {
        std::cout << "Copied!" << std::endl;
    }
};

int main()
{
    std::vector<Vertex> vertices;
    vertices.push_back(Vertex(1, 2, 3 )); //同vertices.push_back({ 1, 2, 3 });
    vertices.push_back(Vertex(4, 5, 6 ));
    vertices.push_back(Vertex(7, 8, 9 ));

    std::cin.get();
}
```

输出：

```cpp
Copied!
Copied!
Copied!
Copied!
Copied!
Copied!
```

**发生六次复制的原因：**

理解一：

> 环境:VS2019，x64，C++17标准，经过我自己的测试，vector扩容因子为1.5，初始的capacity为0.
> 第一次push_back，capacity扩容到1，临时对象拷贝到真正的vertices所占内存中，第一次Copied；第二次push_back，发生扩容，capacity扩容到2，vertices发生内存搬移发生的拷贝为第二次Copied，然后再是临时对象的搬移，为第三次Copied；接着第三次push_back，capacity扩容到3（2*1.5 = 3，3之后是4，4之后是6...），vertices发生内存搬移发生的拷贝为第四和第五个Copied，然后再是临时对象的搬移为第六个Copied；

理解二：

```cpp
std::vector<Entity> e;
    Entity data1 = { 1,2,3 }; 
    e.push_back( data1); // data1->新vector内存
    Entity data2 = { 1,2,3 }; 
    e.push_back( data2 ); //data1->新vector内存   data2->vector新vector内存  删除旧vector内存
    Entity data3 = { 1,2,3 };
    e.push_back(data3);  // data1->新vector内存  data2->vector新vector内存  data3->vector新vector内存  删除旧vector内存
所以他的输出的次数分别是1，3，6
他的复制次数你可以这样理解递增。 1+2+3+4+5+....
```

解决:

```cpp
int main()
{   
    std::vector<Vertex> vertices;
    //ver 1 : copy 6 times
    vertices.push_back({ 1,2,3 });
    vertices.push_back({ 4,5,6 });
    vertices.push_back({ 7,8,9 });

    //ver 2 : copy 3 times
    vertices.reserve(3);
    vertices.push_back({ 1,2,3 });
    vertices.push_back({ 4,5,6 });
    vertices.push_back({ 7,8,9 });

    //ver 3 : copy 0 times
    vertices.reserve(3);
    vertices.emplace_back(1, 2, 3);
    vertices.emplace_back(4, 5, 6);
    vertices.emplace_back(7, 8, 9);

    std::cin.get();
}
```

## 库（待补充）

### C++中使用库（静态链接）

C++创建一个动态链接库，编译后会生成两个可用的文件一个是lib文件一个是dll文件，那么这个lib文件是干嘛的呢？
在使用动态库的时候，往往提供两个文件：一个引入库和一个DLL。引入库包含被DLL导出的函数和变量的符号名，DLL包含实际的函数和数据。在编译链接可执行文件时，只需要链接引入库，DLL中的函数代码和数据并不复制到可执行文件中，在运行的时候，再去加载DLL，访问DLL中导出的函数。
**1． Load-time Dynamic Linking 载入时动态链接**
这种用法的前提是在编译之前已经明确知道要调用DLL中的哪几个函数，编译时在目标文件中只保留必要的链接信息，而不含DLL函数的代码；当程序执行时，利用链接信息加载DLL函数代码并在内存中将其链接入调用程序的执行空间中，其主要目的是便于代码共享。
**2． Run-time Dynamic Linking 运行时动态链接**
这种方式是指在编译之前并不知道将会调用哪些DLL函数，完全是在运行过程中根据需要决定应调用哪个函数，并用LoadLibrary和GetProcAddress动态获得DLL函数的入口地址。

### C++中使用动态库

### C++中创建与使用库(VisualStudio多项目)

## C++中如何处理多返回值

### 方法一：通过函数参数传引用或指针的方式

> 把函数定义成`void`，然后通过**参数引用传递**的形式“返回”两个字符串，这个实际上是修改了目标值，而不是返回值，但某种意义上它确实是返回了两个字符串，而且没有复制操作，技术上可以说是很好的。但这样做会使得函数的形参太多了，可读性降低，有利有弊 。

```cpp
#include <iostream>
void GetUserAge(const std::string& user_name,bool& work_status,int& age)
{
    if (user_name.compare("xiaoli") == 0)
    {
        work_status = true;
        age = 18;
    }
    else
    {
        work_status = false;
        age = -1;
    }
}

int main()
{
    bool work_status = false;
    int age = -1;
    GetUserAge("xiaoli", work_status, age);
    std::cout << "查询结果：" << work_status << "    " << "年龄：" << age << std::endl;
    getchar();
    return 0;
}
```

### 方法二： 通过函数的返回值是一个`array`（数组）或vector

> 当然，这里也可以返回一个vector，同样可以达成返回多个数据的目的。
> 不同点**是Array是在栈上创建，而vector会把它的底层储存在堆上**，所以从技术上说，返回Array会更快
> 但以上方法都**只适用于相同类型**的多种数据的返回

```cpp
//设置是array的类型是stirng，大小是2
std::array<std::string, 2> ChangeString() {
    std::string a = "1";
    std::string b = "2";

    std::array<std::string, 2> result;
    result[0] = a;
    result[1] = b;
    return result;

    //也可以return std::array<std::string, 2>(a, b);
}
```

### 方法三：使用std::pair返回两个返回值

> 可以**返回两个不同类型**的数据返。
> 使用std::pair这种抽象数据结构，该数据结构可以绑定两个异构成员。这种方式的弊端是**只能返回两个值。**

```cpp
#include <iostream>

std::pair<bool, int> GetUserAge(const std::string& user_name)
{
    std::pair<bool, int> result;

    if (user_name.compare("xiaoli") == 0)
    {
        result = std::make_pair(true, 18);
    }
    else
    {
        result = std::make_pair(false, -1);
    }

    return result;
}

int main()
{
    std::pair<bool, int> result = GetUserAge("xiaolili");
    std::cout << "查询结果：" << result.first << "   " << "年龄：" << result.second << std::endl;
    getchar();
    return 0;
}
```

### 方法四：使用std::tuple返回三个或者三个以上返回值

> std::tuple这种抽象数据结构可以将三个或者三个以上的异构成员绑定在一起，返回std::tuple作为函数返回值理论上可以返回三个或者三个以上的返回值。
> `tuple`相当于一个类，它可以包含x个变量，但他不关心类型，用`tuple`需要包含头文件`#include`

```cpp
#include <iostream>
#include <tuple>

std::tuple<bool, int,int> GetUserAge(const std::string& user_name)
{
    std::tuple<bool, int,int> result;

    if (user_name.compare("xiaoli") == 0)
    {
        result = std::make_tuple(true, 18,0);
    }
    else
    {
        result = std::make_tuple(false, -1,-1);
    }

    return result;
}

int main()
{
    std::tuple<bool, int,int> result = GetUserAge("xiaolili");

    bool work_status;
    int age;
    int user_id;

    std::tie(work_status, age, user_id) = result;
    std::cout << "查询结果：" << work_status << "    " << "年龄：" << age <<"   "<<"用户id:"<<user_id <<std::endl;
    getchar();
    return 0;
}
```

#### 配合使用结构化绑定

#### C++函数：std::tie 详解

方法是：

1. 从要返回多重返回值的函数中，返回一个 std::tuple ，里面包含多个具体要返回的值
2. 接收端使用 std::tie 来解包 tuple

比如，函数定义成：

```text
std::tuple<int, double> foo()
{
    return std::tuple<int, double>(42, 3.14);
}
```

接收端可以这么写：

```text
int a;
double d;

std::tie(a, d) = foo();
```

如果不需要某个返回值，可以使用 std::ignore 来接收：

```text
std:tie(std::ignore, d) = foo(); // 忽略掉第一个返回值
```

### 方法五：返回一个结构体(推荐)

> 结构体是在栈上建立的，所以在技术上速度也是可以接受的
> 而且不像用pair的时候使用只能`temp.first, temp.second`，这样不清楚前后值是什么，可读性不佳。而如果换成`temp.str, temp.val`后可读性极佳，永远不会弄混！

```cpp
#include <iostream>
struct result {
    std::string str;
    int val;
};
result Function () {
    return {"1", 1};//C++新特性，可以直接这样子让函数自动补全成结构体
}
int main() {
    auto temp = Function();
    std::cout << temp.str << ' ' << temp.val << std::endl;
}
--------------------------------------------
#include <iostream>
using namespace std;

struct Result
{
    int add;
    int sub;
};

Result operation(int a, int b)
{
    Result ret;
    ret.add = a + b;
    ret.sub = a - b;
    return ret;
}

int main()
{
    Result res;
    res = operation(5, 3);
    cout << "5+3=" << res.add << endl;
    cout << "5-3=" << res.sub << endl;
}
```

### 方法六：C++的结构化绑定



## C++的模板

**模板**：模板允许你定义一个可以根据你的用途进行编译的模板（有意义下）。故所谓模板，就是让编译器基于DIY的规则去为你写代码 。

### 函数的模板（对形参）

> 不使用模板

```cpp
  void Print(int temp) {
      cout << temp;
  }
  void Print(string temp) {
      cout << temp;
  }
  void Print(double temp) {
      cout << temp;
  }
  int main() {
      Print(1);
      Print("hello");
      Print(5.5);
      //如果要用一个函数输出三个类型不同的东西，则要手动定义三个不同重载函数
      //这其实就是一种复制粘贴就可以完成的操作
  }
```

> 使用模板

格式： **`template<typename T>`**

```cpp
template<typename T> void Print(T temp) {
    //把类型改成模板类型的名字如T就可以了
    cout << temp;
}
//干净简洁
int main() {
    Print(1);
    Print("hello");
    Print(5.5);
}
```

> 通过`template`定义，则说明定义的是一个模板，它会在编译期被评估，**所以`template`后面的函数其实不是一个实际的代码**，**只有当我们实际调用时，模板函数才会基于传递的参数来真的创建** 。 只有当真正调用函数的时候才会被实际创建 。

模板参数

```cpp
template<typename T> void Print(T temp) {
    cout << temp;
}
int main() {
    Print(96);//这里其实是隐式的传递信息给模板，可读性不高
    Print<int>(96);//可以显示的定义模板参数，声明函数接受的形参的类型！！！
    Print<char>(96);//输出的可以是数字，也可以是字符！这样的操纵性强了很多！！！
}
```

### 类的模板

> 传递数字给模板，来指定要生成的类

```cpp
//不仅仅是typename!
template<int N> class Array {
private:
    //在栈上分配一个数组，而为了知道它的大小，要用模板传一个数字N过来
    int m_Array[N];
};
int main() {
    Array<5> array;//用尖括号给模板传递构造的规则。惊了，看起来尖括号就是对模板进行操作的好东西
}
```

> 传多个规则给模板，**用逗号隔开就行**

```cpp
//可以传类型，也可以传数字，功能太强大了
//两个模板参数：类型和大小
template<typename T, int size> class Array {
private:
    T m_Array[size];
};
int main() {
    Array<int, 5> array;
}
```

**提醒：不要滥用模板！**

### 拓展：模板特例化（待补充）

## C++的堆和栈内存的比较

1.当我们的程序开始的时候，程序被分成了一堆不同的内存区域，除了堆和栈以外，还有很多东西，但我们最关心这两个 。

2.**栈**通常是一个预定义大小的内存区域，通常约为2兆字节左右。**堆**也是一个预定义了默认值的区域，**但是它可以**随着应用程序的进行而改变。

3.**栈和堆内存区域的实际位置（物理位置）在ram中完全一样**（并不是一个存在CPU缓存而另一个存在其他地方）

> 在程序中，内存是用来实际储存数据的。我们需要一个地方来储存允许程序所需要的数据（比如局部变量or从文件中读取的东西）。而栈和堆，它们就是可以储存数据的地方，但**栈和堆的工作原理非常非常不同**，但本质上它们做的事情是一样的

4.栈和堆的区别

区别一：定义格式不同

```cpp
//在栈上分配
int val = 5; 
//在堆上分配
int *hval = new int;    //区别是，我们需要用new关键词来在堆上分配
*hval = 5;
```

区别二：内存分配方式不同

对栈来说：

> **在栈上**，分配的内存都是**连续**的。添加一个int，则**栈指针（栈顶部的指针）**就移动4个字节，所以连续分配的数据在内存上都是**连续**的。栈分配数据是直接把数据堆在一起（所做的就是移动栈指针），所以栈分配数据会很快 。
> 如果离开作用域，在栈中分配的所有内存都会弹出，内存被释放。

对堆来说

> **在堆上**，分配的内存都是**不连续**的，`new`实际上做的是在内存块的**空闲列表**中找到空闲的内存块，然后把它用一个指针圈起来，然后返回这个指针。（但如果**空闲列表**找不到合适的内存块，则会询问**操作系统**索要更多内存，而这种操作是很麻烦的，潜在成本是巨大的）
> 离开作用域后，堆中的内存仍然存在

建议： **能在栈上分配就在栈上分配**，**不能够在栈上分配时或者有特殊需求时（比如需要生存周期比函数作用域更长，或者需要分配一些大的数据），才在堆上分配**

## C++的宏

1.**预处理阶段** ：当编译C++代码时，首先**预处理器**会过一遍C++所有的**以#符号开头（这是预编译指令符号）的语句，当预编译器将这些代码评估完后给到编译器去进行实际的编译**。

2.**宏和模板的区别**：**发生时间**不同，宏是在**预处理阶段**就被评估了，而模板会被评估的更晚一点。

3.**用宏的目的：**写一些宏将代码中的文本**替换**为其他东西（**纯文本替换**）（不一定是简单的替换，是可以自定义调用宏的方式的）

```cpp
#defind WAIT std::cin.get()
//这里可以不用放分号，如果放分号就会加入宏里面了
int main() {
    WAIT;
    //等效于std::cin.get()，属于纯文本替换
    //但单纯做这种操作是很愚蠢的，除了自己以外别人读代码会特别痛苦
}
```

4.宏的用法之一：**宏是可以发送参数的**

```cpp
#include <iostream>
//宏是可以传递参数的，虽然参数也是复制粘贴替换上去的，并没有像函数那样讲究
#define log(x) std::cout << x << std::endl
int main() {
    log("hello");
    //这样子会输出“hello”
    return 0;
}
```

5.宏可以辅助调试

> 在Debug模式下会有很多日志的输出，但是在Release模式下就不需要日志的输出了。正常的方法可能会删掉好多的输出日志的语句或者函数，**但是用宏可以直接取消掉这些语句**

利用宏中的`#if，#else`，`endif`来实现。如：

```cpp
#include <iostream>

#defind PR_DEBUG 1 //可以在这里切换成0，作为一个开关
#if PR_DEBUG == 1   //如果PR_DEBUG为1
#defind LOG(x) std::cout << x << std::endl  //则执行这个宏
#else   //反之
#defind LOG(x)   //这个宏什么也不定义，即是无意义
#endif    //结束

int main() {
    LOG("hello");
    return 0;
}
```

如果在Debug(PR_DEBUG == 1)模式下，则会打印日志，如果在Release(PR_DEBUG == 0)模式，则在**预处理阶段就会把日志语句给删除掉**。

利用`#if 0`和`#endif`删除一段宏.

```cpp
#include <iostream>

#if 0   //从这里到最后的endif的宏都被无视掉了，某种意义上的删除

#defind PR_DEBUG 1 
#if PR_DEBUG == 1   
#defind LOG(x) std::cout << x << std::endl  
#else   
#defind LOG(x)  
#endif    

#endif  //结束

int main() {
    LOG("hello");
    return 0;
}
```

## C++的auto关键字

**auto的使用场景：**

> 在使用iterator 的时候，如：

```cpp
std::vector<std::string> strings;
strings.push_back("Apple");
strings.push_back("Orange");
for (std::vector<std::string>::iterator it = strings.begin(); //不使用auto
    it != strings.end(); it++)
{
    std::cout << *it << std::endl;
}
for (auto it = strings.begin(); it != strings.end(); it++) //使用auto
{
    std::cout << *it << std::endl;
}
```

> 当类型名过长的时候可以使用auto

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <unordered_map>

class Device{};

class DeviceManager
{
private:
    std::unordered_map<std::string, std::vector<Device *>> m_Devices;
public:
    const std::unordered_map<std::string, std::vector<Device *>> &GetDevices() const
    {
        return m_Devices;
    }
};

int main()
{
    DeviceManager dm;
    const std::unordered_map<std::string, std::vector<Device *>> &devices = dm.GetDevices();//不使用auto
    const auto& devices = dm.GetDevices(); //使用auto

    std::cin.get();
}
```

除此之外类型名过长的时候也可以使用using或typedef方法：

```cpp
using DeviceMap = std::unordered_map<std::string, std::vector<Device*>>;
typedef std::unordered_map<std::string, std::vector<Device*>> DeviceMap;

const DeviceMap& devices = dm.GetDevices();
```

**auto使用建议**：如果不是上面两种应用场景，请尽量不要使用auto！能不用，就不用！

### 注意：

（1）auto的使⽤必须马上初始化，否则⽆法推导出类型 

（2）auto在⼀⾏定义多个变量时，各个变量的推导不能产⽣⼆义性，否则编译失败 

（3）auto不能⽤作函数参数 

（4）在类中auto不能⽤作⾮静态成员变量 

（5）auto不能定义数组，可以定义指针 

（6）auto⽆法推导出模板参数 

（7）在不声明为引⽤或指针时，auto会忽略等号右边的引⽤类型和cv限定 

（8）在声明为引⽤或者指针时，auto会保留等号右边的引⽤和cv属性

使用 **`auto`** 将删除引用、 **`const`** 限定符和 **`volatile`** 限定符。 请考虑以下示例：

C++复制

```cpp
// cl.exe /analyze /EHsc /W4
#include <iostream>

using namespace std;

int main( )
{
    int count = 10;
    int& countRef = count;
    auto myAuto = countRef;

    countRef = 11;
    cout << count << " ";

    myAuto = 12;
    cout << count << endl;
}
```

在前面的示例中，myAuto 是 **`int`**而不是 **`int`** 引用，因此输出为 `11 11`，而不是 `11 12` ，如果引用限定符未由 **`auto`**删除，则不会如此。

## decltype关键字

decltype 关键字用于检查实体的声明类型或表达式的类型及值分类。语法：

```cpp
decltype ( expression )
```

decltype 使用

```cpp
// 尾置返回允许我们在参数列表之后声明返回类型
template <typename It>
auto fcn(It beg, It end) -> decltype(*beg)
{
    // 处理序列
    return *beg;    // 返回序列中一个元素的引用
}
// 为了使用模板参数成员，必须用 typename
template <typename It>
auto fcn2(It beg, It end) -> typename remove_reference<decltype(*beg)>::type
{
    // 处理序列
    return *beg;    // 返回序列中一个元素的拷贝
}
```

## C++的静态数组（std::array)

1.std::array是一个实际的标准数组类，是C++标准模板库的一部分。

2.**静态**的是指**不增长的数组**，当**创建array**时就要**初始化其大小**，不可再改变。

3.使用格式

```cpp
#include <array>  // 先要包含头文件
int main() {
    std::array<int, 5> data;  //定义，有两个参数，一个指定类型，一个指定大小
    data[0] = 1;
    data[4] = 10;
    return 0;
}
```

4.array和原生数组都是创建在栈上的（vector是在堆上创建底层数据储存的）

5.原生数组越界的时候不会报错，而array会有越界检查，会报错提醒。

6.使用std::array的好处是可以**访问它的大小**（通过s**ize()**函数），它是一个**类**。

```cpp
#include<iostream>
#include<array>

void PrintArray(const std::array<int, 5>& data)  //显式指定了大小
{
    for (int i = 0;i < data.size();i++)  //访问数组大小
    {
        std::cout << data[i] << std::endl;
    }
}
int main()
{
    std::array<int, 5> data;
    data[0] = 0;
    data[1] = 1;
    data[2] = 2;
    data[3] = 3;
    data[4] = 4;
    PrintArray(data);
    std::cin.get();
}
```

如何传入一个**标准数组作为参数**，但**不知道数组的大小**？

方法：使用模板

```cpp
#include <iostream>
#include <array>

template <typename T>
void printarray(const T &data)
{
    for (int i = 0; i < data.size(); i++)
    {
        std::cout << data[i] << std::endl;
    }
}

template <typename T, unsigned long N> // or // template <typename T, size_t N>
void printarray2(const std::array<T, N> &data)
{
    for (int i = 0; i < N; i++)
    {
        std::cout << data[i] << std::endl;
    }
}

int main()
{
    std::array<int, 5> data;
    data[0] = 2;
    data[4] = 1;
    printarray(data);
    printarray2(data);
}
//代码参考：https://github.com/UrsoCN/NotesofCherno/blob/main/Cherno57.cpp
```

## C语言风格的函数指针

1.定义方式：

```cpp
void(*function)() = Print; //很少用，一般用auto关键字
```

2.函数指针的使用

> 无参数的函数指针

```cpp
void Print() {
    std::cout << "hello，world" << std::endl;
}
int main() {
    //void(*function)() = Print； 正常写法，但一般用auto就可以了
    auto function = Print();    //ERROR!，auto无法识别void类型
    auto function = Print;  //OK!，去掉括号就不是在调用这个函数，而是在获取函数指针，得到了这个函数的地址。就像是带了&取地址符号一样"auto function = &Print;""(隐式转换)。
    function();//调用函数
    //这里函数指针其实也用到了解引用（*），这里是发生了隐式的转化，使得代码看起来更加简洁明了！
}
//输出：
hello,world
```

> 对于有参数的函数指针，在使用的时候传上参数即可

```cpp
void Print(int a) {
    std::cout << a << std::endl;
}
int main() {
    auto temp = Print;  //正常应该是 void(*temp)(int) = Print,太过于麻烦，用auto即可
    temp(1);    //在用函数指针的时候也传参数进去就可以正常使用了
}
```

> 也可以用typedef或者using来使用函数指针

```cpp
#include<iostream>

void HelloWorld()
{
    std::cout << "Hello World!" << std::endl;
}
int main()
{
    typedef void(*HelloWorldFunction)(); 

    HelloWorldFunction function = HelloWorld;
    function();

    std::cin.get();
}
```

为什么要首先使用函数指针

> 如果需要**将一个函数作为另一个函数的形参**，那么就要需要函数指针 .

```cpp
void Print(int val) {
    std::cout << val << std::endl;
}

//下面就将一个函数作为形参传入另一个函数里了
void ForEach(const std::vector<int>& values, void(*function)(int)) {
    for (int temp : values) {
        function(temp); //就可以在当前函数里用其他函数了
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    ForEach(values, Print); //这里就是传入了一个函数指针进去！！！！
}
```

**优化：lambda**

> **lambda本质上是一个普通的函数**，只是它不像普通函数这样声明，它是我们的代码**在过程中生成的，用完即弃的函数**，不算一个真正的函数，是**匿名函数** 。
> 格式：`[] ({形参表}) {函数内容}`

```cpp
void ForEach(const std::vector<int>& values, void(*function)(int)) {
    for (int temp : values) {
        function(temp);     //正常调用lambda函数
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    ForEach(values, [](int val){ std::cout << val << std::endl; });     //如此简单的事就交给lambda来解决就好了
}
```

## C++的lambda

1.lambda本质上是一个**匿名函数**。 用这种方式创建函数不需要实际创建一个函数 ，它就像一个**快速的一次性函数** 。 lambda更像是一种变量，在实际编译的代码中作为一个符号存在，而不是像正式的函数那样。

2.使用场景：

> 在我们会设置函数指针指向函数的任何地方，我们都可以将它设置为lambda

3.lambda表达式的写法(使用格式)：`[]( {参数表} ){ 函数体 }`

> 中括号**表示的是**捕获，作用是**如何传递变量** lambda使用**外部（相对）**的变量时，就要**使用捕获**。

如果使用捕获,则：

- 添加头文件： `#include <functional>`
- 修改相应的函数签名 `std::function <void(int)> func`替代 `void(*func)(int)`
- 捕获[]使用方式：

> `[=]`，则是将所有变量**值传递**到lambda中
> `[&]`，则是将所有变量**引用传递**到lambda中
> `[a]`是将变量a通过值传递，如果是`[&a]`就是将变量a引用传递
> 它可以有0个或者多个捕获

```cpp
//If the capture-default is `&`, subsequent simple captures must not begin with `&`.
struct S2 { void f(int i); };
void S2::f(int i)
{
    [&]{};          // OK: by-reference capture default
    [&, i]{};       // OK: by-reference capture, except i is captured by copy
    [&, &i] {};     // Error: by-reference capture when by-reference is the default
    [&, this] {};   // OK, equivalent to [&]
    [&, this, i]{}; // OK, equivalent to [&, i]
}


//If the capture-default is `=`, subsequent simple captures must begin with `&` or be `*this` (since C++17) or `this` (since C++20). 
struct S2 { void f(int i); };
void S2::f(int i)
{
    [=]{};        // OK: by-copy capture default
    [=, &i]{};    // OK: by-copy capture, except i is captured by reference
    [=, *this]{}; // until C++17: Error: invalid syntax
                  // since C++17: OK: captures the enclosing S2 by copy
    [=, this] {}; // until C++20: Error: this when = is the default
                  // since C++20: OK, same as [=]
}

详情参考：https://en.cppreference.com/w/cpp/language/lambda
```

事例：

```cpp
#include <iostream>
#include <vector>
#include <functional>
void ForEach(const std::vector<int>& values, void(*function)(int)) {
    for (int temp : values) {
        function(temp);     //正常调用lambda函数
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    //函数指针的地方都可以用auto来简化操作，lambda亦是
    //这样子来定义lambda表达式会更加清晰明了
    auto lambda = [](int val){ std::cout << val << std::endl; }
    ForEach(values, lambda);    
}
-------------------------------------------------
//lambda可以使用外部（相对）的变量，而[]就是表示打算如何传递变量
#include <functional>   //要用捕获就必须要用C++新的函数指针！
//新的函数指针的签名有所不同！
void ForEach(const std::vector<int>& values, const std::function<void(int)>& func) {
    for (int temp : values) {
        func(temp);     
    }
}

int main() {
    std::vector<int> valus = { 1, 2, 3, 4, 5 };
    //注意这里的捕获必须要和C++新带的函数指针关联起来！！！
    int a = 5;  //如果lambda需要外部的a向量
    //则在捕获中写入a就好了
    auto lambda = [a](int val){ std::cout << a << std::endl; }
    ForEach(values, lambda);    
}
```

我们有一个可选的修饰符**mutable**，它**允许lambda函数体**修改通过拷贝传递捕获的参数。若我们在lambda中给a赋值会报错，需要写上mutable 。

```cpp
int a = 5;
auto lambda = [=](int value) mutable { a = 5; std::cout << "Value: " << value << a << std::endl; };
```

另一个使用lambda的场景**`find_if`**

> 我们还可以写一个lambda接受vector的整数元素，遍历这个vector找到比3大的整数，然后返回它的迭代器，也就是满足条件的第一个元素。
> **`find_if`**是一个搜索类的函数，区别于`find`的是：**它可以接受一个函数指针来定义搜索的规则，返回满足这个规则的第一个元素的迭代器**。这个情况就很适合lambda表达式的出场了

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> values = { 1, 5, 3, 4, 2 };
    //下面就用到了lambda作为函数指针构成了find_it的规则
    auto it = std::find_if(values.begin(), values.end(), [](int value) { return value > 3; });  //返回第一个大于3的元素的迭代器 
    std::cout << *it << std::endl;  //将其输出
}
```

## 为什么我不使用using namespace std

1.不容易分辨各类函数的来源

> 比如我在一个自己的库中定义了一个vector，而标准库里又有一个vector，那么如果用了using namespace std 后，所用的vector到底是哪里的vector呢？

```cpp
std::vector<int>vec1;   //good
DiyClass::vector<int>vec2   //good

using namespace std;
using namespace DiyClass    //万一有其他人用了DiyClass的命名空间
vector<int>vec3 //便会有歧义，完全不知道到底是哪里的vector
```

2.一定**不要**在**头文件内**使用`using namespace std`

> 如果别人用了你的头文件，就会把这些命名空间用在了你原本没有打算用的地方，会导致莫名其妙的产生bug，如果有大型项目，追踪起来会很困难。 如果公司有自己的模板库，然后里面有很多重名的类型或者函数，就容易弄混；

3.可以就在一些小作用域里用，**但能不用就不用！养成良好的代码书写习惯！**

## C++的命名空间

1.命名空间是C++独有，C是没有的，故写C时会有命名冲突的风险。

2.**类本身就是名称空间** 。

> 类外使用一个类内的成员需要加`::`

3.命名空间（名称空间）的主要目的是**避免命名冲突**，便于管理各类命名函数。使用名称空间的原因，是因为我们希望**能够在不同的上下文中调用相同的符号**。

```cpp
#include <iostream>
#include <string>
#include <algorithm>
namespace apple {
    void print(const char *text) {
        std::cout << text << std::endl;
    }
}

namespace orange {
    void print(const char *text) {
        std::string temp = text;
        std::reverse(temp);
        std::cout << temp << std::endl;
    }
}
int main() {
    //using namespace apple::print; //单独引出一个print函数
    //using namespace apple;//引出apple名称空间的所有成员

    apple::print("hello");  //输出正常text
    orange::print("world"); //输出反转的text
}
```

拓展:详情请参考原文链接：https://zhuanlan.zhihu.com/p/441602923

大型程序往往会使用多个独立开发的库，这些库会定义大量的全局名字，如类、函数和模板等，不可避免会出现某些名字相互冲突的情况。命名空间`namespace`分割了全局命名空间，其中每个命名空间是一个作用域。

```cpp
namespace foo {
    class Bar { /*...*/ };
}  // 命名空间结束后无需分号
```

**命名空间定义**

**1. 每个命名空间都是一个作用域**

> 同其他作用域类似，命名空间中的每个名字都必须表示该空间内的唯一实体。因为不同命名空间的作用域不同，所以在不同命名空间内可以有相同名字的成员。

**2. 命名空间可以不连续**

> 命名空间的定义可以不连续的特性使得我们可以将几个独立的接口和实现文件组成一个命名空间，定义多个类型不相关的命名空间也应该使用单独的文件分别表示每个类型。

**3. 模板特例化**

何为模板特例化请参考：[http://t.csdn.cn/hpQOF](https://link.zhihu.com/?target=http%3A//t.csdn.cn/hpQOF)

> 模板特例化必须定义在原始模板所属的命名空间中，和其他命名空间名字类似，只要我们在命名空间中声明了特例化，就能在命名空间外部定义它了：

```cpp
// 我们必须将模板特例化声明成std的成员
namespace std {
    template <> struct hash<Foo>;
}

// 在std中添加了模板特例化的声明后，我们就可以在命名空间std的外部定义它了
template<> struct std::hash<Foo> {
    size_t operator()(const Foo& f) const {
        return hash<string>()(f.str) ^
            hash<double>()(f.d);
    }
};
```

**4. 全局命名空间**

全局作用域中定义的名字（即在所有类、函数以及命名空间之外定义的名字）也就是定义在全局命名空间`global namespace`中。全局作用域是隐式的，所以它并没有名字，下面的形式表示全局命名空间中一个成员：

```cpp
::member_name
```

**5. 嵌套的命名空间**

```cpp
namespace foo {
    namespace bar {
        class Cat { /*...*/ };
    }
}

// 调用方式
foo::bar::Cat
```

**6. 内联命名空间**

C++11新标准引入了一种新的嵌套命名空间，称为内联命名空间`inline namespace`。内联命名空间可以被外层命名空间直接使用。定义内联命名空间的方式是在关键字`namespace`前添加关键字`inline`：

```cpp
// inline必须出现在命名空间第一次出现的地方
inline namespace FifthEd {
    // ...
}
// 后续再打开命名空间的时候可以写inline也可以不写
namespace FifthEd {  // 隐式内敛
    // ...
}
```

当应用程序的代码在一次发布和另一次发布之间发生改变时，常使用内联命名空间。例如我们把第五版`FifthEd`的所有代码放在一个内联命名空间中，而之前版本的代码都放在一个非内联命名空间中：

```cpp
namespace FourthEd {
    // 第4版用到的其他代码
    class Cat { /*...*/ };
}

// 命名空间cplusplus_primer将同时使用这两个命名空间
namespace foo {
#include "FifthEd.h"
#include "FoutthEd.h"
}
```

因为`FifthEd`是内联的，所以形如`foo::`的代码可以直接获得`FifthEd`的成员，如果我们想用到早期版本的代码，则必须像其他嵌套的命名空间一样加上完整的外层命名空间名字：

```cpp
foo::FourthEd::Cat
```

**7. 未命名的命名空间**

关键字`namespace`后紧跟花括号括起来的一系列声明语句是未命名的命名空间`unnamed namespace`。未命名的命名空间中定义的变量具有静态生命周期：它们在第一次使用前被创建，直到程序结束时才销毁。

> *Tips：每个文件定义自己的未命名的命名空间，如果两个文件都含有未命名的命名空间，则这两个空间互相无关。在这两个未命名的命名空间里面可以定义相同的名字，并且这些定义表示的是不同实体。如果一个头文件定义了未命名的命名空间，则该命名空间中定义的名字将在每个包含了该头文件的文件中对应不同实体。*

和其他命名空间不同，未命名的命名空间仅在特定的文件内部有效，其作用范围不会横跨多个不同的文件。未命名的命名空间中定义的名字的作用域与该命名空间所在的作用域相同，如果未命名的命名空间定义在文件的最外层作用域中，则该命名空间一定要与全局作用域中的名字有所区别：

```cpp
// i的全局声明
int i;
// i在未命名的命名空间中的声明
namespace {
    int i;  
}
// 二义性错误: i的定义既出现在全局作用域中, 又出现在未嵌套的未命名的命名空间中
i = 10;
```

> *未命名的命名空间取代文件中的静态声明：*
> *在标准C++引入命名空间的概念之前，程序需要将名字声明成`static`的以使其对于整个文件有效。在文件中进行静态声明的做法是从C语言继承而来的。在C语言中，声明为`static`的全局实体在其所在的文件外不可见。* ***在文件中进行静态声明的做法已经被C++标准取消了，现在的做法是使用未命名的命名空间。***

## C++的线程

1.使用多线程，首先要添加头文件`#include <thread>`。

2.在**Linux平台**下编译时需要加上**"-lpthread"链接库**

3.创建一个线程对象：**`std::thread objName (一个函数指针以及其他可选的任何参数)`**

4.等待一个线程完成它的工作的方法 :`worker.join()`

> **这里的线程名字是worker，换其他的也可以,自己决定的）** 调用`join`的目的是：**在主线程上等待 工作线程 完成所有的执行之后，再继续执行主线程**

```cpp
//这个代码案例相当无用，只是为了展示多线程的工作而展示的。
#include <iostream>
#include <thread>
void DoWork() {
    std::cout << "hello" << std::endl;
}
int main() {
    //DoWork即是我们想要在另一个执行线程中发生的事情
    std::thread worker(DoWork); //这里传入的是函数指针！！！函数作为形参都是传函数指针！！！
    //一旦写完这段代码，它就会立即启动那个线程，一直运行直到我们等待他退出
    worker.join();  //join函数本质上，是要等待这个线程加入进来（而线程加入又是另一个复杂的话题了）

    //因为cin.get()是join语句的下一行代码，所以它不会运行，直到DoWork函数中的所有内容完成！
    std::cin.get();
}
#include <iostream>
#include <thread>

static bool is_Finished = false;
void DoWork() {
    using namespace std::literals::chrono_literals; //等待时间的操作可以先using一个命名空间，为 1s 提供作用域
    while (is_Finished) {
        std::cout << "hello" << std::endl;
        std::this_thread::sleep_for(1s);    //等待一秒
    }
}

int main() {
    std::thread worker(DoWork); //开启多线程操作

    std::cin.get(); //此时工作线程在疯狂循环打印，而主线程此时被cin.get()阻塞
    is_Finished = true;// 让worker线程终止的条件，如果按下回车，则会修改该值，间接影响到另一个线程的工作。

    worker.join();  //join:等待工作线程结束后，才会执行接下来的操作

    std::cin.get();
}
```

如果是正常情况，`DoWork`应该会一直循环下去，但因为这里是多线程，所以可以在另一个线程中修改工作线程的变量，来停止该线程的循环。 **多线程对于加速程序是十分有用的，线程的主要目的就是优化。**

## C++的计时

**作用**:

计时的使用很重要。在逐渐开始集成更多复杂的特性时，如果编写性能良好的代码时，需要用到计时来看到差异。

**利用chrono类计时**：

1.包含头文件**`#include`** 2.获取当前时间：

```cpp
std::chrono::time_point<std::chrono::steady_clock> start = std::chrono::high_resolution_clock::now();
//或者，使用auto关键字
auto  start = std::chrono::high_resolution_clock::now();
auto  end = std::chrono::high_resolution_clock::now();
----------------------------------------------------------
//实例
#include <iostream>
#include <chrono>
#include <thread>

int main() {
    //literals：文字
    using namespace std::literals::chrono_literals; //有了这个，才能用下面1s中的's'
    auto start = std::chrono::high_resolution_clock::now(); //记录当前时间
    std::this_thread::sleep_for(1s);    //休眠1s，实际会比1s大。函数本身有开销。
    auto end = std::chrono::high_resolution_clock::now();   //记录当前时间
    std::chrono::duration<float> duration = end - start;    //也可以写成 auto duration = end - start; 
    std::cout << duration.count() << "s" << std::endl;
    return 0;
}
```

3.获得时间差：

```cpp
std::chrono::duration<float> duration = end - start;
//或者
auto duration = end - start;
```

> 注意：在**自定义计时器类的构造函数、析构函数**中，**不要使用auto关键字**，应该在计时器类的构造函数、析构函数**前**定义start、end、duration变量。

```cpp
struct Timer   //写一个计时器类。
{
    std::chrono::time_point<std::chrono::steady_clock> start, end;
    std::chrono::duration<float> duration;

    Timer()
    {
        start = std::chrono::steady_clock::now(); //如果使用auto关键字会出现警告
    }

    ~Timer()
    {
        end = std::chrono::steady_clock::now();
        duration = end - start;

        float ms = duration.count() * 1000;
        std::cout << "Timer took " << ms << " ms" << std::endl;
    }
};
void Function()
{
    Timer timer;
    for (int i = 0; i < 100; i++)
        std::cout << "Hello\n"; //相比于std::endl更快
}

int main()
{
    Function();
}
```

## C++多维数组

数组优化的一个方法：**把二维数组转化成一维数组来进行存储。**

```cpp
//代码参考来源：https://github.com/UrsoCN/NotesofCherno/blob/main/Cherno64.cpp
#include <iostream>
#include <array>

int main()
{

    // 要知道，这样处理数组的数组，会造成内存碎片的问题
    // 我们创建了5个单独的缓冲区，每个缓冲区有5个整数，他们会被分配到内存的随机(空闲)位置
    // 在大量调用时，很可能造成cache miss，损失性能

    int *array = new int[5];
    int **a2d = new int *[5]; // 5个int指针
    for (int i = 0; i < 5; i++)
        a2d[i] = new int[5]; // allocate the memory

    for (int y = 0; y < 5; y++)
    {
        for (int x = 0; x < 5; x++)
        {
            a2d[y][x] = 2;
        }
    }

    // int ***a3d = new int **[5]; // 5个int指针的指针   三维数组
    // for (int i = 0; i < 5; i++)
    // {
    //     a3d[i] = new int *[5];
    //     for (int j = 0; j < 5; j++)
    //     {
    //         // int **ptr = a3d[i];
    //         // ptr[j] = new int[5];
    //         a3d[i][j] = new int[5];
    //     }
    // }

    for (int i = 0; i < 5; i++) // 需要先释放真正的多维数组
        delete[] a2d[i];
    delete[] a2d;
    // 这只会释放5个指针的内存，而后面分配的内存由于丢失掉了这些指针，
    // 也无法释放了，这就造成了内存泄漏

    int *array = new int[6 * 5];  //二维
    // for (int i = 0; i < 6 * 5; i++)
    // {
    //     array[i] = 2;
    // }
    for (int y = 0; y < 5; y++)   //数组优化，将二维数组转化为一维数组
    {
        for (int x = 0; x < 6; x++)
        {
            array[y * 5 + x] = 2;
        }
    }

    std::cin.get();
}
```

> B评论区的一个讨论问题：

```cpp
#include<iostream>
#include<chrono>

struct Timer   //写一个计时器类。
{
    std::chrono::time_point<std::chrono::steady_clock> start, end;
    std::chrono::duration<float> duration;

    Timer()
    {
        start = std::chrono::steady_clock::now(); //如果使用auto关键字会出现警告
    }

    ~Timer()
    {
        end = std::chrono::steady_clock::now();
        duration = end - start;

        float ms = duration.count() * 1000;
        std::cout << "Timer took " << ms << " ms" << std::endl;
    }
};
struct Rgb
{
    int r;
    int g;
    int b;
};

#define M 8000
#define N 5000

void draw()
{
    Timer timer;
    Rgb* a = new Rgb[M * N];
    for (int i = 0; i < M; i++)
    {
        for (int j = 0; j < N; j++)
        {
            a[i + j * M] = { 1,2,3 };
        }
    }
    //delete[] a;
}

//void draw()
//{
//  Timer timer;
//  Rgb* a = new Rgb[M * N];
//  for (int j = 0; j < N; j++)
//  {
//      for (int i = 0; i < M; i++)
//      {
//          a[j + i * N] = { 1,2,3 };
//      }
//  }
//  //delete[] a;
//}

void draw2()
{
    Timer timer;
    Rgb** a = new Rgb * [M];
    for (int i = 0; i < M; i++)
    {
        a[i] = new Rgb[N];
        for (int j = 0; j < N; j++)
        {
            a[i][j] = { 1,2,3 };
        }
        //delete[] a[i];   //这一句很神奇，加上后在release模式下，速度快5倍
    }
    //delete[] a;
}

int main()
{
    draw();
    draw2();
}
```

> 结论与Cherno的完全相反，二维数组比一维在debug与release下，均快1倍，如果在二维数组方式下，加上一句delete【】，再快将近5倍。
> 应该是你draw里面赋值的时候有问题。你这个两层循环内层是j,但j又是列指标，所以相当于本来完全连续的赋值变成每次赋值都要跑隔M的地方才能赋所以会变得很慢。
> 这里分配必然是慢的，因为是间隔分配，了解内存分配都知道越分散性能越差。Cherno说的快应该是读取的时候，读取的时候因为少了间接性（多层指针指向），读取性能要比多维高很多，修改性能应该也高很多。 另这里不应该用 【i + j * M】 而是应该用 【j + i * N】这样性能也会好很多，因为这是连续分配。
> release模式会优化代码，不一定会执行全部。 另外按升序遍历，索引应该是i*N+j，因为j走一遍，i才加1。连续的内存才能容易cache hit
> 我把样本数据扩大到5000*5000 之后 , release 下一维明显更快 , 而 debug 模式下二维更快一点

## C++内置的排序函数

1.sort( vec.begin(), vec.end(), 谓语)

谓语可以设置排序的规则，谓语可以是内置函数，也可以是lambda表达式。

2.**默认**是从小到大排序

```cpp
#include<iostream>
#include<vector>
#include<algorithm>

int main()
{
    std::vector<int>  values = {3, 5, 1, 4, 2};
    std::sort(values.begin(), values.end());  
    for (int value : values)
    std::cout << value << std::endl; // 1 2 3 4 5
    std::cin.get();
}
```

3.使用内置函数，添加头文件functional，使用std::greater函数，则会按照从大到小顺序排列。

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<functional>

int main()
{
    std::vector<int>  values = {3, 5, 1, 4, 2};             
    std::sort(values.begin(), values.end(),std::greater<int>()); 
    for (int value : values)
    std::cout << value << std::endl; // 5 4 3 2 1
    std::cin.get();
}
```

4.使用 lambda 进行灵活排序

```cpp
std::sort(values.begin(), values.end(), [](int a, int b)
    {
            return a < b;
    });
```

> 对于已定的传入参数的顺序`[](int a, int b)`，函数体中如果参数a在前面，则返回true，如果参数a在后面则返回false

```text
a < b //返回true，a排在前面。此时为升序排列（如果a小于b，那么a就排在b的前面）
a > b //返回true, a排在前面，此时为降序排列（如果a大于b，那么a就排在b的前面）
#include<iostream>
#include<vector>
#include<algorithm>
#include<functional>

int main()
{
    std::vector<int>  values = {3, 5, 1, 4, 2};          

    std::sort(values.begin(), values.end(), [](int a, int b)
    {
            return a < b;  // 如果a小于b，那么a就排在b的前面。 1 2 3 4 5
    });

    for (int value : values)
    std::cout << value << std::endl;

    std::cin.get();
}
```

5.如果把1排到最后

> 如果a==1，则把它移到后面去，即返回false，不希望它在b前。 如果b==1，我们希望a在前面，要返回true。

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
#include<functional>

int main()
{
    std::vector<int>  values = {3, 5, 1, 4, 2};          
    std::sort(values.begin(), values.end(), [](int a, int b)
    {
            if (a == 1)
                return false;
            if(b == 1)
                return true;
            return a < b;   //2 3 4 5 1
    });
    for (int value : values)
    std::cout << value << std::endl;
    std::cin.get();
}
```

## C++的类型双关(type punning)

1.将同一块内存的东西通过不同type的指针给取出来

> 把一个int型的内存，换成double去解释，当然**这样做很糟糕**，因为添加了四字节不属于原本自己的内存，只是作为演示。 原始方法：**（取地址，换成对应类型的指针，再解引用）**

```cpp
#include <iostream>
int main()
{
    int a = 50;
    double value = *(double*)&a;
    std::cout << value << std::endl;

    std::cin.get();
}
//可以用引用，这样就可以避免拷贝成一个新的变量：（只是演示，这样做很糟糕）
#include <iostream>
int main()
{
    int a = 50;
    double& value = *(double*)&a;
    std::cout << value << std::endl;

    std::cin.get();
}
```

2.把一个结构体转换成数组进行操作（? 还不理解）

```cpp
#include <iostream>
struct Entity
{
    int x, y;
};

int main()
{
    Entity e = {5, 8};
    int *position = (int *)&e;
    std::cout << position[0] << ", " << position[1] << std::endl;

    int y = *(int *)((char *)&e + 4);
    std::cout << y << std::endl;
}
```

## C++的联合体( union )

1.`union { };`，注意结尾有**分号**。

2.通常union是匿名使用的，但是匿名union不能含有成员函数

3.在可以使用**类型双关**的时候，使用union时，可读性更强 。

4.union的特点是**共用内存** 。可以像使用结构体或者类一样使用它们，也可以给它添加静态函数或者普通函数、方法等待。然而**不能使用虚方法**，还有其他一些限制。

```cpp
#include <iostream>
int main() {  
    union {  //匿名使用，不写名字
        float a;
        int b;
    };  
    a = 2.0f;  //共享内存，a被赋值了一个浮点数，整形的b也被复制了一个浮点数
    std::cout << a << '，' << b << std::endl;
    //输出： 2，107165123
    //原因：int b取了组成浮点数的内存，然后把它解释成一个整型（类型双关）
}
```

较实用的一个例子：

```cpp
#include <iostream>
struct Vector2
{
    float x, y;
};

struct Vector4
{
    union // 不写名称，作为匿名使用
    {
        struct //第一个Union成员
        {
            float x, y, z, w;
        };
        struct // 第二个Union成员，与第一个成员共享内存
        {
            Vector2 a, b;//a和x，y的内存共享，b和z，w的内存共享
        };
    };
};

void PrintVector2(const Vector2 &vector)
{
    std::cout << vector.x << ", " << vector.y << std::endl;
}

int main()
{
    Vector4 vector = {1.0f, 2.0f, 3.0f, 4.0f};
    PrintVector2(vector.a);
    PrintVector2(vector.b);
    vector.z = 500;
    std::cout << "-----------------------" << std::endl;
    PrintVector2(vector.a);
    PrintVector2(vector.b);
}
//输出：
1，2
3，4
-----------------------
1，2
500，4
```

> 引自评论： union里的成员会共享内存，分配的大小是按最大成员的sizeof, 视频里有两个成员，也就是那两个结构体，改变其中一个另外一个里面对应的也会改变. 如果是这两个成员是结构体struct{ int a,b} 和 int k , 如果k=2 ; 对应 a也=2 ，b不变； union我觉得在这种情况下很好用，就是用不同的结构表示同样的数据 ，那么你可以按照获取和修改他们的方式来定义你的 union结构 很方便

## C++的虚析构函数

1.如果用基类指针来引用派生类对象，那么基类的析构函数必须是 virtual 的，否则 C++ 只会调用基类的析构函数，不会调用派生类的析构函数。

2.继承时，要养成的一个好习惯就是，**基类析构函数中，加上virtual。**

> 为什么要调用派生类析构函数？
> 若派生类有一个成员int数组在堆上分配东西，在构造函数中分配，在析构函数中删除。运行当前代码发现没有调用那个派生析构函数，但是它调用了派生类的构造函数。我们在构造函数中分配了一些内存，但是永远不会调用派生析构函数delete释放内存，因为析构函数没有被调用，永远不会删除堆分配数组，这就是所谓的内存泄漏。

```cpp
#include <iostream>

class Base
{
public:
    Base() { std::cout << "Base Constructor\n"; }
    virtual ~Base() { std::cout << "Base Destructor\n"; }
};

class Derived : public Base
{
public:
    Derived()
    {
        m_Array = new int[5];
        std::cout << "Derived Constructor\n";
    }
    ~Derived()
    {
        delete[] m_Array;
        std::cout << "Derived Destructor\n";
    }

private:
    int *m_Array;
};

int main()
{
    Base *base = new Base();
    delete base;
    std::cout << "-----------------" << std::endl;
    Derived *derived = new Derived();
    delete derived;
    std::cout << "-----------------" << std::endl;
    Base *poly = new Derived();
    delete poly; // 基类析构函数中如果不加virtual，则此处会造成内存泄漏
    // Base Constructor
    // Base Destructor
    // -----------------
    // Base Constructor
    // Derived Constructor
    // Derived Destructor
    // Base Destructor
    // -----------------
    // Base Constructor
    // Derived Constructor
    // Derived Destructor //基类析构函数中如果不加virtual，子类的虚构函数不会被调用
    // Base Destructor
}
```

> 引自B站评论区：
> 此处这位外国友人说错了，定义基类的虚析构并不是什么相加，而是：**基类中只要定义了虚析构（且只能在基类中定义虚析构，子类析构才是虚析构，如果在二级子类中定义虚析构，编译器不认，且virtual失效），在编译器角度来讲，那么由此基类派生出的所有子类地析构均为对基类的虚析构的重写，当多态发生时，用父类引用，引用子类实例时，此时的虚指针保存的子类虚表的地址，该函数指针数组中的第一元素永远留给虚析构函数指针。所以当delete 父类引用时，即第一个调用子类虚表中的子类重写的虚析构函数此为第一阶段。然后进入第二阶段：（二阶段纯为内存释放而触发的逐级析构与虚析构就没有半毛钱关系了）而当子类发生析构时，子类内存开始释放，因内存包涵关系，触发父类析构执行，层层向上递进，至到子类所包涵的所有内存释放完成。**

## C++的类型转换

> cast 分为 `static_cast` `dynamic_cast` `reinterpret_cast` `const_cast`

### static_cast

使⽤： 

1. ⽤于基本数据类型之间的转换，如把int转换成char。  
2. 把任何类型的表达式转换成void类型。
3. 

static_cast用于进行比较“自然”和低风险的转换，如整型和浮点型、字符型之间的互相转换,不能用于指针类型的强制转换

> 任何具有明确定义的类型转换，只要不包含底层const，都可以使用static_cast。

```cpp
double dPi = 3.1415926;
int num = static_cast<int>(dPi);  //num的值为3
double d = 1.1;
void *p = &d;
double *dp = static_cast<double *>(p);
```



- 用于非多态类型的转换
- 不执行运行时类型检查（转换安全性不如 dynamic_cast）
- 通常用于转换数值数据类型（如 float -> int）
- 可以在整个类层次结构中移动指针，子类转化为父类安全（向上转换），父类转化为子类不安全（因为子类可能有不在父类的字段或方法）

> 向上转换是一种隐式转换。

### reinterpret_cast

reinterpret_cast 用于进行各种不同类型的指针之间强制转换。

> 通常为运算对象的位模式提供较低层次上的重新解释。危险，不推荐。

```cpp
int *ip;
char *pc = reinterpret_cast<char *>(ip);
```



- 用于位的简单重新解释
- 滥用 reinterpret_cast 运算符可能很容易带来风险。 除非所需转换本身是低级别的，否则应使用其他强制转换运算符之一。
- 允许将任何指针转换为任何其他指针类型（如 `char*` 到 `int*` 或 `One_class*` 到 `Unrelated_class*` 之类的转换，但其本身并不安全）
- 也允许将任何整数类型转换为任何指针类型以及反向转换。
- reinterpret_cast 运算符不能丢掉 const、volatile 或 __unaligned 特性。
- reinterpret_cast 的一个实际用途是在哈希函数中，即，通过让两个不同的值几乎不以相同的索引结尾的方式将值映射到索引。

### const_cast

const_cast 添加或者移除const性质

> 用于改变运算对象的**底层const**。常用于有函数重载的上下文中。
> **顶层const：**表示**对象是常量**。举例int *const p1 = &i; //指针p1本身是一个常量，不能改变p1的值，p1是顶层const。
> **底层const：**与指针和引用等复合类型部分有关。举例：const int *p2 = &ci; //指针所指的对象是一个常量，允许改变p2的值，但不允许通过p2改变ci的值，p2是底层const

```cpp
const string &shorterString(const string &s1, const string &s2)
{
    return s1.size() <= s2.size() ? s1 : s2;
}

//上面函数返回的是常量string引用，当需要返回一个非常量string引用时，可以增加下面这个函数
string &shorterString(string &s1, string &s2) //函数重载
{
    auto &r = shorterString(const_cast<const string &>(s1), 
                            const_cast<const string &>(s2));
    return const_cast<string &>(r);
}
```

### dynamic_cast

在进⾏下⾏转换时，dynamic_cast具有类型检查（信息在虚函数中）的功能，⽐static_cast 更安全。 

转换后必须是类的指针、引⽤或者void*，基类要有虚函数，可以交叉转换。 

dynamic本⾝只能⽤于存在虚函数的⽗⼦关系的强制类型转换；对于指针，转换失败则返回 nullptr，对于引⽤，转换失败会抛出异常。

dynamic_cast 不检查转换安全性，仅运行时检查，如果不能转换，返回NULL。

> 支持运行时类型识别(run-time type identification,RTTI)。
> 适用于以下情况：我们想使用基类对象的指针或引用执行某个派生类操作并且该操作不是虚函数。一般来说，只要有可能我们应该尽量使用虚函数，使用RTTI运算符有潜在风险，程序员必须清楚知道转换的目标类型并且必须检查类型转换是否被成功执行。

```cpp
//https://github.com/UrsoCN/NotesofCherno/blob/main/Cherno69.cpp
#include <iostream>
class Base
{
public:
    Base() { std::cout << "Base Constructor\n"; }
    virtual ~Base() { std::cout << "Base Destructor\n"; }
};

class Derived : public Base
{
public:
    Derived()
    {
        m_Array = new int[5];
        std::cout << "Derived Constructor\n";
    }
    ~Derived()
    {
        delete[] m_Array;
        std::cout << "Derived Destructor\n";
    }

private:
    int *m_Array;
};

class AnotherClass : public Base
{
public:
    AnotherClass(){};
    ~AnotherClass(){};
};

int main()
{
    // double value = 5.25;
    // // int a = value;
    // // int a = (int)value;
    // double a = (int)value + 5.3; // 10.3 // C style cast here

    // double s = static_cast<int>(value) + 5.3; // C++ style cast here

    // std::cout << a << std::endl;
    // std::cout << s << std::endl;

    Derived *derived = new Derived();

    Base *base = derived;

    // AnotherClass *ac = static_cast<AnotherClass*>(base);  //NULL
    Derived *ac = dynamic_cast<Derived *>(base);

    delete derived;
}
```

#### bad_cast

- 由于强制转换为引用类型失败，dynamic_cast 运算符引发 bad_cast 异常。

bad_cast 使用

```cpp
try {  
    Circle& ref_circle = dynamic_cast<Circle&>(ref_shape);   
}  
catch (bad_cast b) {  
    cout << "Caught: " << b.what();  
} 
```

## 现代C++中的安全以及如何教授

用于生产环境使用智能指针，用于学习和了解工作积累，使用原始指针，当然，如果你需要定制的话，也可以使用自己写的智能指针

## C++预编译头文件

1.作用：

> 为了解决一个项目中同一个头文件被反复编译的问题。使得写代码时不需要一遍又一遍的去`#include`那些常用的头文件，而且能**大大提高编译速度**

2.使用限制：预编译头文件中的内容最好都是**不需要反复更新修改的东西**。

> 每修改一次，预编译头文件都要重新编译一次，会导致变异速度降低。但像**C++标准库**，window的api这种不会大改的文件可以放到预编译头文件中，可以节省编译时间

3.缺点：

> 预编译头文件的使用会隐藏掉这个cpp文件的依赖。比如用了`#include <vector>`，就清楚的知道这个cpp文件中需要vector的依赖，而如果放到预编译头文件中，就会将该信息隐藏。

4..使用流程：

在Visual Studio中：[https://www.bilibili.com/video/BV1eu411f736?share_source=copy_web&vd_source=48739a103c73f618758b902392cb372e](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1eu411f736%3Fshare_source%3Dcopy_web%26vd_source%3D48739a103c73f618758b902392cb372e)

> 视频讲解更为详细。

在g++中：

> 首先确保main.cpp（主程序文件）、pch.cpp(包含预编译头文件的cpp文件)、pch.h（预编译头文件）在同一源文件目录下

注：pch.h文件的名字是自己命名的，改成其他名称也没问题。

```cpp
  g++ -std=c++11 pch.h //先编译pch头文件
  //time的作用是在控制台显示编译所需要的时间。
  time g++ -std=c++11 main.cpp  //然后编译主程序文件即可，编译速度大大提升。
```

## C++的dynamic_cast

1.dynamic_cast是专门用于沿继承层次结构进行的强制类型转换。并且**dynamic_cast只用于多态类类型**。

2.如果转换失败会返回NULL，使用时需要保证是多态，即**基类**里面**含有虚函数**。

3.dynamic_cast运算符，用于将基类的指针或引用安全地转换成派生类的指针或引用。

> 支持运行时类型识别(run-time type identification,RTTI)。
> 适用于以下情况：我们想使用基类对象的指针或引用执行某个派生类操作并且该操作不是虚函数。一般来说，只要有可能我们应该尽量使用虚函数，使用RTTI运算符有潜在风险，程序员必须清楚知道转换的目标类型并且必须检查类型转换是否被成功执行。

4.使用形式

其中，**type必须是一个类类型**，并且通常情况下该类型应该**含有虚函数**。

```cpp
dynamic cast<type*> (e) //e必须是一个有效的指针
dynamic cast<type&> (e) //e必须是一个左值
dynamic cast<type&&> (e) //e不能是左值
```

在上面的所有形式中，e的类型必须符合以下三个条件中的任意一个：

> 1)e的类型是目标type的**公有派生类** 2)e的类型是目标type的**公有基类** 3)e的类型就是**目标type的类型**。

如果符合，则类型转换可以成功。否则，转换失败。

5.如果一条dynamic_cast语句的转换目标是**指针类型**并且**失败**了，则**结果为0**。

```cpp
//假定Base类至少含有一个虚函数，Derived是Base的公有派生类。
//如果有一个指向Base的指针bp，则我们可以在运行时将它转换成指向Derived的指针。
if (Derived *dp = dynamic_cast<Derived *>bp) //在条件部分执行dynamic_cast操作可以确保类型转换和结果检查在同一条表达式中完成。
{
    //成功。使用dp指向的Derived对象
}
else
{
    //失败。使用bp指向的Base对象
}
```

6.如果转换目标是**引用类型**并且**失败**了，则dynamic_cast运算符将**抛出一个bad cast异常**。

> 引用类型的dynamic_cast与指针类型的dynamic_cast在表示错误发生的方式上略有不同。因为不存在所谓的空引用，所以对于引用类型来说无法使用与指针类型完全相同的错误报告策略。当对引用的类型转换失败时，程序抛出一个名为std:：bad cast的异常，该异常定义在typeinfo标准库头文件中。

```cpp
void f(const Base&b){
try{
    const Derived &d = dynamic cast<const Derived&>（b）；
    //使用b引用的Derived对象
    }catch(bad cast){
    //处理类型转换失败的情况
    }
}
```

**cherno的代码案例：**

```cpp
//代码参考：https://zhuanlan.zhihu.com/p/352420950
#include<iostream>
class Base
{
public:
    virtual void print(){}
};
class Player : public Base
{
};
class Enemy : public Base
{
};
int main()
{
    Player* player = new Player();
    Base* base = new Base();
    Base* actualEnemy = new Enemy();
    Base* actualPlayer = new Player();

    // 旧式转换
    Base* pb1 = player; // 从下往上，是隐式转换，安全
    Player*  bp1 = (Player*)base; // 从上往下，可以用显式转换，危险
    Enemy* pe1 = (Enemy*)player; // 平级转换，可以用显式转换，危险

    // dynamic_cast
    Base* pb2 = dynamic_cast<Base*>(player); // 从下往上，成功转换
    Player* bp2 = dynamic_cast<Player*>(base); // 从上往下，返回NULL
    if(bp2) { } // 可以判断是否转换成功
    Enemy* pe2 = dynamic_cast<Enemy*>(player); // 平级转换，返回NULL
    Player* aep = dynamic_cast<Player*>(actualEnemy); // 平级转换，返回NULL
    Player* app = dynamic_cast<Player*>(actualPlayer); // 虽然是从上往下，但是实际对象是player，所以成功转换
}
```

## C++的基准测试

1.编写一个计时器对代码测试性能。记住要**在release模式**去测试，这样才更有意义 。

2.该部分内容基本同"C++计时"一节（对应视频P63）

```cpp
#include <iostream>
#include <memory>
#include <chrono>   //计时工具
#include <array>
class Timer {
public:
    Timer() {
        m_StartTimePoint =  std::chrono::high_resolution_clock::now();
    }
    ~Timer() {
        Stop();
    }
    void Stop() {
        auto endTimePoint = std::chrono::high_resolution_clock::now();
        auto start = std::chrono::time_point_cast<std::chrono::microseconds>(m_StartTimePoint).time_since_epoch().count();
        //microseconds 将数据转换为微秒
        //time_since_epoch() 测量自时间起始点到现在的时长
        auto end = std::chrono::time_point_cast<std::chrono::microseconds>(endTimePoint).time_since_epoch().count();
        auto duration = end - start;
        double ms = duration * 0.001; ////转换为毫秒数
        std::cout << duration << "us(" << ms << "ms)\n";
    }
private:
    std::chrono::time_point<std::chrono::high_resolution_clock> m_StartTimePoint;
};

int main() 
{
    struct Vector2 {
        float x, y;
    };

    {
        std::array<std::shared_ptr<Vector2>, 1000> sharedPtrs;
        Timer timer;
        for (int i = 0; i < sharedPtrs.size(); i++) {
            sharedPtrs[i] = std::make_shared<Vector2>();
        }
    }

    {
        std::array<std::shared_ptr<Vector2>, 1000> sharedPtrs;
        Timer timer;
        for (int i = 0; i < sharedPtrs.size(); i++) {
            sharedPtrs[i] = std::shared_ptr<Vector2>(new Vector2());
        }
    }

    {
        Timer timer;
        std::array<std::unique_ptr<Vector2>, 1000> sharedPtrs;
        for (int i = 0; i < sharedPtrs.size(); i++) {
            sharedPtrs[i] = std::make_unique<Vector2>();
        }
    }
}
```

## C++的结构化绑定(Structured Binding)

1.结构化绑定struct binding是C++17的新特性，能让我们更好地处理多返回值。可以在将函数返回为tuple、pair、struct等结构时且赋值给另外变量的时候，**直接得到成员**，而不是结构。

> 在视频P52谈过如何处理多返回值，当时是用结构体去处理，而这个结构化绑定就是在这个的基础上拓展的一种新方法，特别是处理元组，对组（pairs）以及返回诸如此类的东西。

2.用**g++编译**时需要加上‘**-std=c++17’** or ‘-std=gnu++17’

实例：

**老方法**（tuple、pair）

> 结构体方法这里不再演示，具体见之前的笔记。

```cpp
#include <iostream>
#include <string>
#include <tuple>

// std::pair<std::string,int> CreatPerson() // 只能有两个变量
std::tuple<std::string, int> CreatPerson() // 可以理解为pair的扩展
{
    return {"Cherno", 24};
}

int main()
{
    //元组的数据获取易读性差，还不如像结构体一样直接XXX.age访问更加可读。
    // std::tuple<std::string, int> person = CreatPerson();
     auto person = CreatPerson(); //用auto关键字
     std::string& name = std::get<0>(person);
     int age = std::get<1>(person);

    //tie 可读性好一点
     std::string name;
     int age;
     std::tie(name, age) = CreatPerson();
}
```

**C++17新方法：结构化绑定处理多返回值**

```cpp
#include <iostream>
#include <string>
#include <tuple>


std::tuple<std::string, int> CreatPerson() 
{
    return {"Cherno", 24};
}

int main()
{
    auto[name, age] = CreatPerson(); //直接用name和age来储存返回值
    std::cout << name;
}
```

## C++如何处理optional数据(std::optional)

1.C++17 在 STL 中引入了`std::optional`，就像`std::variant`一样，`std::optional`是一个“**和类型(sum type)**”，也就是说，`std::optional`类型的变量要么是一个`T`类型的**变量**，要么是一个表示“什么都没有”的**状态**。

2.基本用法：

> 首先要包含`#include <optional>`

3.**has_value()**

> 我们可以通过has_value()来判断对应的**optional是否处于已经设置值**的状态, 代码如下所示:

```cpp
int main()
{
std::string text = /*...*/;
std::optional<unsigned> opt = firstEvenNumberIn(text);
if (opt.has_value())  //直接if(opt)即可，代码更简洁
{
 std::cout << "The first even number is "
           << opt.value()
           << ".\n";
}
}
```

4.访问optional对象中的数据

```text
1. opt.value()
2. (*opt)
3. value_or() //value_or()可以允许传入一个默认值, 如果optional为std::nullopt, 
              //则直接返回传入的默认值.（如果数据确实存在于std::optional中，
              //它将返回给我们那个字符串。如果不存在，它会返回我们传入的任何值）
```

`std::optional`是C++17的新东西，用于检测数据是否存在or是否是我们期盼的形式，用于处理那些可能存在，也可能不存在的数据or一种我们不确定的类型 。

> 比如在读取文件内容的时候，往往需要判断读取是否成功，常用的方法是传入一个引用变量或者判断返回的std::string是否为空，C++17引入了一个更好的方法，std::optional

老方法：传入一个引用变量或者判断返回的std::string是否为空

```cpp
#include <iostream>
#include <fstream>
#include <string>
std::string ReadFile(const std::string &fileapath, bool &outSuccess) {
    std::ifstream stream(filepath);
    //如果成功读取文件
    if (stream) {
        std::string result;
        getline(stream,result);
        stream.close();
        outSuccess = true;  //读取成功，修改bool
        return result;
    }
    outSuccess = false; //反之
}

int main() {
    bool flag;
    auto data = ReadFile("data.txt", flag);
    //如果文件有效，则接着操作
    if (flag) {

    }
}
```

**新方法：std::optional**

```cpp
// 用g++编译时需要加上‘-std=c++17’ or ‘-std=gnu++17’
// std::optional同样是C++17的新特性，可以用来处理可能存在、也可能不存在的数据
//data.txt在项目目录中存在，且其中的内容为"data!"
#include <iostream>
#include <fstream>
#include <optional>
#include <string>

std::optional<std::string> ReadFileAsString(const std::string& filepath)
{
    std::ifstream stream(filepath);
    if (stream)
    {
        std::string result;
        getline(stream, result);
        stream.close();
        return result;
    }
    return {};
    //如果文本存在的话，它会返回所有文本的字符串。如果不存在或者不能读取；则返回optional {}
}

int main()
{
     std::optional<std::string> data = ReadFileAsString("data.txt");
    //auto data = ReadFileAsString("data.txt"); //可用auto关键字
    if (data)
    {   
       // std::string& str = *data;
       // std::cout << "File read successfully!" << str<< std::endl;
        std::cout << "File read successfully!" << data.value() << std::endl;      
    }
    else
    {
        std::cout << "File could not be opened!" << std::endl;
    }

    std::cin.get();
}
//输出
File read successfully!"data!"
```

> 如果文件无法打开，或者文件的特定部分没有被设置或读取，也许我们有一个默认值，这很常见。此时就可以使用value_or()函数。其作用就是：如果数据确实存在于std::optional中，它将返回给我们那个字符串。如果不存在，它会返回我们传入的任何值。

删除data.txt,此时文件不存在打不开，则被设置为默认值

```cpp
#include <iostream>
#include <fstream>
#include <optional>
#include <string>

std::optional<std::string> ReadFileAsString(const std::string& filepath)
{
    std::ifstream stream(filepath);
    if (stream)
    {
        std::string result;
        //getline(stream, result);
        stream.close();
        return result;
    }

    return {}; //返回空
}

int main()
{
    std::optional<std::string>  data = ReadFileAsString("data.txt");

    std::string value = data.value_or("Not present");
    std::cout << value << std::endl;

    if (data)
    {
        std::cout << "File read successfully!" << std::endl;
    }
    else
    {
        std::cout << "File could not be opened!" << std::endl;
    }
}
//输出
Not present
File could not be opened!
```

## C++单一变量存放多种类型的数据(std::variant)

1.std::variant是C++17的新特性，可以让我们不用担心处理的确切数据类型 ,是一种可以容纳多种类型变量的结构 。

> 它和`option`很像，它的作用是让我们不用担心处理确切的数据类型，只有一个变量，之后我们在考虑它的具体类型
> 故我们做的就是指定一个叫`std::variant`的东西，然后列出它可能的数据类型

2.与union的区别

> 1)union 中的成员内存共享。union更有效率。 2)std::variant的大小是<>里面的大小之和 。**variant**更加**类型安全**，不会造成未定义行为，**所以应当去使用它,除非做的是底层优化，非常需要性能**。

3.简单的运用：

```cpp
std::variant<string, int> data; //列举出可能的类型
data = "hello";
// 索引的第一种方式：std::get，但是要与上一次赋值类型相同，不然会报错
cout << std::get<string>(data) <<endl;//print hello
data = 4;
cout << std::get<int>(data) <<endl;//print 4
cout << std::get<string>(data) <<endl;//编译通过，但是runtime会报错，显示std::bad_variant_access
data = false;//能编译通过
cout << std::get<bool>(data) <<endl;//这句编译失败
```

index()索引

```cpp
//std::variant的index函数
data.index();// 返回一个整数，代表data当前存储的数据的类型在<>里的序号，比如返回0代表存的是string, 返回1代表存的是int
```

get_if()

```cpp
// std::get的变种函数，get_if
auto p = std::get_if<std::string>(&data);//p是一个指针，如果data此时存的不是string类型的数据，则p为空指针，别忘了传的是地址
// 如果data存的数据是string类型的数据
if(auto p = std::get_if<string>(&data)){
    string& s = *p;
}
```

**cherno的代码：**

```cpp
//参考：https://zhuanlan.zhihu.com/p/352420950
#include<iostream>
#include<variant>
int main()
{
    std::variant<std::string,int> data; // <>里面的类型不能重复
    data = "ydc";
    // 索引的第一种方式：std::get，但是要与上一次赋值类型相同，不然会报错
    std::cout<<std::get<std::string>(data)<<std::endl;
    // 索引的第二种方式，std::get_if，传入地址，返回为指针
    if (auto value = std::get_if<std::string>(&data))
    {
        std::string& v = *value;
    }
    data = 2;
    std::cout<<std::get<int>(data)<<std::endl;
    std::cin.get();
}
```

## C++如何存储任意类型的数据(std::any)

1.也是C++17引入的可以存储多种类型变量的结构，其本质是一个union，但是不像std::variant那样需要列出类型。使用时要包含头文件`#include <any>`

2.对于小类型(small type)来说，any将它们存储为一个严格对齐的Union， 对于大类型，会用void*，动态分配内存 。

3.**评价：基本无用**。 当在一个变量里储存多个数据类型，用any的类型安全版本即可：`variant`

```cpp
#include <iostream>
#include <any>
// 这里的new的函数，是为了设置一个断点，通过编译器观察主函数中何处调用了new，看其堆栈。
void *operator new(size_t size)
{
    return malloc(size);
}

int main()
{
    std::any data;
    data = 2;
    data = "Cherno";
    data = std::string("Cherno");

    std::string& string = std::any_cast<std::string&>(data); //用any_cast指定转换的类型,如果这个时候any不是想要转换的类型，则会抛出一个类型转换的异常
    // 通过引用减少复制操作，以免影响性能
}
```

## 如何让C++运行得更快(std::async)

1.利用std::async，封装了异步编程的操作，提高了性能。

> 两个问题： 1、为什么不能传引用？ 线程函数的参数按值移动或复制。如果引用参数需要传递给线程函数，它必须被包装（例如使用std :: ref或std :: cref）
> 2、std::async为什么一定要返回值？ 如果没有返回值，那么在一次for循环之后，临时对象会被析构，而析构函数中需要等待线程结束，所以就和顺序执行一样，一个个的等下去 如果将返回值赋值给外部变量，那么生存期就在for循环之外，那么对象不会被析构，也就不需要等待线程结束。

具体实现原理还不明白，此处留个坑，以后学了再填。

相关参考资料：

cherno的视频讲解：[https://www.bilibili.com/video/BV1UR4y1j7YL?share_source=copy_web&vd_source=48739a103c73f618758b902392cb372e](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1UR4y1j7YL%3Fshare_source%3Dcopy_web%26vd_source%3D48739a103c73f618758b902392cb372e)

官方文档：[https://en.cppreference.com/w/cpp/thread/async](https://link.zhihu.com/?target=https%3A//en.cppreference.com/w/cpp/thread/async)



```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <numeric>
#include <future>
#include <string>
#include <mutex>
 
std::mutex m;
struct X {
    void foo(int i, const std::string& str) {
        std::lock_guard<std::mutex> lk(m);
        std::cout << str << ' ' << i << '\n';
    }
    void bar(const std::string& str) {
        std::lock_guard<std::mutex> lk(m);
        std::cout << str << '\n';
    }
    int operator()(int i) {
        std::lock_guard<std::mutex> lk(m);
        std::cout << i << '\n';
        return i + 10;
    }
};
 
template <typename RandomIt>
int parallel_sum(RandomIt beg, RandomIt end)
{
    auto len = end - beg;
    if (len < 1000)
        return std::accumulate(beg, end, 0);
 
    RandomIt mid = beg + len/2;
    auto handle = std::async(std::launch::async,
                             parallel_sum<RandomIt>, mid, end);
    int sum = parallel_sum(beg, mid);
    return sum + handle.get();
}
 
int main()
{
    std::vector<int> v(10000, 1);
    std::cout << "The sum is " << parallel_sum(v.begin(), v.end()) << '\n';
 
    X x;
    // 以默认策略调用 x.foo(42, "Hello") ：
    // 可能同时打印 "Hello 42" 或延迟执行
    auto a1 = std::async(&X::foo, &x, 42, "Hello");
    // 以 deferred 策略调用 x.bar("world!")
    // 调用 a2.get() 或 a2.wait() 时打印 "world!"
    auto a2 = std::async(std::launch::deferred, &X::bar, x, "world!");
    // 以 async 策略调用 X()(43) ：
    // 同时打印 "43"
    auto a3 = std::async(std::launch::async, X(), 43);
    a2.wait();                     // 打印 "world!"
    std::cout << a3.get() << '\n'; // 打印 "53"
} // 若 a1 在此点未完成，则 a1 的析构函数在此打印 "Hello 42"
```

### 注解

实现可以通过在默认运行策略中启用额外（实现定义的）位，扩展 **std::async** 第一重载的行为。

实现定义的运行策略的例子是同步策略（在 async 调用内立即执行）和任务策略（类似 async ，但不清理线程局域对象）。

若从 `std::async` 获得的 `std::future` 未被移动或绑定到引用，则在完整表达式结尾， [std::future](https://zh.cppreference.com/w/cpp/thread/future) 的析构函数将阻塞直至异步计算完成，实质上令如下代码同步：

```
std::async(std::launch::async, []{ f(); }); // 临时量的析构函数等待 f()
std::async(std::launch::async, []{ g(); }); // f() 完成前不开始
```

（注意，以调用 std::async 以外的方式获得的 std::future 的析构函数决不阻塞）

## 如何让C++字符串更快 in C++

1.内存分配建议：能分配在栈上就别分配到堆上，因为把内存分配到堆上会降低程序的速度 。

2.std::string_view同样是C++17的新特性

3.gcc的string默认大小是32个字节，字符串小于等于15直接保存在栈上，超过之后才会使用new分配

4.string的常用优化：**SSO**(短字符串优化)、**COW**（写时复制技术优化）

5.**为何优化字符串？**

> 1)`std::string`和它的很多函数都喜欢分配在堆上，这实际上并不理想 。 2)一般处理字符串时，比如使用`substr`切割字符串时，这个函数会自己处理完原字符串后**创建出**一个全新的字符串，它可以变换并有自己的内存（new,堆上创建）。 3)在数据传递中减少拷贝是提高性能的最常用办法。在C中指针是完成这一目的的标准数据结构，而在C++中引入了安全性更高的引用类型。所以在C++中若传递的数据仅仅可读，const string&成了C++天然的方式。但这并非完美，从实践上来看，它至少有以下几方面问题：
> **字符串字面值、字符数组、字符串指针**的**传递依然要数据拷贝** 这三类低级数据类型与string类型不同，传入时编译器要做**隐式转换**，即需要拷贝这些数据生成string临时对象。const string&指向的实际上是这个临时对象。通常字符串字面值较小，性能损失可以忽略不计；但字符串指针和字符数组某些情况下可能会比较大（比如读取文件的内容），此时会引起频繁的内存分配和数据拷贝，影响程序性能。
> **substr O(n)复杂度** substr是个常用的函数，好在std::string提供了这个函数，美中不足的时每次都要返回一个新生成的子串，很容易引起性能热点。实际上我们本意不是要改变原字符串，为什么不在原字符串基础上返回呢？

6.**如何优化字符串**？通过 **string_view**

> std::string_view是C++ 17标准中新加入的类，正如其名，它**提供一个字符串的视图**，即可以通过这个类以各种方法“观测”字符串，但不允许修改字符串。由于它只读的特性，它并不真正持有这个字符串的拷贝，而是与相对应的字符串共享这一空间。即——**构造时不发生字符串的复制**。同时，你也**可以自由的移动这个视图**，**移动视图并不会移动原定的字符串**。
> 通过调用 string_view 构造器可将字符串转换为 string_view 对象。string 可隐式转换为 string_view。
> 1）string_view 是只读的轻量对象，它对所指向的字符串没有所有权。
> 2）string_view通常用于函数参数类型，可用来取代 const char* 和 const string&。string_view 代替 const string&，可以避免不必要的内存分配。
> 3）string_view的成员函数即对外接口与 string 相类似，但只包含读取字符串内容的部分。 4）string_view::substr()的返回值类型是string_view，不产生新的字符串，不会进行内存分配。 5）string::substr()的返回值类型是string，产生新的字符串，会进行内存分配。
> 6）string_view字面量的后缀是 sv。（string字面量的后缀是 s）

```cpp
#include <iostream>
#include <string>

//一种调试在heap上分配内存的方法，自己写一个new的方法，然后设置断点或者打出log，就可以知道每次分配了多少内存，以及分配了几次
static uint32_t s_AllocCount = 0;
void* operator new(size_t size) 
{
    s_AllocCount++;
    std::cout << "Allocating " << size << " bytes\n";
    return malloc(size);
}

#define STRING_view 1
#if STRING_view
void PrintName(std::string_view name)
{
    std::cout << name << std::endl;
}
#else
void PrintName(const std::string& name)
{
    std::cout << name << std::endl;
}
#endif

int main()
{
    const std::string name = "Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs";
    // const char *cname = "Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs"; // C-like的编码风格

#if STRING_view
    std::string_view firstName(name.c_str(), 3);
    std::string_view lastName(name.c_str() + 4, 9);
#else
    std::string firstName = name.substr(0, 3); //substr切割字符串
    std::string lastName = name.substr(4, 9);
#endif

    PrintName(name);
    PrintName(firstName);
    PrintName(lastName);

    std::cout << s_AllocCount << " allocations" << std::endl;

    return 0;
}
```

输出：

```cpp
//无#define STRING_view 1
Allocating 8 bytes
Allocating 80 bytes
Allocating 8 bytes
Allocating 8 bytes
Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgsgsgsgsgsgsgsdgsgsgnj
Yan
Chernosaf
4 allocations

//有#define STRING_view 1
Allocating 8 bytes
Allocating 64 bytes
Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs
Yan
Chernosaf
2 allocations
```

可见 使用string_view减少了内存在堆上的分配。

**进一步优化：使用C风格字符串**

```cpp
int main()
{
    //const std::string name = "Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs";
    const char *cname = "Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs"; // C-like的编码风格

#if STRING_view
    std::string_view firstName(name, 3); //注意这里要去掉 .c_str()
    std::string_view lastName(name + 4, 9);
#else
    std::string firstName = name.substr(0, 3); 
    std::string lastName = name.substr(4, 9);
#endif

    PrintName(name);
    PrintName(firstName);
    PrintName(lastName);

    std::cout << s_AllocCount << " allocations" << std::endl;

    return 0;
}
```

输出

```cpp
//有#define STRING_view 1
Yan Chernosafhiahfiuauadvkjnkjasjfnanvanvanjasdfsgs
Yan
Chernosaf
0 allocations
```

注意：不同编译器的结果有所不同。

## C++的可视化基准测试

1.利用工具： chrome://tracing （chrome浏览器自带的一个工具，将该网址输入即可）

2.基本原理： cpp的计时器配合自制简易json配置写出类，将时间分析结果写入一个json文件，用chrome://tracing 这个工具进行可视化 。

3.多线程可视化实现：

> 视频：[https://www.bilibili.com/video/BV1gZ4y1R7SG?share_source=copy_web&vd_source=48739a103c73f618758b902392cb372e](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1gZ4y1R7SG%3Fshare_source%3Dcopy_web%26vd_source%3D48739a103c73f618758b902392cb372e) 代码改进链接：[https://github.com/GavinSun0921/InstrumentorTimer](https://link.zhihu.com/?target=https%3A//github.com/GavinSun0921/InstrumentorTimer)

4.实现代码：

```cpp
#pragma once
#include <string>
#include <chrono>
#include <algorithm>
#include <fstream>
#include <cmath>
#include <thread>
#include <iostream>

struct ProfileResult
{
    std::string Name;
    long long Start, End;
    uint32_t ThreadID; //线程ID
};

struct InstrumentationSession
{
    std::string Name;
};

class Instrumentor
{
private:
    InstrumentationSession* m_CurrentSession;
    std::ofstream m_OutputStream;
    int m_ProfileCount;
public:
    Instrumentor()
        : m_CurrentSession(nullptr), m_ProfileCount(0)
    {
    }

    void BeginSession(const std::string& name, const std::string& filepath = "results.json")
    {
        m_OutputStream.open(filepath);
        WriteHeader();
        m_CurrentSession = new InstrumentationSession{ name };
    }

    void EndSession()
    {
        WriteFooter();
        m_OutputStream.close();
        delete m_CurrentSession;
        m_CurrentSession = nullptr;
        m_ProfileCount = 0;
    }

    void WriteProfile(const ProfileResult& result)
    {
        if (m_ProfileCount++ > 0)
            m_OutputStream << ",";

        std::string name = result.Name;
        std::replace(name.begin(), name.end(), '"', '\'');

        m_OutputStream << "{";
        m_OutputStream << "\"cat\":\"function\",";
        m_OutputStream << "\"dur\":" << (result.End - result.Start) << ',';
        m_OutputStream << "\"name\":\"" << name << "\",";
        m_OutputStream << "\"ph\":\"X\",";
        m_OutputStream << "\"pid\":0,";
        m_OutputStream << "\"tid\":" << result.ThreadID << ","; //多线程
        m_OutputStream << "\"ts\":" << result.Start;
        m_OutputStream << "}";

        m_OutputStream.flush();
    }

    void WriteHeader()
    {
        m_OutputStream << "{\"otherData\": {},\"traceEvents\":[";
        m_OutputStream.flush();
    }

    void WriteFooter()
    {
        m_OutputStream << "]}";
        m_OutputStream.flush();
    }

    static Instrumentor& Get()
    {
        static Instrumentor instance;
        return instance;
    }
};

class InstrumentationTimer
{
public:
    InstrumentationTimer(const char* name)
        : m_Name(name), m_Stopped(false)
    {
        m_StartTimepoint = std::chrono::high_resolution_clock::now();
    }

    ~InstrumentationTimer()
    {
        if (!m_Stopped)
            Stop();
    }

    void Stop()
    {
        auto endTimepoint = std::chrono::high_resolution_clock::now();

        long long start = std::chrono::time_point_cast<std::chrono::microseconds>(m_StartTimepoint).time_since_epoch().count();
        long long end = std::chrono::time_point_cast<std::chrono::microseconds>(endTimepoint).time_since_epoch().count();

        uint32_t threadID = std::hash<std::thread::id>{}(std::this_thread::get_id());//在timer中的stop函数看看timer实际上是在哪个线程上运行的
        Instrumentor::Get().WriteProfile({ m_Name, start, end, threadID }); 

        m_Stopped = true;
    }
private:
    const char* m_Name;
    std::chrono::time_point<std::chrono::high_resolution_clock> m_StartTimepoint;
    bool m_Stopped;
};

//测试代码
//计时器不是一直用的，应该有一个简单的方法来关闭这些，因为这会增加一些开销。通过定义宏可以做到。
//定义宏PROFILING，如果这个设置为1，那么PROFILING是启用的，这意味着我们会有PROFILE_SCOPE引入一个InstrumentationTimer，并做所有这些事情
#define PROFILING 1 //如果被禁用，比如设为0，则运行空代码，这意味着PROFILE_SCOPE没有任何代码.这可以有效地从PROFILING设为0的构建中，剥离计时器
#if PROFILING
//定义一个宏叫做PROFILE_SCOPE，它会把name作为参数，这会包装我们的InstrumentationTimer调用
#define PROFILE_SCOPE(name) InstrumentationTimer timer##__LINE__(name) //##__LINE__作用市拼接行号，让每个实例的实例名不一样，加上之后，就不是time了，而是time加行号。在这个例子中确实可以不用，但是如果你在一个作用域调用两次，就会造成实例名重定义错误。
#define PROFILE_FUNCTION() PROFILE_SCOPE(__FUNCSIG__) //定义一个PROFILE_FUNCTION()宏，会调用PROFILE_SCOPE宏，但对于name，它会接受函数的名字，我们可以用这个编译宏__FUNCSIG__来做(__FUNCSIG__可以避免函数重载带来的问题)
#else
#define PROFILE_SCORE(name)
#endif

//如果你的代码中有你想要分析的区域，特别的，不是在函数中你可以将PROFILE_SCOPE放到任何作用域当中
namespace Benmarks  //可以把PROFILE_FUNCTION()放到程序的每一个函数中。比如这个Benmarks命名空间。且因为用了宏__FUNCSIG__，可以得到空间内的所有信息
{
    void Function(int value) {
        // PROFILE_SCOPE("Function1"); //同InstrumentationTimer timer("Function1");
        PROFILE_FUNCTION(); //同 PROFILE_SCOPE("Function1");
        for (int i = 0; i < 1000; i++)
            std::cout << "hello world" << (i + value) << std::endl;
    }
    void Function() {
        //PROFILE_SCOPE("Function2");
        PROFILE_FUNCTION();
        for (int i = 0; i < 1000; i++)
            std::cout << "hello world" << i << std::endl;
    }

    void RunBenchmarks() {
        //PROFILE_SCOPE("RunBenchmarks");
        PROFILE_FUNCTION();
        std::cout << "Runint Benchmarks....\n";
        Function(2);
        Function();
    }
}
int main() {
    Instrumentor::Get().BeginSession("Profile");    
    Benmarks::RunBenchmarks();
    Instrumentor::Get().EndSession();
    std::cin.get();
}
```

## C++的单例模式

1.Singleton只允许被实例化一次，用于组织一系列全局的函数或者变量，与namespace很像。例子：随机数产生的类、渲染器类。

2.**C++中的单例只是一种组织一堆全局变量和静态函数的方法**

3.什么时候用单例模式：

当我们想要**拥有应用于某种全局数据集的功能**，且我们**只是想要重复使用**时，单例是非常有用的

> 有些单例的例子，比如一个随机数生成器类 我们只希望能够查询它，得到一个随机数，而不需要实例化它去遍历所有这些东西我们只想实例化它一次（单例），这样它就会生成随机数生成器的种子，建立起它所需要的任何辅助的东西了 另一个例子就是渲染器，渲染器通常是一个非常全局的东西 我们通常不会有一个渲染器的多个实例，我们有一个渲染器，我们向它提交所有这些渲染命令，然后它就会为我们渲染

4.实现单例的基本方法：

1)**将构造函数设为私有**，因为单例类不能有第二个实例

2)提供一个静态访问该类的方法

> 设一个**私有的静态的实例**，**并且在类外将其定义！** 然后用一个静态函数返回其引用or指针，便可正常使用了

3)为了安全，**标记拷贝构造函数为delete**（删除拷贝构造函数）

```cpp
#include <iostream>

class SingleTon {
    SingleTon(const SingleTon&) = delete; //删除拷贝构造函数
public:
    //static在类里表示将该函数标记为静态函数
    static SingleTon& get() {
        return m_temp;
    }

    void Function() {}  //比如说这里有一些方法可供使用

private:
    SingleTon() {}; //将构造函数标记为私有
    static SingleTon m_temp;    //在私有成员里创建一个静态实例
};

SingleTon SingleTon::m_temp;    //像定义任何静态成员一样定义它


int main() {
    //SingleTon temp2 = SingleTon::get();       //会报错，因为无法复制了
    SingleTon& temp2 = SingleTon::get();        //会报错，因为无法复制了
   // SingleTon::get().Function();  //这般使用便可
    temp2.Function();
}
```

一个简单的随机数类的例子

1)将构造函数设为私有，防止从外部被实例化

2)设置get()函数来返回静态引用的实例

> 直接在get()中设置静态实例就可以了，在调用get()的时候就直接设置静态实例

3)标记复制构造函数为delete

```cpp
#include<iostream>
class Random
{
public:
    Random(const Random&) = delete; // 删除拷贝复制函数
    static Random& Get() // 通过Get函数来获取唯一的一个实例
    {
        static Random instance; // 在此处实例化一次
        return instance;
    }
    static float Float(){ return Get().IFloat();} // 调用内部函数,可用类名调用
private:
    float IFloat() { return m_RandomGenerator; } // 将函数的实现放进private
    Random(){} // 不能让别人实例化，所以要把构造函数放进private
    float m_RandomGenerator = 0.5f;
};
// 与namespace很像
namespace RandomClass {
    static float s_RandomGenerator = 0.5f;
    static float Float(){return s_RandomGenerator;}
}
int main()
{
    float randomNum = Random::Float();
    std::cout<<randomNum<<std::endl;
    std::cin.get();
}
```

## C++的小字符串优化

VS开发工具在release模式下面 （debug模式都会在堆上分配） ，使用size小于16的string，不会分配内存，而大于等于16的string，则会分配32bytes内存以及更多，所以16个字符是一个分界线 （注：不同编译器可能会有所不同）

```cpp
#include <iostream>

void* operator new(size_t size)
{
    std::cout << "Allocated: " << size << " bytes\n";
    return malloc(size);
}

int main()
{
    // debug模式都会在堆上分配
    std::string longName = "cap cap cap cap "; // 刚好16个字符，会在堆上分配32个bytes内存
    std::string testName = "cap cap cap cap"; // 15个字符，栈上分配
    std::string shortName = "cap";

    std::cin.get();
}
//debug模式输出
Allocated: 16 bytes
Allocated: 32 bytes
Allocated: 16 bytes
Allocated: 16 bytes

//release模式输出：
Allocated: 32 bytes
```

## 跟踪内存分配的简单方法

内存泄漏 

1、什么是内存泄露? 

内存泄漏(memory leak)是指由于疏忽或错误造成了程序未能释放掉不再使⽤的内存的情 况。内存泄漏并⾮指内存在物理上的消失，⽽是应⽤程序分配某段内存后，由于设计错误， 失去了对该段内存的控制，因⽽造成了内存的浪费。 

可以使⽤Valgrind, mtrace进⾏内存泄漏检查。 

2、内存泄漏的分类 

（1）堆内存泄漏 （Heap leak）  

对内存指的是程序运⾏中根据需要分配通过malloc,realloc new等从堆中分配的⼀块内存， 再是完成后必须通过调⽤对应的 free或者 delete 删掉。如果程序的设计的错误导致这部分 内存没有被释放，那么此后这块内存将不会被使⽤，就会产⽣ Heap Leak.  

（2）系统资源泄露（Resource Leak）  

主要指程序使⽤系统分配的资源⽐如 Bitmap,handle ,SOCKET 等没有使⽤相应的函数释放 掉，导致系统资源的浪费，严重可导致系统效能降低，系统运⾏不稳定。  

（3）没有将基类的析构函数定义为虚函数  

当基类指针指向⼦类对象时，如果基类的析构函数不是 virtual，那么⼦类的析构函数将不 会被调⽤，⼦类的资源没有正确是释放，因此造成内存泄露。



重写new和delete操作符函数，并在里面打印分配和释放了多少内存，也可在重载的这两个函数里面设置断点，通过查看调用栈即可知道什么地方分配或者释放了内存

> 我们知道一个class的new是分为三步：operator new（其内部调用malloc）返回void*、static_cast转换为这个对象指针、构造函数。而delete则分为两步：构造函数、operator delete。 new和delete都是表达式，是不能重载的；而把他们行为往下分解则是有operator new和operator delete，是有区别的。 直接用的表达式的行为是不能变的，不能重载的，即new分解成上图的三步与delete分解成上图的两步是不能重载的。这里内部的operator new和operator delete底层其实是调用的malloc，这些内部的几步则是可以重载的。 原文链接：[https://blog.csdn.net/weixin_47652005/article/details/121026982](https://link.zhihu.com/?target=https%3A//blog.csdn.net/weixin_47652005/article/details/121026982)

```cpp
// 重写new、free操作符之后就能方便地跟踪内存分配了(加断点)

#include <iostream>
#include <memory>

struct AllocationMetrics
{
    uint32_t TotalAllocated = 0; //总分配内存
    uint32_t TotalFreed = 0; //总释放内存

    uint32_t CurrentUsage() { return TotalAllocated - TotalFreed; } //写一个小函数来输出 当前用了多少内存
};

static AllocationMetrics s_AllocationMetrics; //创建一个全局静态实例

void *operator new(size_t size)
{
    s_AllocationMetrics.TotalAllocated += size; //在每一个new里计算总共分配了多少内存
    // std::cout << "Allocate " << size << " bytes.\n";
    return malloc(size);
}

void operator delete(void *memory, size_t size)
{
    s_AllocationMetrics.TotalFreed += size;
    // std::cout << "Free " << size << " bytes.\n";
    free(memory);
}

struct Object
{
    int x, y, z;
};
//可以用一个函数输出我们的内存使用情况
static void PrintMemoryUsage()
{
    std::cout << "Memory Usage:" << s_AllocationMetrics.CurrentUsage() << " bytes\n";
}

int main()
{
    PrintMemoryUsage();
    {
        std::unique_ptr<Object> obj = std::make_unique<Object>();
        PrintMemoryUsage();
    }

    PrintMemoryUsage();
    Object *obj = new Object;
    PrintMemoryUsage();
    delete obj;
    PrintMemoryUsage();
    std::string string = "Cherno";
    PrintMemoryUsage();

    return 0;
}
```

## C++的左值与右值(lvalue and rvalue)

1.左值：

> 有地址 数值 有存储空间的值，往往长期存在； 左值是**由某种存储支持的变量**；**左值有地址和值**，可以出现在赋值运算符左边或者右边。

2.左值引用:

> 左值引用仅仅接受左值，除非用了const兼容（ 非const的左值引用只接受左值 ） 所以C++常用**常量**引用。**因为它们兼容临时的右值和实际存在的左值变量**

3.右值：

> 是**临时量**，无地址（或者说有地址但访问不到，它只是一个临时量） 没有存储空间的短暂存在的值 。

4.右值引用：

> 右值引用不能绑定到左值 可以通过常引用或者右值引用延长右值的生命周期 “有名字的右值引用是左值”

5.右值引用的优势：**优化**

> 如果我们知道传入的是一个临时对象的话，那么我们就不需要担心它们是否活着，是否完整，是否拷贝。我们可以简单地偷它的资源，给到特定的对象，或者其他地方使用它们。因为我们知道它是暂时的，它不会存在很长时间 而如果如上使用const string& str，虽然可以兼容右值，但是却不能从这个字符串中窃取任何东西！因为这个str可能会在很多函数中使用，不可乱修改！（所以才加了const）

6.在给函数形参列表传参时，有四种情况：

```cpp
#include<iostream>
void PrintName(std::string name) // 可接受左值和右值
{
    std::cout<<name<<std::endl;
}
void PrintName(std::string& name) // 只接受左值引用，不接受右值
{
    std::cout << name << std::endl;
}
void PrintName(const std::string& name) // 接受左值和右值，把右值当作const lvalue&
{
    std::cout << name << std::endl;
}
void PrintName(std::string&& name) // 接受右值引用
{
    std::cout << name << std::endl;
}
int main()
{
    std::string firstName = "yang";
    std::string lastName = "dingchao";
    std::string fullName = firstName + lastName; //右边的表达式是个右值。
    PrintName(fullName);
    PrintName(firstName+lastName);
    std::cin.get();
```

7.cherno的演示代码如下：

```cpp
#include <iostream>

int &GetValue()
{ // 左值引用
    static int value = 10;
    return value;
}

void SetValue(int value) {}

void PrintName(std::string &name)
{ // 非const的左值引用只接受左值
    std::cout << "[lvalue]" << name << std::endl;
}

void PrintName(const std::string &&name)
{ // 右值引用不能绑定到左值
    std::cout << "[rvalue]" << name << std::endl;
}

int main()
{
    int i = GetValue();
    GetValue() = 5;

    SetValue(i);  // 左值参数调用
    SetValue(10); // 右值参数调用，当函数被调时，这个右值会被用来创建一个左值

    // 关于const，const引用可以同时接受左值和右值
    // int& a = 10; // 不能用左值作为右值的引用
    const int &a = 10; // 通过创建一个左值实现

    std::string firstName = "Yan";
    std::string lastName = "Chernikov";
    std::string fullname = firstName + lastName;
    PrintName(fullname);             // 接受左值
    PrintName(firstName + lastName); // 接受右值

    return 0;
}
```

## C++ 完美转发

**完美转发 = 引用折叠 + 万能引用 + std::forward**

### 动机：C++为什么需要完美转发？

我们从一个简单的例子出发。 假设有这么一种情况，用户一般使用testForward函数，testForward什么也不做，只是简单的转调用到print函数。

```cpp
template<typename T>
void print(T & t){
    std::cout << "Lvalue ref" << std::endl;
}

template<typename T>
void print(T && t){
    std::cout << "Rvalue ref" << std::endl;
}

template<typename T>
void testForward(T && v){ 
    print(v);//v此时已经是个左值了,永远调用左值版本的print
    print(std::forward<T>(v)); //本文的重点
    print(std::move(v)); //永远调用右值版本的print

    std::cout << "======================" << std::endl;
}

int main(int argc, char * argv[])
{
    int x = 1;
    testForward(x); //实参为左值
    testForward(std::move(x)); //实参为右值
}
```

上面的程序的运行结果：

```text
Lvalue ref
Lvalue ref
Rvalue ref
======================
Lvalue ref
Rvalue ref
Rvalue ref
======================
```

用户希望`testForward(x);`最终调用的是左值版本的print，而`testForward(std::move(x));`最终调用的是右值版本的print。

**可惜的是，在testForward中，虽然参数v是右值类型的，但此时v在内存中已经有了位置，所以v其实是个左值！（请仔细阅读这段话，保证你理解了）**

所以，`print(v)`永远调用左值版本的print，与用户的本意不符。`print(std::move(v));`永远调用右值版本的print，与用户的本意也不符。只有`print(std::forward<T>(v));`才符合用户的本意，这就是本文的主题。

不难发现，本质问题在于，左值右值在函数调用时，都转化成了左值，使得函数转调用时无法判断左值和右值。

在STL中，随处可见这种问题。比如C++11引入的`emplace_back`，它接受左值也接受右值作为参数，接着，它转调用了空间配置器的construct函数，而construct又转调用了`placement new`，`placement new`根据参数是左值还是右值，决定调用拷贝构造函数还是移动构造函数。

这里要保证从`emplace_back`到`placement new`，参数的左值和右值属性保持不变。这其实不是一件简单的事情。

### 前置知识 引用折叠 万能引用

C++ Primer 里面写的比较容易理解，在P608（我的是第5版）。

也可以参考我之前的文章：

[万能引用，引用折叠，移动构造函数，emplace_back及其实现，完美转发及其实现32 赞同 · 3 评论文章](https://zhuanlan.zhihu.com/p/260508149)

### 原理：完美转发

std::forward不是独自运作的，在我的理解里，完美转发 = std::forward + 万能引用 + 引用折叠。三者合一才能实现完美转发的效果。

std::forward的正确运作的前提，是引用折叠机制，为T &&类型的万能引用中的模板参数T赋了一个恰到好处的值。我们用T去指明std::forward的模板参数，从而使得std::forward返回的是正确的类型。（看不懂？没关系，先看下面的例子和代码）

当然，我们还是先回到一开始的例子。

### testForward(x)

回到上面的例子。先考虑`testForward(x);`这一行代码。

### step 1 实例化testForward

根据万能引用的实例化规则，实例化的testForward大概长这样：

```cpp
T = int &
void testForward(int & && v){
    print(std::forward<T>(v));
}
```

又根据引用折叠，上面的等价于下面的代码：

```cpp
T = int &
void testForward(int & v){
    print(std::forward<int &>(v));
}
```

如果你正确的理解了引用折叠，那么这一步是很好理解的。

### step 2 实例化std::forward

> 注：C++ Primer：forward必须通过显式模板实参来调用，不能依赖函数模板参数推导。

接下来我们来看下`std::forward`在libstdc++中的实现：

```cpp
68   /**
69    *  @brief  Forward an lvalue.
70    *  @return The parameter cast to the specified type.
71    *
72    *  This function is used to implement "perfect forwarding".
73    */
74   template<typename _Tp>
75     constexpr _Tp&&
76     forward(typename std::remove_reference<_Tp>::type& __t) noexcept
77     { return static_cast<_Tp&&>(__t); }
```

由于Step1中我们调用`std::forward<int &>`，所以此处我们代入`T = int &`，我们有：

```cpp
constexpr int & && //折叠
forward(typename std::remove_reference<int &>::type& __t) noexcept //remove_reference的作用与名字一致，不过多解释
 { return static_cast<int & &&>(__t); } //折叠
```

这里又发生了2次引用折叠，所以上面的代码等价于：

```cpp
constexpr int & //折叠
forward(int & __t) noexcept //remove_reference的作用与名字一致，不过多解释
 { return static_cast<int &>(__t); } //折叠
```

所以最终`std::forward<int &>(v)`的作用就是将参数强制转型成`int &`，而`int &`为左值。所以，调用左值版本的print。

### testForward(std::move(x))

接下来，考虑`testForward(std::move(x))`的情况。

### step 1 实例化testForward

`testForward(std::move(x))`也就是`testForward(static_cast<int &&>(x))`。根据万能引用的实例化规则，实例化的testForward大概长这样：

```cpp
T = int 
void testForward(int && v){
    print(std::forward<int>(v));
}
```

万能引用绑定到右值上时，不会发生引用折叠，所以这里没有引用折叠。

### step 2 实例化std::forward

> 注：C++ Primer：forward必须通过显式模板实参来调用，不能依赖函数模板参数推导。
> 这里用到的std::forward的代码和上面的一样，故略去。
> 由于Step1中我们调用`std::forward<int>`，所以此处我们代入`T = int`，我们有：

```cpp
constexpr int && 
forward(typename std::remove_reference<int>::type& __t) noexcept //remove_reference的作用与名字一致，不过多解释
 { return static_cast<int &&>(__t); }
```

这里又发生了2次引用折叠，所以上面的代码等价于：

```cpp
constexpr int &&
forward(int & __t) noexcept //remove_reference的作用与名字一致，不过多解释
 { return static_cast<int &&>(__t); }
```

所以最终`std::forward<int>(v)`的作用就是将参数强制转型成`int &&`，而`int &&`为右值。所以，调用右值版本的print。

## C++持续集成（CI）

> 1、 CI(Continuous integration，中文意思是持续集成)是一种软件开发时间。持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。 
>
> 2、主要讲解如何在linode租一个服务器，来运行Jenkins

## C++静态分析

- 主要讲了一个工具PVS-studio的用法，可以static analyze代码
- 开源的推荐 clang-tidy

> [如何在VS Code中运行clang-tidy?： https://zhuanlan.zhihu.com/p/446084601](https://zhuanlan.zhihu.com/p/446084601)

## C++的参数计算顺序

1、讲了一个undefined behavior的例子：

```cpp
//参考：https://zhuanlan.zhihu.com/p/352420950
#include<iostream>
void PrintSum(int a, int b)
{
    std::cout<<a<<"+"<<b<<"="<<a+b<<std::endl;
}
int main()
{
    int value = 0;
    PrintSum(value++,++value); //行为未定义！
    std::cin.get();
}
```

类似这样在传参时使用++，这种行为是不确定的，在不同编译器不同语言版本和配置下，其行为不一致，所以严禁这样使用

## C++移动语义

1.当我们知道左值和右值，左值引用和右值引用后，我们可以看看它们最大的一个用处：移动语义

2.为什么需要移动语义？

> 很多时候我们只是单纯创建一些右值，然后赋给某个对象用作构造函数。这时候会出现的情况是：首先需要在main函数里创建这个右值对象，然后复制给这个对象相应的成员变量。 如果我们可以直接把这个右值变量**移动**到这个成员变量而不需要做一个额外的复制行为，**程序性能**就能**提高**。

3.**noexcept 指定符**

> 含义：指定函数是否抛出异常。 举例：void f() noexcept {};*// 函数 f() 不抛出*异常

案例:

> **不用移动构造函数**：

```cpp
//笔记代码参考：https://www.cnblogs.com/zhangyi1357/p/16018810.html
#include <iostream>
#include <cstring>

class String {
public:
    String() = default;
    String(const char* string) {  //构造函数
        printf("Created!\n");
        m_Size = strlen(string);
        m_Data = new char[m_Size];
        memcpy(m_Data, string, m_Size);
    }

    String(const String& other) { // 拷贝构造函数
        printf("Copied!\n");
        m_Size = other.m_Size;
        m_Data = new char[m_Size];
        memcpy(m_Data, other.m_Data, m_Size);
    }

    ~String() {
        delete[] m_Data;
    }

    void Print() {
        for (uint32_t i = 0; i < m_Size; ++i)
            printf("%c", m_Data[i]);

        printf("\n");
    }
private:
    char* m_Data;
    uint32_t m_Size;
};

class Entity {
public:
    Entity(const String& name)
        : m_Name(name) {}
    void PrintName() {
        m_Name.Print();
    }
private:
    String m_Name;
};

int main(int argc, const char* argv[]) {
    Entity entity(String("Cherno"));
    entity.PrintName();

    return 0;
}
//输出结果：
Created!
Copied!
Cherno
```

可以看到中间发生了一次copy，实际上这次copy发生在Entity的初始化列表里。 从String的复制构造函数可以看到，复制过程中还申请了新的内存空间！这会**带来很大的消耗**。

> **使用移动构造函数**

```cpp
#include<iostream>
class String
{
public:
    String() = default;
    String(const char* string) //构造函数
    {
        printf("Created\n");
        m_Size = strlen(string);
        m_Data = new char[m_Size];
        memcpy(m_Data, string, m_Size);
    }
    String(const String& other) // 拷贝构造函数
    {
        printf("Copied\n");
        m_Size = other.m_Size;
        m_Data = new char[m_Size];
        memcpy(m_Data, other.m_Data, m_Size);
    }
    String(String&& other) noexcept // 右值引用拷贝，相当于移动，就是把复制一次指针，原来的指针给nullptr
    {
        //让新对象的指针指向指定内存，然后将旧对象的指针移开
        //所以这里做的其实是接管了原来旧的内存，而不是将这片内存复制粘贴！
        printf("Moved\n");
        m_Size = other.m_Size;
        m_Data = other.m_Data;
        //这里便完成了数据的转移，将other里的数据偷走了
        other.m_Size = 0;
        other.m_Data = nullptr;
    }
    ~String()
    {
        printf("Destroyed\n");
        delete m_Data;
    }
    void Print() {
        for (uint32_t i = 0; i < m_Size; ++i)
            printf("%c", m_Data[i]);

        printf("\n");
    }
private:
    uint32_t m_Size;
    char* m_Data;
};
class Entity
{
public:
    Entity(const String& name) : m_Name(name)
    {
    }
    void PrintName() {
        m_Name.Print();
    }

    Entity(String&& name) : m_Name(std::move(name)) // std::move(name)也可以换成(String&&)name
    {
    }
private:
    String m_Name;
};
int main()
{
    Entity entity(String("Cherno"));
    entity.PrintName();
    std::cin.get();
}
//输出:
Created!
Moved!
Destroyed!
Cherno
Destroyed
```

没有copied！问题完美解决。

有名字的右值引用是左值

> 每个表达式都有两种特征：一是类型二是值类别。很多人迷惑的右值引用为啥是个左值，那是因为右值引用是它的类型，左值是它的值类别。 想理解右值首先要先知道类型和值类别的区别；其次是各个值类别的定义是满足了某种形式它就是那个类别，经常说的能取地址就是左值，否则就是右值，这是定义之上的不严谨经验总结，换句话说，是左值还是右值是强行规定好的，你只需要对照标准看这个表达式满足什么形式就知道它是什么值类别了。 为什么要有这个分类，是为了语义，当一个表达式出现的形式表示它是一个右值，就是告诉编译器，我以后不会再用到这个资源，放心大胆的转移销毁，这就可以做优化，比如节省拷贝之类的。 move的作用是无条件的把表达式转成右值，也就是rvalue_cast，虽然编译器可以推断出左右值，但人有时比编译器“聪明”，人知道这个表达式的值以后我不会用到，所以可以在正常情况下会推成左值的地方强行告诉编译器，我这是个右值，请你按右值的语义来做事。

## std::move与移动赋值操作符

1.使用std::move，返回一个右值引用，可以将本来的copy操作变为move操作

2.有时候我们想要将一个已经存在的对象移动给另一个已经存在的对象，就像下面这样。

> **移动赋值**相当于把别的对象的资源都偷走，那如果移动到自己头上了就没必要自己偷自己 。 更重要的是**原来自己的资源一定要释放掉**，否则指向自己原来内容内存的指针就没了，这一片内存就泄露了！

```cpp
#include<iostream>
class String
{
public:
    String() = default;
    String(const char* string)
    {
        printf("Created\n");
        m_Size = strlen(string);
        m_Data = new char[m_Size];
        memcpy(m_Data, string, m_Size);
    }
    String(const String& other) 
    {
        printf("Copied\n");
        m_Size = other.m_Size;
        m_Data = new char[m_Size];
        memcpy(m_Data, other.m_Data, m_Size);
    }
    String(String&& other) noexcept
    {
        printf("Moved\n");
        m_Size = other.m_Size;
        m_Data = other.m_Data;

        other.m_Size = 0;
        other.m_Data = nullptr;
    }
    ~String()
    {
        printf("Destroyed\n");
        delete m_Data;
    }
    void Print() {
        for (uint32_t i = 0; i < m_Size; ++i)
            printf("%c", m_Data[i]);

        printf("\n");
    }
    String& operator=(String&& other) // 移动复制运算符重载
    {
        printf("Moved\n");
        if (this != &other)
        {
            delete[] m_Data;

            m_Size = other.m_Size;
            m_Data = other.m_Data;

            other.m_Data = nullptr;
            other.m_Size = 0;
        }
        return *this;
    }
private:
    uint32_t m_Size;
    char* m_Data;
};
class Entity
{
public:
    Entity(const String& name) : m_Name(name)
    {
    }
    void PrintName() {
        m_Name.Print();
    }
    Entity(String&& name) : m_Name(std::move(name)) // std::move(name)也可以换成(String&&)name
    {
    }
private:
    String m_Name;
};
int main()
{
    String apple = "apple";
    String orange = "orange";

    printf("apple: ");
    apple.Print();
    printf("orange: ");
    orange.Print();

    apple = std::move(orange);

    printf("apple: ");
    apple.Print();
    printf("orange: ");
    orange.Print();
    std::cin.get();
}
//输出：
Created
Created
apple: apple
orange: orange
Moved
apple: orange
orange:
```



## copy and swap

> 为什么我们需要复制和交换习惯？

任何管理资源的类（包装程序，如智能指针）都需要实现big three。尽管拷贝构造函数和析构函数的目标和实现很简单。

但是复制分配运算符无疑是最细微和最困难的。

> 应该怎么做？需要避免什么陷阱？

copy-swap是解决方案，可以很好地协助赋值运算符实现两件事：避免代码重复，并提供强大的**异常保证**。

> 它是如何工作的？

**从概念上讲，它通过使用拷贝构造函数的功能来创建数据的本地副本，然后使用交换功能获取复制的数据，将旧数据与新数据交换来工作。然后，临时副本将销毁，并随身携带旧数据。我们剩下的是新数据的副本。**

为了使用copy-swap，我们需要三件事：

- 一个有效的拷贝构造函数
- 一个有效的析构函数（两者都是任何包装程序的基础，因此无论如何都应完整）以及交换功能。

交换函数是一种不抛异常函数，它交换一个类的两个对象或者成员。我们可能很想使用std :: swap而不是提供我们自己的方法，但这是不可能的。 std :: swap在实现中使用了copy-constructor和copy-assignment运算符，我们最终将尝试根据自身定义赋值运算符！

具体例子如下：

```cpp
namespace A {
    template<typename T>
    class smart_ptr {
    public:
        smart_ptr() noexcept : ptr_(new T()) {

        }

        smart_ptr(const T &ptr) noexcept : ptr_(new T(ptr)) {

        }

        smart_ptr(smart_ptr &rhs) noexcept {
            std::cout << "copy ctor" << std::endl;
            ptr_ = rhs.release();       // 释放所有权,此时rhs的ptr_指针为nullptr
        }

        // 方法1：为了避免自赋值,通常采用下面写法   不好!  不具备异常安全,只具备自我赋值安全性
//        smart_ptr &operator=(const smart_ptr &rhs) {
//            if (*this != rhs) {
//                delete ptr_;
//                ptr_ = new T(rhs.ptr_);  // 当new 发生异常,此时ptr_指向的而是一块被删除区域,而不是被赋值对象的区域
//                return *this;
//            }
//            return *this;
//        }
        // 方法2：如果new出现异常,ptr_会保持原装!  也可以处理自我赋值! 还是不够好!
//        smart_ptr &operator=(const smart_ptr &rhs) {
//            T *origin = ptr_;
//            ptr_ = new T(rhs.ptr_);
//            delete origin;
//            return *this;
//        }
        // 方法3：copy and swap 很好!
//        smart_ptr &operator=(smart_ptr &rhs) noexcept {
//            smart_ptr tmp(rhs);
//            swap(tmp);
//            return *this;
//        }

        // 方法4：同方法3,改为传值
        // 既适用于copy ctor也适用于 move ctor
        smart_ptr &operator=(smart_ptr rhs) noexcept {
            swap(rhs);
            return *this;
        }
        // move ctor
        smart_ptr(smart_ptr &&rhs) noexcept {
            std::cout << "move ctor" << std::endl;
            ptr_ = rhs.ptr_;
            if (ptr_)
                rhs.ptr_ = nullptr;
        }

        // move assignment
//        smart_ptr &operator=(smart_ptr &&rhs) noexcept {
//            std::cout << "move assignment" << std::endl;
//            smart_ptr tmp(rhs);
//            swap(rhs);
//            return *this;
//        }

        void swap(smart_ptr &rhs) noexcept { // noexcept == throw() 保证不抛出异常
            using std::swap;
            swap(ptr_, rhs.ptr_);
        }

        T *release() noexcept {
            T *ptr = ptr_;
            ptr_ = nullptr;
            return ptr;
        }

        T *get() const noexcept {
            return ptr_;
        }

    private:
        T *ptr_;
    };

// 提供一个非成员swap函数for ADL(Argument Dependent Lookup)
    template<typename T>
    void swap(A::smart_ptr<T> &lhs, A::smart_ptr<T> &rhs) noexcept {
        lhs.swap(rhs);
    }
}
// 注释开启,会引发ADL冲突
//namespace std {
//    // 提供一个非成员swap函数for ADL(Argument Dependent Lookup)
//    template<typename T>
//    void swap(A::smart_ptr<T> &lhs, A::smart_ptr<T> &rhs) noexcept {
//        lhs.swap(rhs);
//    }
//
//}

int main() {

    using std::swap;
    A::smart_ptr<std::string> s1("hello"), s2("world");
    // 交换前
    std::cout << *s1.get() << " " << *s2.get() << std::endl;
    swap(s1, s2);      // 这里swap 能够通过Koenig搜索或者说ADL根据s1与s2的命名空间来查找swap函数
    // 交换后
    std::cout << *s1.get() << " " << *s2.get() << std::endl;
//    s1 = s2;

    A::smart_ptr<std::string> s3 = s1;
    A::smart_ptr<std::string> s4 = std::move(s1);
}
```



**实际上，我们比那个不需要多写代码move assignment，copy-and-swap 技巧 和 move-and-swap 技巧是共享同一个函数的。**当copy构造为上述的方法4时，**对于C++ 11，编译器会依据参数是左值还是右值在拷贝构造函数和移动构造函数间进行选择：**

```cpp
smart_ptr &operator=(smart_ptr rhs) noexcept {
    swap(rhs);
    return *this;
}
```

所以当这个同上述写的

```cpp
smart_ptr &operator=(smart_ptr &&rhs) noexcept{}
```

同时存在，就会出现error: ambiguous overload for ‘operator=’ 。

调用处如下：

```cpp
A::smart_ptr<std::string> s1("hello"), s2("world");
A::smart_ptr<std::string> s3 = s1;
A::smart_ptr<std::string> s4 = std::move(s1);
```

- 如果是 s3 = s1，这样就会调用拷贝构造函数来初始化other（因为s1是左值），赋值操作符会与新创建的对象交换数据，深度拷贝。这就是copy and swap 惯用法的定义：构造一个副本，与副本交换数据，并让副本在作用域内自动销毁。
- 如果是s4 = std::move(s1)，这样就会调用移动构造函数来初始化rhs（因为std::move(s1)是右值），所以这里没有深度拷贝，只有高效的数据转移。

因此也可以称呼它为“统一赋值操作符”，因为它合并了"拷贝赋值"与"移动赋值"。

## C++惯用法之pImpl

“指向实现的指针”或“pImpl”是一种 C++ 编程技巧,它将类的实现细节从对象表示中移除，放到一个分离的类中，并以一个不透明的指针进行访问。

使用pImpl惯用法的原因如下：

考虑如下例子：

```cpp
class X
{
private:
  C c;
  D d;  
} ;
```

变成pImpl就是下面这样子

```cpp
class X
{
private:
  struct XImpl;
  XImpl* pImpl;       
};
```

CPP定义：

```cpp
struct X::XImpl
{
  C c;
  D d;
};
```

- 二进制兼容性

开发库时，可以在不破坏与客户端的二进制兼容性的情况下向XImpl添加/修改字段（这将导致崩溃！）。 由于在向Ximpl类添加新字段时X类的二进制布局不会更改，因此可以安全地在次要版本更新中向库添加新功能。

当然，您也可以在不破坏二进制兼容性的情况下向X / XImpl添加新的公共/私有非虚拟方法，但这与标准的标头/实现技术相当。

- 数据隐藏

如果您正在开发一个库，尤其是专有库，则可能不希望公开用于实现库公共接口的其他库/实现技术。 要么是由于知识产权问题，要么是因为您认为用户可能会被诱使对实现进行危险的假设，或者只是通过使用可怕的转换技巧来破坏封装。 PIMPL解决/缓解了这一难题。

- 编译时间

编译时间减少了，因为当您向XImpl类添加/删除字段和/或方法时（仅映射到标准技术中添加私有字段/方法的情况），仅需要重建X的源（实现）文件。 实际上，这是一种常见的操作。

使用标准的标头/实现技术（没有PIMPL），当您向X添加新字段时，曾经重新分配X（在堆栈或堆上）的每个客户端都需要重新编译，因为它必须调整分配的大小 。 好吧，每个从未分配X的客户端也都需要重新编译，但这只是开销（客户端上的结果代码是相同的）。

## ADL实参依赖查找

实参依赖查找（argument-dependent lookup），又称 ADL 或 Koenig 查找 [[1\]](https://zh.cppreference.com/w/cpp/language/adl#cite_note-1)，是一组对[函数调用表达式](https://zh.cppreference.com/w/cpp/language/operator_other)（包括对[重载运算符](https://zh.cppreference.com/w/cpp/language/operators)的隐式函数调用）中的无限定的函数名进行查找的规则。在通常[无限定名字查找](https://zh.cppreference.com/w/cpp/language/lookup)所考虑的作用域和命名空间之外，还会在它的各个实参的命名空间中查找这些函数。

实参依赖查找使得使用在不同命名空间定义的运算符成为可能。例如：

```c++
#include <iostream>
int main()
{
    std::cout << "测试\n"; // 全局命名空间中没有 operator<<，但 ADL 检验 std 命名空间，
                           // 因为左实参在 std 命名空间中
                           // 并找到 std::operator<<(std::ostream&, const char*)
    operator<<(std::cout, "测试\n"); // 同上，用函数调用记法
 
    // 然而，
    std::cout << endl; // 错误：'endl' 未在此命名空间中声明。
                       // 这不是对 endl() 的函数调用，所以不适用 ADL
 
    endl(std::cout); // OK：这是函数调用：ADL 检验 std 命名空间，
                     // 因为 endl 的实参在 std 中，并找到了 std::endl
 
    (endl)(std::cout); // 错误：'endl' 未在此命名空间声明。
                       // 子表达式 (endl) 不是函数调用表达式
}
```

因为实参依赖查找，在相同命名空间定义的非成员函数和非成员运算符被认为是该类公开接口的一部分（如果它们被 ADL 找到）[[2\]](https://zh.cppreference.com/w/cpp/language/adl#cite_note-2)。

ADL 是在泛型代码中为交换两个对象而建立的手法能成立的原因：

```c++
using std::swap;
swap(obj1, obj2);
```

因为直接调用 [std::swap](https://zh.cppreference.com/w/cpp/algorithm/swap)(obj1, obj2) 不会考虑用户定义的 swap() 函数，它可能在与 obj1 或 obj2 类型之定义相同的空间定义，而仅调用无限定的 swap(obj1, obj2)会在没有用户定义重载时无法调用任何函数。特别是 [std::iter_swap](https://zh.cppreference.com/w/cpp/algorithm/iter_swap) 与所有其他标准库算法在处理[*可交换* *(Swappable)* ](https://zh.cppreference.com/w/cpp/named_req/Swappable)类型时使用此手段。



A simple code example:

```
namespace MyNamespace
{
    class MyClass {};
    void doSomething(MyClass) {}
}

MyNamespace::MyClass obj; // global object


int main()
{
    doSomething(obj); // Works Fine - MyNamespace::doSomething() is called.
}
```

In the above example there is neither a -declaration nor a -directive but still the compiler correctly identifies the unqualified name as the function declared in namespace by applying *Koenig lookup*.`using``using``doSomething()``MyNamespace`

**How does it work?**

The algorithm tells the compiler to not just look at local scope, but also the namespaces that contain the argument's type. Thus, in the above code, the compiler finds that the object , which is the argument of the function , belongs to the namespace . So, it looks at that namespace to locate the declaration of .`obj``doSomething()``MyNamespace``doSomething()`

**What is the advantage of Koenig lookup?**

As the simple code example above demonstrates, Koenig lookup provides convenience and ease of usage to the programmer. Without Koenig lookup there would be an overhead on the programmer, to repeatedly specify the fully qualified names, or instead, use numerous -declarations.`using`

**Why the criticism of Koenig lookup?**

Over-reliance on Koenig lookup can lead to semantic problems, and catch the programmer off guard sometimes.

Consider the example of **[`std::swap`](http://en.cppreference.com/w/cpp/algorithm/swap)**, which is a standard library algorithm to swap two values. With the Koenig lookup one would have to be cautious while using this algorithm because:

```
std::swap(obj1,obj2);
```

may not show the same behavior as:

```
using std::swap;
swap(obj1, obj2);
```

With ADL, which version of function gets called would depend on the namespace of the arguments passed to it.`swap`

If there exists a namespace , and if , , and exist, then the second example will result in a call to , which might not be what the user wanted.`A``A::obj1``A::obj2``A::swap()``A::swap()`

Further, if for some reason both and are defined, then the first example will call but the second will not compile because would be ambiguous.`A::swap(A::MyClass&, A::MyClass&)``std::swap(A::MyClass&, A::MyClass&)``std::swap(A::MyClass&, A::MyClass&)``swap(obj1, obj2)`



## STL实现原理及其实现

STL提供了六⼤组件，彼此之间可以组合套⽤，这六⼤组件分别是:容器、算法、迭代器、 仿函数、适配器（配接器）、空间配置器。

STL六⼤组件的交互关系：  

1. 容器通过空间配置器取得数据存储空间  
2. 算法通过迭代器存储容器中的内容  
3. 仿函数可以协助算法完成不同的策略的变化  
4. 适配器可以修饰仿函数。 

**容器** 

各种数据结构，如vector、list、deque、set、map等，⽤来存放数据，从实现⾓度来看， STL容器是⼀种class template。  

**算法** 

各种常⽤的算法，如sort、find、copy、for_each。从实现的⾓度来看，STL算法是⼀种 function tempalte.  

**迭代器** 

扮演了容器与算法之间的胶合剂，共有五种类型，从实现⾓度来看，迭代器是⼀种将 operator* , operator-> , operator++,operator–等指针相关操作予以重载的class template.。  

所有STL容器都附带有⾃⼰专属的迭代器，只有容器的设计者才知道如何遍历⾃⼰的元素。  

原⽣指针(native pointer)也是⼀种迭代器。 

**仿函数** 

⾏为类似函数，可作为算法的某种策略。从实现⾓度来看，仿函数是⼀种重载了operator() 的class 或者class template  

**适配器** 

⼀种⽤来修饰容器或者仿函数或迭代器接⼜的东西。  

STL提供的queue 和 stack，虽然看似容器，但其实只能算是⼀种容器配接器，因为它们的 底部完全借助deque，所有操作都由底层的deque供应。  

**空间配置器** 

负责空间的配置与管理。从实现⾓度看，配置器是⼀个实现了动态空间配置、空间管理、空 间释放的class tempalte.  

⼀般的分配器的std:alloctor都含有两个函数allocate与deallocte，这两个函数分别调⽤ operator new()与delete()，这两个函数的底层又分别是malloc()and free();但是每次malloc 会带来格外开销（因为每次malloc⼀个元素都要带有附加信息） 

容器之间的实现关系以及分类：

![image-20230404130502922](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20230404130502922.png)

STL的优点 STL 具有⾼可重⽤性，⾼性能，⾼移植性，跨平台的优点。 

1、⾼可重⽤性：  

STL 中⼏乎所有的代码都采⽤了模板类和模版函数的⽅式实现，这相⽐于传统的由函数和类 组成的库来说提供了更好的代码重⽤机会。 

 2、⾼性能：  

如 map 可以⾼效地从⼗万条记录⾥⾯查找出指定的记录，因为 map 是采⽤红⿊树的变体实 现的。  

3、⾼移植性：  

如在项⽬ A 上⽤ STL 编写的模块，可以直接移植到项⽬ B 上。

STL 的⼀个重要特性是将数据和操作分离 

数据由容器类别加以管理，操作则由可定制的算法定义。迭代器在两者之间充当“粘合剂”, 以使算法可以和容器交互运作。 

