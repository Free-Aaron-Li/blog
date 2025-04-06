---
title: "C++ 基础： 第十二章 动态内存"
date: 2025-04-06
categories: 
    - "C++ 学习笔记"
tags: 
    - "C++"
weight: 100
image: "https://s3.bmp.ovh/imgs/2024/11/23/f8eb19dce3c8aec6.jpg"
---
# 第十二章 动态内存

## 前言

在此之前，我们使用的程序中对象都有着严格定义的生存期：

- 全局对象，在程序启动时分配，在程序结束时销毁。
- 局部自动对象，当进入定义所在程序时创建，在离开块时销毁。
- 局部static对象，在第一次使用前分配，在程序结束时销毁。

显然这存在限制，为此C++支持动态**分配对象**。动态分配对象的生存期与它们在哪里创建无关，只有当显式地被释放时，这些对象才会被销毁。

**动态对象的正确释放是编程中极其容易出错的地方**。那么为了更安全释放，标准库定义了两个**智能指针**来管理动态分配的对象。当一个对象应该被释放时，指向它的智能指针可以确保自动地释放它。

> 静态内存：局部static对象、类static数据成员以及定义在任何函数之外的变量
>
> 栈内存：定义在函数内的非static对象
>
> 分配在静态或栈内存的对象由编译器自动创建和销毁。除了静态内存和栈内存，每个程序还拥有一个内存池。称为**自由空间**或**堆**，程序用堆来存储**动态分配**（dynamically allocate）的对象——即那些在程序运行时分配的对象。

## 动态内存和智能指针

在C++中，动态内存的管理通过一对运算符完成：	

- **new**，在动态内存中为对象分配空间并返回一个指向对象的指针，可以选择对对象初始化
- **delete**，接受一个动态对象的指针，销毁该指针指向的对象，并释放与之关联的内存

但是动态内存的使用容易出问题，难以确保在正确时间释放内存，那么就可能出现“内存泄露”、“引用非法指针”等。对此，标准库引入两种智能指针（smart pointer）：

- **shared_ptr**，允许多个指针指向同一个对象
- **unique_ptr**，独占所指向的对象

> 智能指针的行为类似常规指针，区别在于智能指针负责自动释放所指向的对象。上述的两种指针区别在于管理底层指针的方式不同。
>
> 同时，标准库还定义了一个**weak_ptr**的伴随类，其是一个弱引用，指向shared_ptr所管理的对象。三者均定义在`memory`头文件中。

### shared_ptr类

智能指针为模板，在创建时需要提供额外的信息表明指针可以指向的类型。

```cpp
shard_ptr<string> p_1;
shard_ptr<list<int>> p_2;
```
----

|      shared_ptr和unique_ptr都支持的操作      |                      解释                      |
|:-------------------------------------:|:--------------------------------------------:|
| shared_ptr\<T> sp<br/>unique_ptr\<T> up |              空智能指针，可以指向类型为T的对象               |
|                   p                   |          将p用作一个条件判断，若p指向一个对象，则为true          |
|                  *p                   |                解引用p,获得它指向的对象                 |
|               p->member               |                等价于(*p).member                |
|                p.get()                | 返回p中保存的指针。要小心使用，若智能指针释放了其对象，返回的指针所指向的对象也就消失了 |
|        swap(p,q)<br/>p.swap(q)        |                  交换p和q中的指针                   |

|       shard_ptr独有的操作        |                                     解释                                     |
|:---------------------------:|:--------------------------------------------------------------------------:|
| make_shared\<T>(<i>args</i>) |            返回一个shared_ptr，指向一个动态分配的类型为T的对象。使用<i>args</i>初始化此对象             |
|      shared_ptr\<T>p(q)      |                p是shared_ptr q的拷贝。此操作会递增q中的计数器。q中的指针必须能转化为T*                |
|             p=q             | p和q都是shared_ptr，所保存的指针必须能相互转换。此操作会递减p的引用计数，递增q的引用计数；若p的引用计数变为0,则将其管理的原内存释放 |
|         p.unique()          |                     若p.use_count()为1,返回true,否则返回false                      |
|        p.use_count()        |                        返回与p共享对象的智能指针数量；可能很慢，主要用于调试                         |

#### make_shared函数

**最安全的分配和使用动态内存**的方法是调用make_shared标准库函数，此函数在动态内存中分配一个对象并初始化它，返回指向此对象的shared_ptr。

```cpp
shared_ptr<int> p_3=make_shared<int>(42); // 指向一个值为42的int
shared_ptr<string> p_4=make_shared<string>(2,'9'); // 指向一个值为“99”的string
shared_ptr<int> p_5=make_shared<int>(); // 指向一个值初始化的int
```

#### shared_ptr的拷贝和赋值

当进行拷贝或赋值操作时，每个shared_ptr都会记录有多少个其他shared_ptr指向相同的对象：

```cpp
auto p=make_shared<int>(42); // p指向的对象只有p一个引用者
auto q(P); // p和q指向相同对象，此对象有两个引用者
```

我们可以认为每个shared_ptr都有一个关联的计数器，通常称其为**引用计数**（reference count）。无论何时拷贝一个shared_ptr,计数器都会递增。例如，当用一个shared_ptr初始化另一个shared_ptr,或将它作为参数传递给一个函数以及作为函数的返回值时，它所关联的计数器就会递增。当我们给shared_ptr赋予一个新值或是shared_ptr被销毁（例如一个局部的shared_ptr离开其作用域）时，计数器就会递减。

当一个shared_ptr的计数器变为0,它就会自动释放自己所管理的对象：

```cpp
auto p = make_shared<int>(42); // p指向的int只有一个引用者
p = q; // 给p赋值，令它指向另一个地址
	   // 递增q指向的对象的引用计数
	   // 递减p原来指向的对象的引用计数
	   // p原来指向的对象已经没有引用者，会自动释放
```

> 到底是用一个计数器还是其他数据结构来记录有多少指针共享对象，完全由标准库的具体实现来决定。关键是智能指针类能记录有多少个shared_ptr指向相同的对象，并能在恰当的时候自动释放对象。

#### shared_ptr自动销毁所管理的对象

shared_ptr类会通过**析构函数**（destructor）完成销毁工作。shared_ptr的析构函数会递减它所指向的对象的引用计数。如果引用计数为0,shared_ptr的析构函数就会销毁对象，并释放它所占用的内存。

> 如果将shared_ptr存放在一个容器中，而后不再需要某些元素，而只使用其中的一部分，要记得用erase删除不再需要的那些元素

#### 使用了动态生存期的资源的类

程序使用动态内存出于以下三种原因之一：

- 程序不知道需要多少对象
- 程序不知道所需对象的准确类型
- 程序需要在多个对象间共享数据

### 直接管理内存

在前面我们知道C++定义了new和delete进行动态内存的分配和释放。相对于智能指针，这两个运算符管理非常容器出错。自己直接管理内存的类与使用智能指针的类不同，它们不能依赖于类对象拷贝、赋值和销毁操作的任何默认定义，因此，使用智能指针的程序更容易编写和调试。

常见的new：

```cpp
int* pi =new int; // pi指向一个动态分配的、未初始化的无名对象
```

默认情况下，动态分配的对象是默认初始化的，这意味着内置类型或组合类型的对象的值将是为定义的，而类类型对象将用默认构造函数进行初始化。

```cpp
string *p_s=new string;
int *pi=new int;
```

> 默认初始化
>
> 如果定义变量时没有指定初值，则变量被默认初始化（default initialized），此时变量被赋予了“默认值”。默认值到底是什么由变量类型决定，同时**定义变量**的位置也会对此有影响。
>
> 如果是内置类型的变量未被显式初始化，它的值由定义的位置决定。定义于任何函数体之外的变量被初始化为0；定义在函数体内部的内置类型变量将**不被初始化**。一个未被初始化的内置类型变量的值是未定义的。如果试图拷贝或以其他形式访问此类值将引发错误。

对动态分配的对象，我们可以采用直接初始化、传统的构造方式、列表初始化、值初始化的方式进行初始化。

```cpp
int* p_s0=new string; // 默认初始化
int* p_i1=new int(123); // 直接初始化
int* p_s1=new string(10,"1"); // 传统的构造方式
vector<int>* p_v1=new vector<int>{0,1,2,3}; // 列表初始化
string* p_s2=new string(); // 值初始化 
```

> 对于定义了自己的构造函数的类类型（如string）来说，值初始化是没有意义的，但是对内置类型对象来说则有很大的意义：值初始化的的内置类型对象有着良好的值，而默认初始化的对象的值则是未定义的。
>
> 同样，对于类中依赖于编译器合成的默认构造函数的内置类型成员，如果它们未在类内被初始化，那么它们的值也是未定义的。
>
> 所以，出于与变量初始化相同的原因，**对动态分配的对象进行初始化通常是个好主意**

这里就有一个“骚操作”：

```cpp
auto p_1 = new auto(obj); // 从obj那里推断想要分配的对象的类型
auto p_2 = new auto{a,b,c}; // 错误，括号中只能有单个初始化器
```

#### 动态分配的const对象

类似其他任何const对象，一个动态分配的const对象必须进行初始化。对于一个定义了默认构造函数的类类型，其const动态对象可以隐式初始化。

```cpp
const int* P_c_i = new const int(1024);
const string* p_c_s = new const string;
```

#### 内存耗尽

如果自由空间（堆空间）资源耗尽，new表达式将会失败，并抛出`bad_alloc`异常。可以改变new方式阻止其抛出异常：

```cpp
int *p_1 = new (nothrow) int; // 如果分配失败，new返回一个空指针
```

对上述代码中形式的new称为**定位new**（placement new）。定位new表达式允许我们向new传递额外的参数。上述我们向new传递一个由标准库定义的nothrow对象，告诉它不能抛出异常。bad_alloc和nothrow都定义在new头文件中。

#### 释放动态内存

传递给delete的指针必须指向动态分配的内存，或者是一个空指针。释放一块并非new分配的内存，或者将相同的指针值释放多次，其行为都是未定义的。

虽然一个const对象的值不能被改变，但是其本身是可以被销毁的。

> 由内置指针（而不是智能指针）管理的动态内存在会被显式释放前一直都会存在
>
> 所以，动态内存的管理非常容易出错。
>
> 使用new和delete管理动态内存存在三个常见问题：
>
> 1. 忘记delete内存。忘记释放动态内存会导致“内存泄露”问题，因为这种内存永远不可能被归还给自由空间。查找内存泄露错误是非常困难的，因为通常应用程序运行很长时间后，真正耗尽内存后，才能检测到这种错误。
> 2. 使用已经释放掉的对象。即“空指针异常”。
> 3. 同一块内存被释放两次。这种情况下自由空间可能被破坏掉。
>
> 相对于查找和修正这些错误来说，制造出错误简单很多。

当delete一个指针后，其指针值无效，但是机器上指针仍然保存已释放动态内存的地址。即**空悬指针**（dangling pointer）。

未初始化指针的所有缺点空悬指针一个不落。其中“在指针即将离开其作用域前释放掉它所关联的内存”可以作为一种解决方案，如果想要保留指针，需要在delete之后nullptr赋予它，这样清楚地指出指针不指向任何对象。

但这仅仅做出一点小的修复，动态内存的另一个基本问题是存在多个指针指向相同内存的情况，仅解决其中一个指针，其他指针依旧存在问题。

所以，**对于动态内存请务必使用智能指针**！

### shared_ptr和new的结合使用

如果不初始化一个智能指针，那么它就会被初始化为一个空指针。其实，还可以用new返回的指针来初始化智能指针。

```cpp
shared_ptr<double> p1;
shared_ptr<int> p2(new int(42));
```

接受指针参数的智能指针构造函数是explicit的，所以，不能将一个内置指针隐式转换为一个智能指针，必须使用直接初始化形式。相同的，不能将一个智能指针不能隐式转换为内置指针。

```cpp
shared_ptr<int> p1 = new int(1024); //false 内置指针隐式转换为智能指针
```

默认情况下，一个用来初始化智能指针的普通指针必须指向动态内存，因为智能指针默认使用delete释放它所关联的对象。

|           定义和改变shared_ptr的其他方法            |                                               解释                                                |
|:-----------------------------------------:|:-----------------------------------------------------------------------------------------------:|
|            shared_ptr<T> p(q)             |                             p管理内置指针q所指向的对象；q必须指向new分配的内存，且能够转换为T*类型                             |
|            shared_ptr<T> p(u)             |                                 p从unique_ptr u那里接管了对象的所有权：将u置为空                                 |
|           shared_ptr<T> p(q,d)            |                        p接管了内置指针q所指向的对象的所有权。q必须能转换为T*类型。p将使用可调用d来代替delete                        |
|           shared_ptr<T> p(p2,d)           |                           p是shared_ptr p2的拷贝，唯一的区别是p将用可调用对象d来代替delete                           |
| p.reset()<br/>p.reset(q)<br/>p.reset(q,d) | 若p是唯一指向其对象的shared_ptr,reset会释放此对象。若传递了可选的参数内置指针q,会令p指向q,否则会将p置为空。若还传递了参数d,就会调用d而不是delete来释放q |

#### 不要混合使用普通指针和智能指针

shared_ptr可以协调对象的析构，但这仅限于其自身的拷贝（也是shared_ptr）之间。这就是推荐使用make_shared而不是new的原因。可以在分配对象的同时就将shared_ptr与之绑定，从而避免无意中将同一块内存绑定到多个独立创建的shared_ptr上。

```cpp
void process(shared_ptr<int> ptr){ // 值传递，ptr引用计数+1
	// 使用ptr
} // 销毁ptr

shared_ptr<int> p(new int(42)); // 引用计数为1
process(p); // 在process中引用计数为2
int i = *p; // 正确，引用计数为1

int* x(new int(1024)); // 危险：x是一个普通指针
process(x); // 错误，无法隐式转换
process(shared_ptr<int>(x)); // 合法，临时shared_ptr，内存会被释放
int j = *x; //未定义：x是一个空悬指针！
```

当将一个shared_ptr绑定到一个普通指针时，我们就将内存的管理责任交给了这个shared_ptr。一旦这样做了，我们就不应该再使用内置指针来访问shared_ptr所指向的内存了。

> 使用一个内置指针访问一个智能指针所负责的对象是危险的，因为我们无法知道对象何时被销毁！

#### 不要使用get初始化另一个智能指针或为智能指针赋值

智能指针类型定义了一个get函数，用于返回一个指向“智能指针管理的对象”的内置指针。此函数设计的目的是：将指针的**访问权限**传递给代码。**使用get返回的指针的代码不能delete此指针**。

**将另一个智能指针绑定到get返回的指针上是错误的**。

```cpp
shared_ptr<int> p(new int(42)); // 引用计数为1
int* q=p.get(); // 使用q时需要注意，不要让它管理的指针被释放
{
	shared_ptr<int>(q);
} // 当程序块结束，q被销毁，它指向的内存被释放
int x = *p; // 错误，p指向的内存已经被释放
```

p和q指向相同的内存。由于二者相互独立，因此各自引用计数为1.当q所在程序块结束，q被销毁，这会导致q指向的内存被释放。从而导致p变成空悬指针，当意图使用p时，将发生为定义行为。同时，当p被销毁时，这块内存将会被二次delete。

#### 其他shared_ptr操作

可以使用reset来将一个新的指针赋予一个shared_ptr：

```cpp
p = new int(1024); // false
p.reset(new int(1024)); // true
```

与赋值类似，reset会更新引用计数，如果需要的话，会释放p指向的对象。reset成员经常与unique一起使用，来控制多个shared_ptr共享的对象。**在改变底层对象之前，需要检查是否是当前对象仅有的用户**。如果不是，在改变之前需要进行一份拷贝。

```cpp
if(!p.unique())
	p.reset(new string(*p)); // 分配新的拷贝
*p += newVal; // 是唯一的用户，可以改变对象的值
```

### 智能指针和异常

```cpp
void f_v1(){ 
	shared_ptr<int> pt(new int(42));// 分配一个新对象
	// 如果在这里抛出异常，且在f中未被捕获
} // 函数结束时，shared_ptr仍然会自动释放内存

void f_v2(){
	int* p=new int(42); // 动态分配一个新对象
	// 如果在这里抛出异常，且f并未捕获异常
	delete p; // 在此之前退出，内存无法被释放
}
```

智能指针确保了内存在不需要时将其释放，但是如果是内置指针则不一定能够正常释放。

#### 智能指针和哑类

包括所有标准库类在内的很多C++类都定义了析构函数，通过析构函数清理对象使用的资源。但是并不是所有的类都是这样良好的定义的，对于某些类需要用户显式地释放所使用的任何资源。对于这种类称为**哑类**。

在使用哑类的过程中，如果程序员忘记释放资源或者在资源分配和释放之间发生异常，都会导致资源泄露。那么基于这种情况，可以使用智能指针管理动态内存的类似技术来管理哑类。

```cpp
// 假设存在一个网络库，其中connection没有析构函数
struct destination; // 表示需要连接什么
struct connection;  // 连接所需信息
connection connect(destination*); // 打开连接
void disconnect(connection); // 关闭给定的连接

void f(destination &d){
	// 获得一个连接，需要显式关闭它
	connction c= connect(&d); 
	// 使用连接
	// 忘记调用disconnect函数关闭连接，那么f函数结束就无法关闭它。
} // 资源泄露
```

对于上述出现的情况，通过使用shared_ptr修改默认的**删除器**实现对connection的智能管理：

```cpp
void end_connection(connection* p){disconnect(*p));
void f(destination &d){
	connection c=connect(&d);
	shared_ptr<connection> p(&c,end_connection);
} // 当p被销毁时，调用disconnect函数确保连接关闭
```

> 注意：智能指针陷阱
>
> 为了正确使用智能指针，必选坚持的一些基本规范：
>
> - 不使用相同的内置指针初始化（或reset）**多个**智能指针
> - 不delete get()返回的指针
>> get函数仅是为了将智能指针访问内存的权力暂时给予内置指针
> - 不使用get()初始化或reset另一个智能指针
>> get()本质是某个智能指针的内置指针化，如果初始化或reset给另一个智能指针。将导致同一块动态内存被多个智能指针所管理，将导致出现空悬指针、多次delete等问题。
> - 如果你使用get()返回的指针，记住当最后一个对应的智能指针销毁后，你的指针就变为无效了
>> get()仅仅作为智能指针向内置指针传递权力的工具，当权力的源头失效了自然所对应的内置指针就变为空悬指针，需要将内置指针赋予nullptr
> - 如果你使用智能指针管理的资源不是new分配的内存，记住传递给它一个删除器
>> 智能指针默认调用delete操作销毁内存

### unique_ptr

一个“unique_ptr”“拥有”它所指向的一个对象。

|             unique_ptr特有操作              |                                 解释                                  |
|:---------------------------------------:|:-------------------------------------------------------------------:|
| unique_ptr<T> u1<br/>unique_ptr<T,D> u2 | 空unique_ptr,可以指向类型为T的对象。u1会使用delete来释放它的指针；u2会使用一个类型为D的可调用对象来释放它的指针 |
|          unique_ptr<T,D> u(d)           |               空unique_ptr,指向类型为T的对象，用类型为D的对象d代替delete               |
|                u=nullptr                |                           释放u指向的对象，将u置为空                            |
|                u.release                |                        u放弃对指针的控制权，**返回指针**并将u置为空                        |
|                u.reset()                |                              释放u指向的对象                               |
|     u.reset(q)<br/>u.reset(nullptr)     |                     如果提供了内置指针q,令u指向这个对象；否则将u置为空                     |

无法使用类似make_shared的标准库函数返回一个unique_ptr。所以当定义一个unique_ptr时，需要绑定到一个new返回的指针上。

```cpp
unique_ptr<int> p(new int(42)); 
```

由于unique_ptr拥有它指向的对象，所以unique_ptr不支持普通的拷贝或赋值操作

```cpp
unique_ptr<int> p1(new int(42));
unique_ptr<int> p2(p1); // false
unique_ptr<int> p3;
p3=p1; // false
```

但是可以通过release或reset实现指针控制权的转移：

```cpp
// 将控制权从p1转移到p2
unique_ptr<int> p2(p1.release()); // release将p1置为空，且返回指针
unique_ptr<int> p3(new int(42));
// 将控制权从p3转移到p2
p2.reset(p3.release()); // reset释放p2原本指向内存
```

调用release函数会切断unique_ptr和它原本管理的对象间的联系。release返回的指针（一般为内置指针）通常被用来初始化另一个智能指针或给另一个智能指针赋值。这仅仅是简单的权力交接，但是，如果不用一个智能指针保存release返回的指针，则需要负责资源的释放。

不能拷贝unique_ptr规则的一个例外：可以拷贝或赋值一个将要被销毁的unique_ptr：

```cpp
unique_ptr<int> clone(int p){
	return unique_ptr<int>(new int(p)); // 从int*创建一个unique_ptr<int>
}

unique_ptr<int> clone(int p){
	unique_ptr<int> ret(new int(p));
	// ...
	return ret;
}
```

编译器知道要返回的对象将要被销毁，所以在这种特殊的情况下，编译器执行一种特殊的“拷贝”。

#### 向unique_ptr传递删除器

类似于shared_ptr,unique_ptr默认情况下用delete释放其对象，也可以重载unique_ptr中的默认删除器。但是,unique_ptr管理删除器的方式与shared_ptr不同。

重载一个unique_ptr中的删除器会影响到unique_ptr类型以及如何构造该类型的对象。所以必须在尖括号中unique_ptr指向类型之后提供删除器类型。在创建或reset一个这种unique_ptr类型的对象时，必须提供一个指向类型的可调用对象（删除器）：

```cpp
unique_ptr<objT,delT> p(new objT,fun);
```

例如：

```cpp
void f(Destination &d){
	Connection c = connect(&d);
	unique_ptr<Connection, decltype(end_connection)*> p (&c,end_connection);
}
```

### weak_ptr

weak_ptr是一种不控制所指向对象生存周期的智能指针，它指向由一个shared_ptr管理的对象。其具有**“弱”共享对象**的特点：**绑定**到某个shared_ptr上并不会改变该shared_ptr引用计数；即使weak_ptr还指向某个对象，当指向该对象的shared_ptr被销毁时，该对象也会被释放。并不会因为weak_ptr关系而不释放。

|     weak_ptr      |                           解释                           |
|:-----------------:|:------------------------------------------------------:|
|   weak_ptr<T> w   |                  空weak_ptr可以指向类型为T的对象                  |
| weak_ptr<T> w(sp) |       与shared_ptr指向相同对象的weak_ptr。T必须能转换为sp指向的类型        |
|        w=p        |         p可以是一个shared_ptr或一个weak_ptr。赋值后w与p共享对象         |
|     w.reset()     |                         将w置为空                          |
|   w.use_count()   |                  与w共享对象的shared_ptr的数量                  |
|    w.expired()    |           若w.use_count()为0,返回true,否则返回false            |
|     w.lock()      | 如果expired为true，返回一个空shared_ptr；否则返回一个指向w的对象的shared_ptr |

由于weak_ptr“弱”共享的特性，存在weak_ptr指向内存已经被释放情况。所以，weak_ptr不能直接访问对象，需要调用lock函数检查weak_ptr指向对象是否存在，如果存在返回指向共享内存的shared_ptr。

## 12.2 动态数组

C++与标准库提供两种一次分配一个对象数组的方法。一种采用new表达式，可以分配并初始化一个对象数组；另一种采用allocator类，允许将分配和初始化分离。

> 注意：尽管提供直接访问动态数组的方法，但是大多数应用应该使用标准库容器而不是动态分配数组。

### new和数组

分配数组必须指明对象元素的数量：

```cpp
int *p_1 = new int[get_size()];

// 或者
typedef int arrT[42];
int *p_2 = new arrT;
```

分配一个数组会得到一个元素类型的指针。同时由于分配的内存并不是数组类型，所以不能调用begin或者end。

初始化动态分配对象的数组，使用new分配的对象，不论是单个分配还是数组中的，都是默认初始化。可以对数组中的元素进行值初始化：

```cpp
int *p_3 = new int[10]();
```

也可以提供一个元素初始化器的花括号列表：

```cpp
string *p_4 = new string[10]{"a","b","c",string(3,"d")};
```

如果初始化器数目小于元素数目，剩余元素进行值初始化；如果大于，则new表达式失败，不会分配任何内存且抛出bad_array_new_length（定义在new中）的异常。

**动态分配一个空数组是合法的**，但是指向该空数组的指针不能解引用。

释放动态数组则采用特殊形式的delete：

```cpp
delete [] *p_5; // 空括号是必需的，其指示编译器此指针指向一个对象数组的第一个元素
```

该delete语句将销毁p_5指向的数组中元素并释放内存。数组中的元素按照逆序销毁。

> 如果在delete一个数组指针时忘记方括号或者delete一个单一对象的指针时使用了方括号都是未定义的。编译器很可能不会给出警告。程序可能在执行过程中在没有任何警告的情况下行为异常。

#### 智能指针和动态数组

标准库提供可以管理new分配数组的unique_ptr版本：

```cpp
unique_ptr<int[]> up(new int[10]);
up.release(); // 自动使用delete[]销毁其指针
```

由于unique_ptr指向的是一个数组而不是对象，所以不能使用点和箭头成员运算符。可以使用下标运算符来访问数组中的元素：

```cpp
for(size_t i = 0; i!=10; ++i）
	up[i]=i;
```

---


|   指向数组的unique_ptr    |               解释                |
|:--------------------:|:-------------------------------:|
|  unique_ptr<T[]> u   |     u可以指向一个动态分配的数组，数组元素类型为T     |
| unique_ptr<T[]> u(p) | u指向内置指针p所指向的动态分配的数组。p必须能转换为类型T* |
|         u[i]         |   返回u拥有的数组中位置i处的对象（u必须指向一个数组）   |

shared_ptr不支持直接管理动态数组：

1. 需要自定义删除器
2. 为定义下标运算符

所以，如果使用shared_ptr管理动态内存数组，需要这样：

释放数组：

```cpp
shared_ptr<int> sp(new int[10],[](int *p){delete[] p;});
sp.reset(); // 使用提供的lambda释放数组
```

访问数组元素：

```cpp
for(size_t i=0; i!=10;++i)
	*(sp.get()+i = i;
```

### allocator类

new将内存分配和对象构造组合在一起，delete操作将对象析构和内存释放组合在一起。这带来一定的局限性。当分配单个对象时，通常希望上述操作合并起来，因为我们几乎肯定知道对象有什么值。但是当分配一大块内存时，我们通常需要事先计划，等到真正执行对象时才执行创建操作。

标准库提供allocator类，其将内存分配和对象构造分离，通过提供一种类型感知的内存分配方法，其分配的内存是原始的、未构造的。

allocator本身是一个模板，当分配内存时，它会根据给定的对象类型来确定恰当的内存大小和对齐位置。

```cpp
allocator<string> alloc;
auto const p=alloc.allocate(n); // 分配n个未初始化的string
```

|     标准库allocator类及其算法      |                                                          解释                                                          |
|:--------------------------:|:--------------------------------------------------------------------------------------------------------------------:|
|       allocator<T> a       |                                         定义了一个名为a的allocator对象，它可以为类型为T的对象分配内存                                         |
|       a.allocate(n)        |                                              分配一段原始的、未构造的内存，保存n个类型为T的对象                                              |
|     a.deallocate(p,n)      | 释放从T*指针p中地址开始的内存，这块内存保存了n个类型为T的对象；p必须是一个先前由allocate返回的指针，且n必须是p创建时所要求的大小。在调用deallocate之前，用户必须对每个在这块内存中创建的对象调用destroy |
| a.construct(p,<i>args</i>) |                           p必须是一个类型为T*的指针，指向一块原始内存；<i>arg</i>被传递给类型为T的构造函数，用来在p指向的内存中构造一个对象                           |
|        a.destroy(p)        |                                              p为T*类型的指针，此算法对p指向的对象执行析构函数                                              |

allocator分配的内存是未构造的，按需要在此内存中构造对象。

```cpp
auto q = p; // q指向最后构造的元素之后的位置
alloc.construct(q++); // *q为空字符串
alloc.construct(q++,2,'c'); // *q为cc
alloc.construct(q++,"hi"); // *q为hi
```

对未构造的对象使用原始内存是错误的！

当用完对象后，必须使用每个构造的元素调用destroy销毁它们。当元素被销毁，那么可以选择重新使用这部分内存也可以通过调用deallocate释放内存。

```cpp
while(q!=p)
	alloc.destroy(--q);
	
alloc.deallocate(p,n);
```

在标准库中还为allocator类定义了两个伴随算法，可以在未初始化内存中创建对象。

|         allocator算法          |                                解释                                |
|:----------------------------:|:----------------------------------------------------------------:|
|  uninitialized_copy(b,e,b2)  | 从迭代器b和e指出的输入范围中拷贝元素到迭代器b2指定的未构造的原始内存中。b2指向的内存必须足够大，能容纳输入序列中元素的拷贝 |
| uninitialized_copy_n(b,n,b2) |                   从迭代器b指向的元素开始，拷贝n个元素到b2开始的内存中                   |
|  uninitialized_fill(b,e,t)   |                 在迭代器b和e指定的原始内存范围中创建对象，对象的值均为t的拷贝                 |
| uninitialized_fill_n(b,n,t)  |        从迭代器b指向的内存地址开始创建n个对象。b必须指向足够大的未构造的原始内存，能够容纳给定数量的对象        |

例如：

```cpp
// vi为int的vector
allocator<int> alloc;
auto p = alloc.allocate(vi.size()*2);
auto q = uninitialized_copy(vi.begin(),vi.end(),p);
uninitialized_fill_n(q.vi.size(),42);
```

## 12.3 使用标准库：文本查询程序

实现一个简单的文本查询程序：允许用户在一个给定的文件中查询单词，查询结果为单词在文件中出现的次数及其所在行的列表。

例如：

```cpp
Enter the file name to look up: QuotesInEnglish.txt
Enter word to look for, or 'q' to exit: the
"the" occurs 17 times:
(line 5) 2、Variety is the spice of life.
(line 17) 5、Doubt is the key to knowledge.
(line 49) 13、Life is the art of drawing sufficient conclusions form insufficient premises.
(line 65) 17、Life is a great big canvas, and you should throw all the paint on it you can.
(line 77) 20、The wealth of the mind is the only wealth.
(line 105) 27、Wealth is the test of a man's character.
(line 109) 28、The best hearts are always the bravest.
(line 117) 30、There's only one corner of the universe you can be sure of improving, and that's your own self.
(line 125) 32、Death comes to all, but great achievements raise a monument which shall endure until the sun grows old.
(line 133) 34、Suffering is the most powerful teacher of life.
(line 149) 38、A fall into the pit, a gain in your wit.
(line 153) 39、A guest should suit the convenience of the host.
(line 161) 41、All rivers run into the sea.
(line 169) 43、An apple a day keeps the doctor away.
(line 181) 46、Behind the mountains there are people to be found.

Enter word to look for, or 'q' to exit: q

进程已结束，退出代码为 0
```

### 文本查询程序的设计

|[源代码](https://github.com/Free-Aaron-Li/Cpp_Study-program/blob/master/Cpp_Primer_Studying/PartII_STL/ChapterTwelve_DynamicMemory/src/main.cpp#L36C9-L36C9)|

**开发一个程序在设计上的一种好方法是列出程序的操作**，了解需要哪些操作会帮助我们分析出需要什么样的数据结构。

- 当程序读取输入文件时，它必须**记住单词出现的每一行**。因此，程序需要**逐行读取**输入文件，并将每一行分解为独立的单词。
- 当程序生成输出时，
	- 它必须能提取每个单词所**关联的行号**
	- 行号必须按照升序出现且**无重复**
	- 它必须能打印给定行号中的文本

对此，可以这样实现：

- 使用一个vector<string>来保存整个输入文件的一份拷贝。输入文件中的每行保存为vector中的一个元素。当需要打印一行时，可以用行号作为下标来提取行文本。
- 使用一个istringstream来将每行分解为单词
- 使用一个set来保存每个单词在输入文本中出现的行号。这保证了每行只出现一次且行号按照升序保存
- 使用一个map来将每个单词与它出现的行号set关联起来。这样我们就可以方便地提取任意单词的set
- 为了实现类之间共享数据，使用shared_ptr来反映数据结构中的共享关系

## 总结

分配动态内存的同时需要负责释放内存，在释放时存在空悬指针、多次delete等问题。智能指针是解决动态内存的好方式，在使用动态指针时，根据计数器会自动释放内存。

现代C++程序应尽可能使用智能指针。