---
title: "C++ 基础： 第七章 类"
date: 2025-04-06
categories: 
    - "C++ 学习笔记"
tags: 
    - "C++"
weight: 100
image: "https://s3.bmp.ovh/imgs/2024/11/23/f8eb19dce3c8aec6.jpg"
---
# 第七章 类

## 前言

基本数据类型有时候并不能解决某些特定问题，而通过自定义的类就可以通过理解问题概念，使得程序更加容易编写、调试和修改。

类的基本思想是**数据抽象**（data abstraction）和**封装**（encapsulation）。 数据抽象是一种依赖于**接口**（interface）和**实现**
（implementation）分离的编程（以及设计）技术。类的接口包括用户所能执行的操作；类的实现则包括类的数据成员、负责接口实现的函数体以及定义类所需的各种私有函数。

可以说，封装实现了类的接口和实现的分离。封装后的类隐藏类它的实现细节。

介绍：

- 定义抽象数据类型
- 访问控制与封装
- 类其他特性及作用域
- 构造函数
- 类静态成员

---

## 7.1 定义抽象数据类型

### 设计Sales_data类

其类接口包括：

- 一个isbn成员函数，用于返回对象的ISBN编号
- 一个combine成员函数，用于将一个Sales_data对象加到另一个对象上
- 一个add函数，执行两个Sales_data对象的加法
- 一个read函数，将数据从istream中读入到Sales_data对象中
- 一个print函数，将Sales_data对象的值输出到ostream

> 开发
>
> 优秀的类设计者除了充分了解并实现用户的需求，还应该密切关注哪些有可能使用该类的程序员的需求。
>
> 一个设计良好的类，既要有易于使用的接口，也必须具备高效的实现过程。

> 注意
>
> 定义在类内部的函数是隐式的inline函数

### 定义改进的Sales_data类

定义成员函数

所有类成员必须在类内部声明，但是可以自由选择在类内或类外定义。

this使用

当我们调用成员函数时，实际上是**替**某个对象调用它。例如：

```cpp
/* 某个成员函数 */
std::string isbn() const { return bookNo; }
```

当我们调用isbn成员函数时，返回的bookNo数据成员，那么其隐式的返回的应该是total.bookNo（假设该函数的对象为total）。可以这样理解：使用this是将对象作为一个整体访问，而非直接访问对象的某个成员。

成员函数通过一个名为this的额外隐式参数来访问调用它的那个对象。当我们调用一个成员函数时，用请求该函数的对象地址初始化this。例如：

```cpp
/* 如果调用 */
total.isbn();
/* 编译器将会把total的地址传递给isbn的隐式形参this,等价于下面这个伪代码 */
Sales_data::isbn(&total);
```

> 注意
>
> 在成员函数内部，我们可以直接使用调用该函数的对象的成员，而无须通过成员访问运算符，因为this所指的正是这个对象。任何对类成员的直接访问都被看作this的隐式引用。
>
> 因为this的目的总是指向“这个”对象，所以this是一个常量指针。


引入const成员函数

在成员函数中，存在这种写法：

```cpp
std::string isbn() const {return bookNo};
```

其中，const关键字作用于修改隐式this指针的类型。

在默认情况下，this的类型是指向类类型非常量版本的常量指针。对使用这种const方式的函数成为常量成员函数（const member function）。

我们可以将上述写法想象成：

```cpp
std::string Sales_data::isbn(const Sales_data * const this){
    return this->bookNo;
}
```

类作用域和成员函数

**类本身就是一个作用域，类的成员函数的定义嵌套在类的作用域之内**。

如果在类的外部定义成员函数，其定义必须与它的声明匹配，同时外部定义的成员的名字必须包含它所属的类名。

定义一个返回this对象的函数

### 定义类相关的非成员函数

类作者常常需要定义一些辅助函数，例如：add、read、print等等。这些辅助函数从概念上讲属于是类的接口的一部分，但是实际上其并不属于类。虽然实际上不属于类，但是概念上属于，所以一般将这些非成员函数写在同一个头文件中。

### 构造函数

构造函数（constructor）用于初始化类对象的数据成员，无论何时只要类被创建，就会执行构造函数。

构造函数名字与类名相同，与其他函数不同的是，构造函数没有返回类型；同时类可以包含多个构造函数，但是与重载函数不同的是，构造函数之间必须在参数列表或者参数类型上有所区别；同时构造函数不能被声明为const。

当我们没有显式声明并定义构造函数，那么类将会通过一个**默认构造函数**（default constructor）来控制默认初始化过程。编译器创建的构造函数同时又被称为**合成的默认构造函数**（synthesized default constructor）。但是合成的默认构造函数仅适合非常简单的类，对于一个普通的类，必须定义一个它自己的默认构造函数，因为：

1. 编译器只有在类不包含任何构造函数的情况下才会替我们生成一个默认的构造函数。
2. 含有内置类型或者复合类型成员的类在类的内部初始化时采用默认初始化很有可能其初始化值为未定义的。
3. 有些时候编译器并不能为某些类合成默认的构造函数。

这里，使用一个示例进行分析：

```cpp
SalesData()=default;        /* 默认构造函数，希望这个函数的作用等同于合成默认构造函数 */
SalesData(const std::string &str):_bookNo(str){}
SalesData(const std::string &str,unsigned number,double price):_bookNo(str),_units_sold(number),_revenue(price*number){}
SalesData(std::istream &);
```

`=default`，在C++标准中，使用`SalesData()=default;`方式要求编译器生成构造函数。该定义既可以在声明处，也可以在类外部。与其他函数一致，如果在类内部，则默认为内联方式，如果在类外部，默认不使用内联方式。

构造函数初始值列表

首先我们可以看看后两个定义的构造函数，在冒号和大括号之间存在的部分成为**构造函数初始值列表**（constructor initialize
list）。其负责为新创建的对象的一个或者几个数据成员赋初值。如果某个数据成员被构造函数初始值列表忽略，那么其将会以合成默认构造函数相同的方式初始化。

通常情况下，构造函数使用类内初始值不失为一种好的选择，因为只要这样的初始值存在那么我们就确保了为成员赋予了一个正确的值。但是，如果编译器不支持类内初始值，那么所有的构造函数都应该显式地初始化每一个内置类型的成员。

在类的外部定义构造函数

构造函数没有返回类型，同时在类外部定义构造函数时必须明确构造函数是哪个类的成员。

### 拷贝、赋值和析构

在实际开发中，我们除了需要知道如何对对象进行初始化，还需要知道如何控制拷贝、赋值和销毁对象时发生的行为。

拷贝操作常见于：初始化变量并且采用值传递方式或者返回一个对象；赋值操作见于使用赋值运算符时发生对象的赋值操作；销毁对象，常见的示例比如：当我们将vector对象销毁时，其存储的对象也一并被销毁。

如果，我们不主动控制或者说是定义这些操作，那么编译器将会替我们合成它们。

但是，某些类其实不能够依赖于合成的版本，我们更加希望自定义这些操作。这些知识需要到第13章进行详细描述。

## 7.2 访问控制与封装

在我们之前对Sales_data类进行编写时，我们并没有任何机制强制使用我们在该类中写下的接口，用户可以随时的直达Sales_data类的内部并控制其具体实现细节。也就是说，该类还没有被**封装**。

没有对类进行封装其内部细节相当于直接暴露出来，这是不安全的！。所以，在C++中，我们可以通过使用**访问说明符**（access
specifiers）加强类的封装：

- 定义在**public**说明符之后的成员在整个程序中都是可以被访问的，例如：public定义类的接口
- 定义在**private**说明符之后的成员仅可以被类的成员函数访问，所以private封装了类的部分实现细节。

常用的格式是：

    部分成员函数和构造函数在public说明符之后，而数据成员和作为实现部分的函数则在private说明符之后。

一个类可以存在0个或者多个访问说明符，每个访问说明符指定了接下来的成员的访问级别，其有效范围知道出现下一个访问说明符或者类的结尾为止。

> 补充
>
> 使用class和struct关键字，二者均是用于表示类的定义，其唯一区别就是对默认访问权限的不同。
>
> 类可以在其出现第一个访问说明符之前定义成员，这些成员的访问权限依赖于类定义的方式。如果是struct，则默认权限为public，如果是class，则默认权限为private。

### 友元

如果想要其他类或者函数访问某个类的非公有成员，只需要令其他类或者函数成为这个类的**友元**（friend）即可。

例如，如果想要把某个函数作为这个类的友元，只需要在该类中增加一条以friend关键字开始的函数声明语句即可。


> 注意
>
> 友元的声明仅仅指定了访问的权限，并不是一个通常意义的声明。所以，如果希望类的用户能够调用某个友元函数，那么我们必须在友元声明之外再专门对函数进行一次声明。也就是说，除了在类的内部进行友元声明，在类的外部也需要提供独立的函数声明。
>
> 但是一些编译器允许在没有友元函数的初始声明的情况下调用函数，但是最好的情况下还是提供一个独立的函数声明。

> 建议
>
> 虽然友元只能声明在类定义的内部，但是友元的出现在类内部的位置是不限的。同时友元不是类的成员所以也不受其所在区域的访问控制级别的约束。但是，建议最好是在类定义开始或者结束前的位置声明友元，保证类内部的成员位置的整齐。

> 补充
>
> 封装的好处
>
> 封装的两个重要的优点：
>
> - 保证用户代码不会修改、删除等操作封装对象中的内容
> - 被封装的类，其具体实现细节可以随时改变，而无须调整用户级别的代码
>
> 一旦把数据成员定义成private的，类的作者就可以比较自由地修改数据了。当实现部分改变时，我们只需要检查类的代码本身以确定这次改变有什么影响；也就是说，只要类的接口不变，用户代码就无需改变。
>
> 如果数据是public的，则所有使用了原来数据成员的代码都有可能失效（例如，我们在前面编写的练习7.3，如果我们将Sales_data中的数据成员设定为private的，那么将会报出许多错误，需要一个个修改），
> 这时我们必须定位并且重写所有依赖于老版本实现的代码，之后才能重新使用该程序。
>
> 把数据成员的访问权限设成private还有一个益处，这么做能够防止用户的原因造成数据被破坏。如果我们发现有程序缺陷破坏了对象的状态，则可以在有限的范围内定位缺陷：因为只要实现部分的代码可能产生这样的错误。因此，将差错限制在有限范围内将能够极大地降低维护代码及修正程序错误的难度。

## 7.3 类的其他特性

在前面的章节我们基本上完成了Sales_data类，也了解和学习类一些类的特性，但是还有一些特性还没有从Sales_data类中体现出来。所以，接下来我们将了解类的其他特性，例如：类型成员、类的成员的类内初始值、可变数据成员、内联成员函数、从成员函数返回*this、关于如何定义并使用类类型及友元类的更多知识。

在此之前，为了展示类的其他特性，我们创建一对相互关联的类，分别是Screen和Window_manager

### 类成员再探

Screen类用于表示显示器的一个窗口。每个Screen包含一个保存Screen内容的string成员和三个string::
size_type类型的成员，用于表示光标的位置以及屏幕的宽高。

这里，写下完整的Screen类：

```cpp
class Screen {
 public:
    typedef std::string::size_type pos;

 public:
    Screen() = default;
    Screen(pos height, pos width, char c) : _height(height), _width(width), _contents(height * width, c) {}

 public:
    char    get() const { return _contents[_cursor]; } /* 读取光标处的字符 */
    char    get(pos height, pos width) const;
    Screen& move(pos height, pos width);               /* 移动光标 */

 private:
    /* 光标位置 */
    pos         _cursor = 0;
    pos         _height = 0, _width = 0;
    /* 保存Screen内容 */
    std::string _contents;
};

inline Screen& Screen::move(Screen::pos height, Screen::pos width) {
    pos row = height * _width;
    _cursor = row + width;
    return *this;
}

inline char Screen::get(Screen::pos height, Screen::pos width) const {
    Screen::pos row = height * _width;
    return _contents[row + width];
}
```

类可以自定义某种类型在类中别名（且同样具有访问控制）。

同时，在写Screen类时需要注意：用来定义类型的成员必须先定义后使用。

想要使得我们的Screen类更加实用，则需要添加构造函数使得用户能够自定义屏幕的尺寸和内容，同时需要两个成员函数来负责移动光标和读取给定位置的字符。

#### 内联函数再探

定义在类内中的成员函数都是自动inline的，但这些成员函数规模一般比较小。例如：用来作为返回数据成员值。

同时，除了可以在类内进行隐式内联外，还可以进行显式内联：

```cpp
char    get() const { return _contents[_cursor]; }  /* 隐式内联 */
inline char    get(pos height, pos width) const;    /* 显式内联 */
```

对于某些函数规模较大，我们可以在类外进行成员函数定义，那么可以通过inline关键字修饰类外定义的成员函数。

虽然我们无须在类内和类外同时说明inline，但是这是合法的。同时，建议在类外成员函数的定义处说明inline，这样便于类的理解。

> 建议
>
> 成员函数应该与相应的类定义在同一个头文件中。

#### 成员函数的重载

成员函数的重载与非成员函数一致，只需要函数之间在参数和/或者类型上有所区别即可。例如，在Screen函数中的两个get()函数。

#### 可变数据成员

存在这种情况，我们希望能够修改某个数据成员，即便是在一个const成员函数内。那么我们可以通过关键字**mutable**。

一个**可变数据成员**（mutable data member)永远都不会是const，即便是作为const对象的成员。示例：

```cpp
public:
void    some_member() const;

private:
mutable size_t _access_times; /* Screen的成员函数被调用次数 */

inline void Screen::some_member() const { ++_access_times; }

```

虽然some_member函数是一个const成员函数，但是其仍然能够改变_access_times成员的值。

#### 类数据成员的初始值

在完成Screen类后，我们希望存在一个类用于管理窗口，所以我们设定一个Window_manager类，该类存在一个vector，用于包含Screen。当然在默认情况下，我们希望存在一个默认的Screen。在C++11标准下，最好的方式是将这个默认值声明成一个类内初始值。

```cpp
class Window_manager {
 private:
    std::vector<Screen> screens{Screen(24, 80, ' ')};
};
```

> 注意
>
> 当我们提供一个类内初始值时，必须以符号=或者大括号表示。

### 返回*this的成员函数

```cpp
Screen& Screen::set(Screen::pos height, Screen::pos width, char character) {
    _contents[height * _width + width] = character;
    return *this;
}

inline Screen& Screen::move(Screen::pos height, Screen::pos width) {
    pos row = height * _width;
    _cursor = row + width;
    return *this;
}

myScreen.move(4,0).set('#');
```

在我们定义set()函数时其返回值是调用set的对象的引用。返回引用的函数是左值的，意味着**函数返回的是对象本身而非对象的副本**。假如我们定义的返回类型不是引用，那么move的返回值将会是*this的副本，那么调用set只能临时改变副本，并不能改变myScreen的值。

#### 基于const的重载

通过区分成员函数是否是const的，我们可以进行重载。存在一种情况，非常量版本的函数对常量对象是不可用的，所以我们只能在一个常量对象上调用const成员函数。另一方面，虽然可以在非常量对象上调用常量版本或者非常量版本，但是显然此时非常量版本是一个更好的匹配。

例如，display函数，我们通过定义一个do_display的私有成员，由它负责打印Screen操作（保证安全性），重载display函数，实现常量版本和非常量版本。

```cpp
class Screen{
 public:
    Screen&       display(std::ostream& ostream);
    const Screen& display(std::ostream& ostream) const;

 private:
    void do_display(std::ostream& ostream) const { ostream << _contents; }

}

inline Screen& Screen::display(std::ostream& ostream) {
    do_display(ostream);
    return *this;
}
inline const Screen& Screen::display(std::ostream& ostream) const {
    do_display(ostream);
    return *this;
}

/* ****************** */

Screen myScreen(5,3);
const Screen replica(5,3);
myScreen.set('#').display(cout);        /* 调用非常量版本 */
blank.display(cout);                    /* 调用常量版本 */
``` 

当一个成员调用另一个成员时，this指针在其中隐式地传递。当display调用do_display时，它的this指针隐式地传递给do_display。而当display的非常量版本调用do_display时，他的this指针将隐式得从指向非常量的指针转换为指向常量的指针。

而当do_display完成后，display函数各自返回解引用this所得的对象。在非常量版本中，this指向一个非常量对象，因此display返回一个普通引用；而const成员返回一个常量引用。

也就是说，我们将do_display封装起来，不影响display的工作。

> 建议
>
> 对于公共代码使用私有功能函数
>
> 原因：
>
> - 避免在多出使用同样的代码
> - 随着预期发展，公共代码可能愈加复杂，那么将操作进行模块化处理显得非常有必要
> - 在某些时候，为某些模块添加调试信息，那么在最终版本中，对于调试信息的位置有个明确的概念
> - 通过使用内联的方式，额外的函数调用不会增加任何开销
> - 出于安全考虑，隐藏实现细节

### 类类型

**每个类定义类唯一的类型**。即便类与类之间的成员完全一致，类与类也是不同类型。

根据类的唯一性，我们就可以把类名作为类型的名字使用，从而直接指向类类型。

```cpp
Sales_data obj_1;
class Sales_data obj_2; /* 继承于C语言，等价于上一条 */
```

#### 类的声明

类似于函数的声明与定义可以分离，类也可以仅声明而暂时不定义它。

```cpp
class Screen;   /* 仅声明 */
```

这种声明称作**前向声明**（forward declaration），其仅向程序表明引入了名为Screen的类。对于该类类型来说，它在声明以后定义之前是一个**不完全类型**（incomplete type），表明：我们已知Screen是一个类类型，但是不清楚其包含什么成员。

不完全类型的使用场景非常有限：

- 可以定义指向这种类型的指针或者引用
- 可以声明以不完全类型作为参数或者返回类型的函数，但是不能够定义。

对于编译器来说，只有定义了类类型，那么我们在创建这个类类型对象时才能够知道需要划分多少的存储空间。所以只有**类被定义了，才能够使用引用或者指针访问其成员**。

额外情况：

```cpp
class Link_Screen{
    Screen window;      /* 必须Screen类被定义了，才能声明这种类类型 */
    Link_Screen *next;
    Link_Screen *prev; 
};
```

直到类被定义之后，数据成员才能被声明成这种类类型。也就是说，我们必须首先完成类的定义，然后编译器才能知道存储该数据成员需要多少空间。因为只有当类全部完成后类才算被定义，所以**一个类的成员类型不能是该类自己**。然而，一旦一个类的名字出现后，他就被认为是声明过了，因此**类允许包含指向它自身类型的引用或者指针**。

### 友元再探

在之前，我们仅将定义类相关的非成员函数设定为友元，除此之外，还可以将其他类、其他类的成员函数定义为友元。当然，友元函数可以定义在类内部，这样的函数与上述的一致都是隐式内联的。

#### 类之间的友元关系

类之间的友元关系只需要在需要被访问的类中为提出类设定友元即可。例如，Window_manager类想要访问Screen类，则：

```cpp
class Screen{
    friend class Window_manager;
}
```

这样Window_manager成员就可以访问Screen类的私有部分。

总体上来说，如果一个类指定类友元类，那么友元类的成员函数就可以访问此类包括非公有成员在内的所有成员。

> 注意
>
> 友元关系不存在传递性，也就是说，如果Window_manager类有它自己的友元，但是这些友元没有访问Screen的特权。
>
> 每个类负责控制自己的友元类或者友元函数。

#### 令成员函数作为友元

当把一个成员函数声明为友元时，我们必须明确指出该成员函数属于哪个类。

但是需要注意的是，想要某个成员函数作为友元，我们必须仔细组织程序的结构以满足声明和定义的彼此依赖关系。所以，需要按照一下方法设计程序：

> 假设存在两个类：Window_manager和Screen，其中Window_manager类存在一个Clear函数，Clear函数想要成为Screen类的友元。

1. 首先需要定义Window_manager类，同时必须声明Clear函数，但是这个时候还不能定义它。在Clear函数使用Screen成员前必须先声明Screen类。（注意，Window_manager类必须写在Screen类前面）
2. 定义Screen类，包括对Clear的友元声明（所以建议友元写在类内的最前面）。
3. 定义Clear函数，这个时候该函数才能使用Screen中的成员。

#### 函数重载和友元

如果一个类想要将一组重载函数声明成它的友元，它需要对这组函数中的每一个分别声明（尽管重载函数名一致）。

#### 友元声明和作用域

类和非成员函数的声明不是必须在它们的友元声明之前。当一个名字第一次出现在一个友元声明中时，我们隐式地**假定**该名字在当前作用域中是可见的。然而，友元本身不一定真的声明在当前作用域中。

也就是说，就算是在类内部定义该函数，那么我们也必须在类外提供相应的声明从而使得函数可见。

## 7.4 类的作用域

对类类型成员必须使用作用域运算符进行访问。不论那种情况，跟在运算符后面的名字必须是对应类的成员。

#### 作用域和定义在类外部的成员

**一个类便是一个作用域**，当我们在类的外部使用类成员，该成员将会被隐藏。

当我们使用了类名，那么定义的剩余部分（包括参数列表、函数体）就已经在类的作用域内了，所以不需要再次授权类作用域。因为编译器在处理参数列表之前已经明知我们位于Screen类的作用域之中。

```cpp
inline Screen& Screen::move(pos height, pos width) {    /* pos类型属于Screen */
    pos row = height * _width;  /* _width属于Screen */
    _cursor = row + width;      /* _cursor属于Screen */
    return *this;
}
```

> 注意
>
> 函数的返回类型通常出现在类名之前，所以如果返回类型中使用的名字属于类成员，那么则需要单独为返回类型添加类名。因为如果不添加，返回类型中使用的名字在类的作用域之外。
>
> ```cpp
> class Window_manager{
> public:
>   screen_index addScreen(const Screen &);
> }
> 
> Window_manager::screen_index Window_manager::addScreen(const Screen &screen) {
>   screens.push_back(screen);
>   return screens.size() - 1;
> }
> ```

### 名字查找与类的作用域

寻找与所用名字最匹配的声明的过程称为**名字查找**（name lookup）。截至目前，编写的程序名字查找还比较直截了当：

1. 在名字所在的块中寻找其声明语句，只考虑在名字的使用之前出现的声明。
2. 如果没有找到，继续查找外层作用域。
3. 最终没有找到匹配的作用域，程序报错。

与之不同的成员函数名字查找，类的定义处理过程：

- 首先，编译成员的声明
- 直到类全部可见后才编译函数体

所以，在此类定义的方式下，因为成员函数体直到整个类都可见才会被处理，所以成员函数可以使用类中定义的任何名字。这也是与普通函数不同之处。

#### ❗ 用于类成员声明的名字查找

上述的类定义处理过程这**针对**成员函数**中**
使用的名字，但是如果是声明中使用的名字（包括返回类型、参数列表）那么必须确保其可见。如果某个成员使用了类中尚未出现的名字，则编译器将会在定义该类的作用域中继续查找。

```cpp
typedef double Money;
string bal;
class Account{
public:
    Money balance(){return bal;}
private:
    Money bal;
}
```

在这个示例中，

1. 编译器首先看到balance函数的声明，那么它先在类内寻找对Money的声明。且只会考虑Account类中在使用Money之前的声明，没有找到，那么编译器会到Account类的外层作用域寻找。
2. 找到Money的typedef语句，那么Money将会被认为是double类型作为balance函数的返回类型以及私有成员bal的类型。
3. 因为类的两步处理过程，那么balance函数体将会在整个类可见后处理，那么该函数的return语句返回的是名为bal的成员，其类型为double而非string。

#### 类型名要特殊处理

一般来说，内层作用域可以重新定义外层作用域中的名字（这个我们在学习函数的时候理解过）。然而在类中，如果成员使用了外层作用域中的某个名字，且该名字代表一种类型，则该类不能在之后
**重新定义**该名字。

```cpp
typedef double Money;
string bal;
class Account{
public:
    Money balance(){return bal;}    /* 已经使用类外层作用域的Money */
private:
    typedef double Money;           /* 无法再次定义，即使类型一致 */
    Money bal;
}
```

尽管重新定义类型的名字是一种错误行为，但是编译器并不会负责🤨。所以一些编译器能够顺序通过这个代码，并且忽略有错误的事实。（g++ 11.2.0 会报错😜）

#### ❗ 成员定义中的普通块作用域的名字查找

如题，成员函数使用名字按照一下方式解析：

1. 在成员函数内部查找该名字的声明，只有在函数（注意：不是成员函数）使用之前出现的声明才会被考虑。（这个时候还在成员函数内）
2. 如果在成员函数内没有找到，则在类内继续查找，这个时候类的所有成员都可以被考虑。（这个时候在类内）
3. 如果类内也没有找到该名字的声明，那么在**成员函数*****定义*****之前**的作用域内继续查找。（这个时候在类外）

> 建议
>
> 1. 不建议使用其他成员的名字作为某个成员函数的参数。
>   ```cpp
>    int height;
>    class Test(){
>    public:
>      void test_1(int height){
>         result=height*2;     /* 这里的height使用的是参数声明 */ 
>      }
>    }
>    
>    private:
>      int result=0;
>   ```
>
> 2. 不建议成员函数中的名字隐藏同名的成员。
>   ```cpp
>    int height;
>    class Test(){
>    public:
>      void test_1(int height){
>         result=height*2;     /* 隐藏了同名的成员，使用的是形参 */ 
>      }
>    }
>    
>    private:
>      int result=0;
>      int height=10;
>   ```
>   可以显式的使用this指针强制访问成员或者加上类名的方式，将`result=height*2;`修改为`result=this->height*2`或者`result=Test::height*2`以此绕过上述的名字查找规则
>
> 3. 建议我们在使用成员函数中给形参起一个不会重名的名字。
>
>   ```cpp
>   void test_1(int ht){    /* 起一个不会冲突的形参名 */
>       result=ht*2;     
>       // result=height*2;    /* 使用的是成员height，而非形参ht */
>   }
>   ```

#### 类作用域之后，在外围的作用域中查找

如果编译器并没有在成员函数和类的作用域中找到名字的声明，那么便会从外围的作用域中接着查找。
例如，我们想要Screen外层的height对象：

```cpp
void test_1(int ht){    
   result=::height*2;   /* 通过显式作用域运算符进行请求 */  
}
```

建议，无论是类外名字、类内的数据成员还是成员函数形参名都不要重名，防止隐藏作用域中可能被用到的名字造成认知逻辑模糊。

#### 在文件中名字的出现处对其进行解析

当成员定义在类外时，名字查找的第三步不仅需要考虑**类定义之前的全局作用域中的声明，还需要考虑在成员函数定义之前的全局作用域中的声明**。

```cpp
int height;
class Screen{
public:
    int pos;
    void setHeight(pos);
    pos height=0;       /* 隐藏了外层作用域中的height */
};

Screen::pos verify(Screen::pos);    /* 全局函数声明 */
Screen::setHeight(pos var){         /* 参数var */
   height=verify(var);              /* height是类成员，verify是全局函数 */ 
}
```

全局函数的声明在Screen类的定义之前是不可见的。然而，名字查找的第三步包括类成员函数出现之前的全局作用域，因为verify的声明位于成员函数setHeight的定义之前，所以能够被正常使用。

## 7.5 构造函数再探

在之前我们了解类构造函数的基本知识，接下来将会对构造函数的一些其他功能进行分析与学习。

### 构造函数初始值列表

类似于我么定义变量时（直接进行初始化，而不是先定义、再赋值），对于对象的数据成员，如果没有在构造函数的初始值列表中显式地初始化成员，则该成员将在构造函数体之前执行默认初始化。

```cpp
Test::Test(const string &s){
    _str=s;
};

Test::Test(const string &s):_str(s){};  /* 上述与本表达式一致 */
```

如示例，二者构造函数的定义效果一致，只不过上面那个版本进行赋值，而下面那个版本直接进行初始化。两种版本的区别的深层次影响完全依赖于数据成员的类型。

#### 构造函数的初始值有时必不可少

有时候我们可以忽略数据成员初始化与赋值之间的差距，但是并非总能够这样。如果成员是const或者引用必须进行初始化。类似的，当成员属于某种类类型且该类没有定义默认构造函数时，也必须将这个成员初始化。

❗ 初始化const或者引用类型的数据成员的唯一机会就是通过构造函数进行初始化。

> 建议
>
> 使用构造函数初始值
>
> 在很多类中，初始化和赋值的区别主要是底层效率问题：前者直接初始化数据成员，后者则先初始化再赋值。
>
> 除了效率问题外，一些数据成员必须被初始化。所以建议养成使用构造函数初始值的习惯，这样能够避免某些意想不到的编译错误，特别是遇到有的类含有需要构造函数初始值的成员时。

#### 成员初始化的顺序

构造函数中函数初始值对每个成员只能够出现一次（多次出现没有意义😂），但是值得注意的一点是：在构造函数中成员的初始化*执行顺序* 并不会被限制，但是成员的初始化顺序会与其在类中定义出现的顺序一致。

一般情况下，构造函数初始化列表中初始值的顺序并不会影响到实际的初始化顺序，but（又来🤣）当我们尝试使用一个成员去初始化另一个成员时便会出现问题。

```cpp
class Test{
private:
    int i;
    int j;
public:
    Test(int val):j(val),i(j){};
}
```

在上述例子中，我们想要的顺序是：首先将通过val初始化j，然后通过j初始化i。但是实际上确是：先将通过j初始化i，再通过val初始化j。这就导致一个问题，试图通过未定义的值初始化i。

> 建议
>
> 通过上述示例，相信明白了初始化顺序的意义，所以建议令构造函数的初始化列表遵循类成员声明顺序，同时尽可能地避免使用某些成员初始化其他成员，使其我们不必考虑成员的初始化顺序。

#### 默认实参和构造函数

> 如果一个构造函数为所有参数都提供了默认实参，那么其实际上也就定义了默认构造函数

例如：我们为SalesData类的默认构造函数重写为使用默认实参方式：

```cpp
//SalesData() = default; 
SalesData(std::string s = "") : _bookNo(std::move(s)){}; /* 形式等同默认构造函数 */
```

> 建议
>
> 为构造函数中所有参数提供默认实参应该从实际出发，在某些时候我们就是希望user提供初始化值，这个时候就不应该为参数添加默认实参。

### 委托构造函数

在C++新标准下扩展了构造函数初始化值的功能，使得可以定义**委托构造函数**（delegating constructor）。何谓委托构造函数，其函数可以使用它所属类的其他构造函数执行它自己的初始化过程，也就是说，这类函数可以将自己的一些（或者全部）职责由其他构造函数代劳。（听起来是不是方便了开发者😄）。

```cpp
/* 非委托构造函数 */
SalesData(std::string str, unsigned number, double price) :
    _bookNo(std::move(str)), _units_sold(number), _revenue(price * number) {}
/* 委托构造函数 */
SalesData() : SalesData("", 0, 0.0){};
SalesData(std::istream &istream) : SalesData() { read(istream, *this); } 
```

当一个构造函数委托给另一个构造函数时，受委托的构造函数的初始值列表和函数体被一次执行，最后将控制权返回给委托者，委托者这个时候执行函数体。

### ❓ 默认构造函数的作用

在《C++ primer 》中详细描述了使用细节：

当对象被默认初始化或者值初始化时自动执行默认构造函数。其发生于：

默认初始化下：

- 当我们在块作用域内不使用任何初始值定义一个非静态变量或者数组时
- 当一个类本身含有类类型的成员且使用合成的默认构造函数时
- 当类类型的成员没有在构造函数初始值列表中显式地初始化时

值初始化下：

- 在数组初始化的过程中如果我们提供的初始值数量小于数组的大小时
- 当我们不使用初始值定义一个局部静态变量时
- 当我们通过书写形如T()的表达式显式地请求值初始化时、

在上述情况下，我们必须在类中包含一个默认构造函数用于上述使用。

> 建议
>
> 在编写类的其他构造函数时，习惯性的编写一个默认构造函数

### 隐式的类类型转换

在内置类型中，某些类型存在自动转换机制（或者说是规则），同样，类中也存在，称为类定义隐式转换规则。*如果构造函数只接受一个实参，则它实际上定义了转换为类类型的隐式转换机制*，这种构造函数称为**转换构造函数**（converting constructor）。可以这样说：*通过一个实参调用的构造函数定义了一条从构造函数的参数类型向类类型隐式转换的规则*。

```cpp
Sales_data item;
string str="123";
/* combine: Sales_data &combine(Sales_data &); */
item.combine(str);  /* 执行隐式的类类型转换 */
```

上述中，str为string类型， 编译器类受到str通过构造函数Sales_data(String &)
自动创建一个Sales_data对象，这个对象传递给combine函数。（注意：这个对象是一个临时量）

需要注意的是，*类类型转换仅允许一次转换*，或者这样说，编译器只会自动执行一步的类型转换。

```cpp
Sales_data item;
string str="123";
/* combine: Sales_data &combine(Sales_data &); */
item.combine(str);      /* 合法 */
item.combine("123");    /* 不合法，进行了两步隐式类型转换 */
```

#### 类类型转换不是总有效

类类型转换依赖于用户的选择与使用，在某些时候进行转换可能得到的是一个不存在的东西。同时类类型转换过程中我们会创建一个临时对象，当转换结束该对象也将会被销毁！

#### 抑制构造函数定义的隐式转换

某些情况下，我们希望构造函数不能进行隐式转换，那么可以通过在函数前添加**explicit**关键词。

注意点：

- 关键词explicit仅对具有一个实参的构造函数有效（毕竟只有一个实参的构造函数才能执行隐式转换）
- 仅能够在类内部声明构造函数时使用（类外部的定义不应该重复使用该关键字）

#### explicit构造函数只能够用于直接初始化

发生隐式转换的情况有很多种，其中一种是当我们执行拷贝形式的初始化时，那么此时不能够使用explict构造函数。

```cpp
string str="123";
Sales_data item_1(str);     /* 合法 */
Sales_data item_2=str;      /* 非法 */
```

#### 为转换显式地使用构造函数

在内置类型中我们可以显式地强制执行类型转换（虽然非常不建议使用），同理，在类类型中也可以使用显式地强制转换。

```cpp
item.combine(Sales_data(str));  /* 合法 */
item.combine(static_cast<Sales_data>(cin)); /* static_cast可以强制转换，使之执行explicit构造函数 */
```

#### 标准库中含有显式构造函数的类

在标准库中的一些类含有单参数的构造函数，其中部分函数具有explicit关键字：

- 接受一个单参数的const char*的string构造函数不是explicit。
    ```cpp
    basic_string(const _CharT* __s, const _Alloc& __a = _Alloc())
      : _M_dataplus(_M_local_data(), __a)
      {
        const _CharT* __end = __s ? __s + traits_type::length(__s)
        // We just need a non-null pointer here to get an exception:
        : reinterpret_cast<const _CharT*>(__alignof__(_CharT));
        _M_construct(__s, __end, random_access_iterator_tag());
      }
    ```
- 接受一个容量参数的vector构造函数是explicit的。
  ```cpp
  explicit
      vector(size_type __n, const allocator_type& __a = allocator_type())
      : _Base(_S_check_init_len(__n, __a), __a)
      { _M_default_initialize(__n); }
  ```
  
### 聚合类

何谓聚合类？**聚合类**指的是：

- 所有成员都是public的
- 没有定义任何构造函数
- 没有类内初始值
- 没有基类，也没有virtual函数

这种类的所有成员都可以被访问，并且具有特殊的初始化语法形式称为聚合类。

例如：

```cpp
class Data{   /* 这就是一个聚合类 */
  int     _number;
  string  _statement;
};

/* 特殊的初始化语句 */
Data data={0,"123"};
```

注意：

1. 初始化的顺序必须与声明的顺序一致。
2. 初始值列表中元素个数不能多于类的数据成员数量。
3. 如果初始值列表中元素个数少于类的数据成员数量，则靠后的成员被值初始化。

> 补充
> 
> 显式地初始化类的对象的成员存在三个明显缺点：
> 
> - 要求类的所有成员都是public的
> - 将正确初始化每个对象的每个成员的重任交给了类的用户。如果用户忘记某个初始值或者提供一个不恰当的初始值，那么这样的初始化过程将会冗长乏味且易出错
> - 添加或者删除一个成员后，所有的初始化语句都需要更新


### 字面值常量类

数据成员都是字面值类型的聚合类即为字面值常量类。如果一个类不是聚合类，如果符合下面条件，也是一个字面值常量类：

- 数据成员都必须是字面值类型
- 类必须至少含有一个constexpr构造函数
- 如果一个数据成员含有类内初始值，则内置类型成员的初始值必须是一条常量表达式；或者如果成员属于某种类类型，则初始值必须使用成员自己的constexpr构造函数
- 类必须使用析构函数的默认定义，该成员负责销毁类的对象

> 补充
> 
> 在6.5.2节中解释过constexpr函数的参数和返回值必须是字面值类型。

#### constexpr构造函数

构造函数不能为const，但是字面值常量类的构造函数可以是constexpr函数。

一个字面值常量类必须至少提供一个constexpr构造函数。

constexpr构造函数可可以声明为“=default”形式或者是删除函数形式。否则constexpr构造函数必须既要符合构造函数规则又要符合constexpr函数的要求。

```cpp
class Test{
private:
    bool hw;    /* 硬件错误 */
    bool io;    /* io错误 */
    bool other; /* 其他错误 */
    
public:
    constexpr Test(bool b=true):hw(b),io(b),other(b){}
    constexpr Test(bool h,bool i,bool o):hw(h),io(i),other(o){}
    constexpr bool any(){return hw || io||other;}
    
public:
    void set_io(bool b){io=b;}
    void set_hw(bool b){hw=b;}
    void set_other(bool b){other=b;}
}
```

constexpr构造函数必须初始化所有数据成员，初始值或者使用constexpr构造函数，或者是一条常量表达式。

constexpr构造函数用于生成constexpr对象以及constexpr函数的参数或者返回类型

```cpp
constexpr Test io_sub(false,true,false);

if(io_sub.any())  /* 调试IO */
  cerr<<"print appropriate error messages"<<endl;
constexpr Test prod(false); /* 无调试 */
if (prod.angy())  
  cerr<<"print an error message"<<endl;
```

## 7.6 类的静态成员

我们有一种需求：希望某些类成员与类相关而非与类的对象相关，这个时候我们就可以使用类的静态成员。

#### 声明类的静态成员  

通过在成员的声明前加上关键字static使得其与类关联在一起。当然，静态成员可以是public或者是private的。静态数据成员的类型可以是常量、引用、指针、类类型等。

类的静态成员存在于任何对象之外，对象中不包含任何与静态成员有关的数据。类似的，静态成员函数也不与任何对象绑定在一起，所以其不包含this指针。

#### 使用类的静态成员

我们使用作用域直接访问静态成员：

```cpp
double r;
r=Account::rate();
```

虽然静态成员不属于类的某个对象，但是我们任然可以使用类的对象、引用或者指针来访问静态成员。至于成员函数则可以不用通过作用域运算符就可以直接使用静态成员。

```cpp
class Account{
public:
    void calculate(){ amount +=amount*interestRate;}  /* 可以直接访问静态成员 */
    static void rate(double);
private:
    std::string   owner;
    double        amount;
    static double interestRate;  
    static double initRate();
};

Account a_1;
Account *a_2=&a_1;

double r;
r=Account::rate();    /* 使用作用域运算符来访问静态成员 */
r=a_1.rate();         /* 通过Account的对象或者引用 */
r=a_2->rate();        /* 通过指向Account对象的指针 */
```

#### 定义静态成员

与其他成员函数一致，静态成员函数也可以在类外部定义但是不能重复使用static关键字（也就是说该关键字只能够在类内使用）。

因为静态数据成员不属于类的任何一个对象，所以它们并不是在创建类的对象时被定义的。这意味着它们不是由类的构造函数初始化的。而且一般来说，我们不能在类的内部初始化静态成员。相反的，必须在类的外部定义和初始化每个静态成员。和其他对象一样，一个静态数据成员只能够定义一次。

> 开发
> 
> 想要确保静态对象只被定义一次，最好的办法是把所有静态数据成员的定义与其他非内联函数的定义放在同一个文件中。

#### 静态成员的类内初始化

通常情况下，类的静态成员不应该在类的内部初始化。然而，我们可以为静态成员提供const整数类型的类内初始值，不过要求静态成员必须是字面值常量类型的。

```cpp
class Account{
private:
    static constexpr int period = 30;   /* period是常量表达式 */
}
```

> 建议
> 
> 即使一个常量静态数据成员在类内部被初始化了，通常情况下也应该在类的外部定义一下该成员。

特别的点，静态数据成员的类型可以就是它所属的类类型，但是非静态成员受到限制，只能声明成它所属类的指针或者引用。

```cpp
class Bar{
private:
  static Bar  item_1;   /* 正确 */
  Bar         *item_2;  /* 正确 */
  Bar         item_3;   /* 错误 */ 
}
```

静态成员和普通成员的另外一个区别就是我们可以使用静态成员作为默认实参。非静态数据成员则不行，因为它的值本身属于对象的一部分，这么做将会导致无法真正提供一个对象以便从中获取成员的值，从而导致错误。


## 总结

类允许我们为自己的应用定义新类型，从而使得程序更加简洁易于修改。

类具备两项基本能力：一是数据抽象，二是封装。

类中的构造函数用于控制初始化对象。

类可以定义可变或者静态成员。