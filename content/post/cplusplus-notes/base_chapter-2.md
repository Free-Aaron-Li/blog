---
title: "C++ 基础： 第二章 变量和基本类型"
date: 2025-04-06
categories: 
    - "C++ 学习笔记"
tags: 
    - "C++"
weight: 100
image: "https://s3.bmp.ovh/imgs/2024/11/23/f8eb19dce3c8aec6.jpg"
---

# 第二章 变量和基本类型

## 前言

数据类型是程序的基础：它告诉我们数据的意义以及我们能在数据上执行的操作。

---

## 2.1 基本内置类型

C++定义了包括**算术类型**（arithmetic type）和**空类型**（void）在内的基本数据类型。

### 2.1.1 算术类型

算术类型分为两类：整型和浮点型。

具体分类：

|     类型      |    含义     |  最小容量   |
|:-----------:|:---------:|:-------:|
|    bool     |   布尔类型    |   未定义   |
|    char     |    字符     |   8位    |
|   wchar_t   |    宽字符    |   16位   |
|  char16_t   | Unicode字符 |   16位   |
|  char32_t   | Unicode字符 |   32位   |
|    short    |    短整型    |   16位   |
|     int     |    整型     |   16位   |
|    long     |    长整型    |   32位   |
|  long long  |    长整型    |   64位   |
|    float    |  单精度浮点数   | 6位有效数字  |
|   double    |  双精度浮点数   | 10位有效数字 |
| long double |  扩展精度浮点数  | 10位有效数字 |

> 基本数据类型char，其空间应确保可以存放机器基本字符集中任意字符对应的数字值。即一个char的大小和一个机器字节一样。

除去布尔型和扩展的字符型之外，其他整型可以划分为**带符号的**（signed）和**无符号的**（unsigned）两种。

> 其中char类型是个特殊存在，其存在三种类型：char、signed char和unsigned char。但是字符的表现形式仅有两种，
> 根据编译器决定类型char实际上应该表现为那种形式（无符号/有符号）。同时由于其表现形式由编译器决定，存在唯二性，
> 所以在算术表达式中不能使用char类型。（实在要使用，**必须规定其具体类型**）

### 2.1.2 类型转换

对象的类型定义了对象能包含的数据和能参与的运算。**类型所能表示的值的范围决定了转换的过程**。


> 当赋给无符号类型一个超出它表示范围的值时，结果时初始值对无符号类型表示数值总数取模后的余数。例如：`unsigned char c = -1 //假设char占8比特，c的值为255`,由于8比特大小的无符号char可以表示0~255内的值，其实际结果便是该值对256取模后得到的余数255。
>
> 当赋给带符号类型一个超出它范围的值时，结果时**未定义的**（undefined），存在一下可能：程序可能正常工作、可能程序崩溃
> 可能产生未知的数据（即垃圾数据）

> PS:
>
> **程序应当尽量避免依赖于实现环境的行为、避免无法预知的行为**

### 2.1.3 字面值常量

形如 12 的值被称为**字面值常量**（literal）。每个字面值常量都对应一种数据类型，其形式和值决定了它的数据类型。

#### 整型和浮点型字面值

整型字面值具体的数据类型由其符号和值决定。

```cpp
20 /* 十进制 */
024 /* 八进制 */
0x24 /* 十六进制 */
```

> 默认，十进制为带符号数，剩下两种都有可能。

#### 字符和字符串字面值

字符串字面值的类型实际是由常量字符构成的数组。编译器在每个字符串的结尾处添加一个空字符“\0”，因此，实际长度比内容多1。

#### 转义字符

C++语言规定的转义序列包括：

```markdown
换行符     \n          横向制表符       \t          报警（响铃）符     \a
纵向制表符  \v          退格符          \b          双引号           \"
反斜杠     \\          问号            \?          单引号           \'
回车符     \r          进纸符          \f
```

通过为字面值添加前缀和后缀，可以改变整型、浮点型和字符型字面值的默认类型。

#### 布尔字面值和指针字面值

true和false是布尔类型的字面值，nullptr是指针字面值。

---

## 2.2 变量

变量提供一个具名的、可供程序操作的**存储空间**
。C++中每个变量都有数据类型，数据类型决定着变量所占内存空间的大小和布局方式、该空间能存储的值的范围，以及变量能参与的运算。对于C++程序员来说，“变量（variable）”和“对象（object）”一般可以互换使用。

### 2.2.1 变量定义

变量定义的基本格式：**类型说明符**（type specifier），一个或多个变量名组成的列表。

> PS:
>
> 对象（object），指一块能存储数据并具有某种类型的内存空间。
> 一部分人认为：仅在与类有关的场景下才能使用“对象”这个词；一部分人认为应该将把命名的对象和未命名的对象区分开将命名了的对象叫做变量；还有一部分人仍未应该将能够被程序修改的数据称为对象，而只读程序称为值（value）

#### 初始化

当对象在创建时获得一个特定的值，称为这个对象被**初始化**（initialized）了。

> 初始化并不是赋值，虽然都是通过=方式，初始化的含义是创建变量时赋予其一个初始值，而赋值的含义是把对象的当前值擦除，而以一个新值来代替。

#### 列表初始化

```cpp
int a=0;
int a={0};
int a{0};
int a(0);
```

上述四种均可为a赋值。

#### 默认初始化

定义于函数体内的内置类型的对象如果没有初始化，则其值未定义。类的对象如果没有显式地初始化，则其值由类确定。

> 未初始化的变量含有一个不确定的值，使用未初始化的变量的值绝对是一种错误且严重的编程行为（此行为很难调试）
> 
> 究其原因：①、编译器虽然会对部分未初始化的变量提出警告，但其本身并未要求检查此类错误；②、使用未初始化的变量会带来无法预测的结果，形成黑箱！
>
> 强烈建议：对**每一个内置类型的变量都应该初始化**，虽无强制要求，但是为了程序安全很有必要。

### 2.2.2 变量声明和定义的关系

为了允许把程序拆分成多个逻辑部分，C++支持**分离式编译**（separate compilation）机制，在此机制下程序可以分割为若干个文件，每个文件皆可独立编译。

为了实现该机制，那么必须能够存在能够在文件共享代码的方法。例如：std::count O(∩_∩)O哈哈~

在此之下，C++语言将声明和定义区分开来，**声明**（declaration）使得名字为程序所知，一个文件若想在别处使用该名字则必须包含对此名字的声明。而**定义**（definition）则负责创建与名字关联的实体。

变量声明规定了变量的类型和名字。（与定义区别在于：定义还多一项为变量申请存储空间，也可能赋予一个初始值的功能）

如果想要声明一个变量而非定义该变量，就会在变量前添加关键字**extern**，同时不要显式的初始化变量。

```cpp
extern int i; //声明i而非定义i
int j; //声明且定义j
```

任何包含了显式初始化的声明即称为定义。同时，在函数体内部，如果试图初始化一个已经被关键词extern标记的变量，将引发错误。同时在函数体外这样做，将会出现警告。

> Note:
>
> 变量能且只能被定义一次，但是可以被多次的声明

### 2.2.3 标识符

**标识符**（identifier）由字母、数字和下划线组成。其必须是字母或下划线开头。

其变量的命名规范

变量命名有许多约定俗成的规范，下面这些规范能够有效地提高提高程序的可读性：

- 标识符要**能体现实际含义**
- 变量名一般用小写字母，比如index，不要使用Index或者INDEX
- 用户自定义的类名一般以大写字母开头，如Search()
- 如果标识符由多个单词组成，则单词间应该有明显区分，如`Student_name`或者`StudentName`,不可`Studentname`

### 2.2.4 名字的作用域

不论在程序的那个位置，使用到的每一个**名字**都会指向其一个实体。然而，同一名字出现在不同的位置，也有可能指向不同实体。

名字的有效区域始于名字的声明语句，以声明语句所在的作用域末端为结束。

- 全局作用域（global scope）定义在函数体之外，声明后在整个程序的范围内都可以使用。
- 块作用域（block scope）定义于函数体之内，在其函数体内可以使用。

> Suggestion
>
> 当第一次使用变量时再去定义它。原因①：有助于更容易找到变量的定义；原因②：当定义位置与其第一次被使用位置很近时，会赋予一个更加合理的初始值。

#### 嵌套作用域

- 内层作用域（inner scope）
- 外层作用域（outer scope）

其被嵌套的作用域是相对的，在作用域中声明了某个名字，其所嵌套的所有作用域都可以使用该名字，同时，也允许在其嵌套作用域中重新定义外层作用域的名字。

> Warning
>
> 如果函数使用到全局变量，不宜再次定义与其名字相同的局部变量。

---

## 2.3 复合类型

**复合类型**(compound type)是指基于其他类型定义的类型。C++语言中有几种复合类型，这里介绍：引用和指针。

### 2.3.1 引用

> Note
>
> C++ 11中新增了一种引用：所谓的“右值引用（rvalue reference）”，这种引用主要用在内置类中。严格来说，我们平常使用的术语“引用（reference）”其实是指“左值引用（lvalue
> reference）”。

**引用**（reference）作用就是为对象起别名。

> PS
>
> 引用其本质上应该是指针常量（即该形式：数据类型* const 变量名），其限制为：能够更改指针对应的值，但是不能修改对应的地址值。所以：引用**必须初始化**。
>
> 同样的道理，对引用进行的所有操作都是在为与之绑定的对象进行操作，**其引用并不是一个对象**，无法定义引用的引用。也就不能使用这样的操作：与字面值绑定或者与某个表达式绑定。

### 2.3.2 指针

**指针**（pointer）是“指向（point to）”另一种类型的复合类型。

> 其与引用类似，指针也实现对其他对象的间接访问。但是指针与引用又有很多地方不相同：
>
> ①、指针本身就是一个对象，允许对指针的赋值和拷贝，并且在指针的生命周期中可以先后指向多个不同的对象。
>
> ②、指针无须定义时赋初值，但同时和其他内置类型一样，在块作用域内定义指针如果没有初始化，也将拥有一个不确定值。

#### 获取对象的地址

**指针存放某个对象的地址**，想要获取地址，需要使用**取地址符**（操纵符&）

> Warning
>
> ①、因为引用不是对象，所以不能定义指向引用的指针
>
> ②、所有指针的类型都要和它所指向的对象严格匹配，因为在声明语句中指针的类型实际上被用来指定它所指向对象的类型

#### 指针值

指针值（即地址值）应该属于下列4种情况之一：

1. 指向一个对象
2. 指向紧邻的对象所占空间的下一个位置
3. 空指针
4. 无效指针，即上述情况之外的其他值

> 任何试图访问无效指针的值都会引发错误，且编译器并不会检查此类错误，**程序员应该清楚任意给定的指针的有效性**；虽然2、3两种情况是有效的，但是试图访问其指针指向的对象同样不被允许。

#### 利用指针访问对象

如果指针指向一个对象，允许使用**解引用符**（操纵符*）来访问该对象。

> Warning
>
> 解引用操作仅适用于那些明确指向对象的有效指针。

#### 空指针

空指针（null pointer）不指向任何对象。

> Suggestion
>
> 使用未经初始化的指针是引发运行时错误的一大原因。
>
> 建议初始化所有指针，并且在可能的情况下，尽量等到定义了对象后再定义指针。实在不行，便将指针定义为nullptr，使得编译器明白该指针并未指向任何对象。

#### 赋值和指针

指针和引用都能提供对其他对象的间接访问，然而在具体实现细节上二者有很大不同，其中最重要的一点就是引用本身并非一个对象。一旦定义了引用，就无法令其再绑定到另外的对象，之后每次使用这个引用都是访问它最初绑定的那个对象。
但是对于指针，则没有这种限制，和其他任何变量（只要不是引用）一样，给指针赋值就是令它存放一个新地址，从而指向一个新对象。

#### 其他指针操作

1. 若指针值为0，则条件取false，其他情况都返回true。
2. 对于两个类型相同的合法指针，可以进行比较，比较结果为布尔类型。其比较对象是两个指针的存放的地址值。

#### void* 指针

**void***是一种特殊的指针类型，可用于存放任意对象的地址，但是对存放何种对象类型并不了解。

由于无法知道指针指向的对象是什么类型，所以就不能直接操作void*指针所指向的对象。

概括来讲：void*就仅仅只是内存空间，无法读取内存空间所存的对象。

### 2.3.3 理解复合类型的声明

前面说过：变量的定义包括一个基本数据类型（base type）和一组声明符。在同一条定义语句中，虽然基本数据类型只有一个，但是声明符的形式却可以不同。例如：

```cpp
int i = 1024, *p = &i, &r = i;  /* i是int类型的数，p是int类型指针，r是int类型引用 */
```

可能觉得基本数据类型和类型修饰符有什么关系，其实**后者不过是声明符的一部分**。

> Warning
>
> 有一种错误的观点认为：在定义语句总，类型修饰符作用于本次定义的全部变量。其造成错误看法的原因很多，其中一种是：
>
> ```cpp
> int* p1,p2;
> ```
> 在此观点下，会认为p1和p2是两个都是int类型的指针变量。其实只有p1是，p2是int类型变量。其*仅仅修饰了p1。
>
> 所以，为了强调标量具有复合类型，建议将修饰符和变量标识符写在一起。

#### 指向指针的引用

虽然，引用本身不是对象，无法使用指针指向引用，但是指针本身是对象，可以使用引用指向指针。

```cpp
int i = 1024;
int *p;
int *&r = p;    /* r是对指针p的引用，建议从右向左看*&r */

r = &i;         /* r本身是引用指针p，所以这里将i的地址赋值给指针p */
*r = 0;         /* 指针p指向的i值赋予为0*/
```

> Suggestion
>
> 面对一条较为复杂的指针或者引用的声明语句时，从右向左阅读有助于弄清楚它的真实含义。

---

### 2.4 const限定符

有时候我们并不希望定义一个变量，它的值无法改变。这个时候，我们就可以通过const对此类变量加以限定。

因为const对象一旦完成创建就无法修改其值，所以**const对象必须初始化**。

#### 初始化和const

const类型的变量与普通变量并没多大区别，其主要限制在于只能在const类型的对象上执行不改变内容的操作。在此限制下还有一种是初始化，如果利用一个对象取初始化另一个对象，则它们是不是const都无关紧要。

```cpp
int i =1024;
const int ci = i;   /* ci的常量特征仅仅在执行改变ci的操作时才发挥作用，这里仅仅对ci进行初始化操作 */
int j = ci;
```

#### 默认情况下，const对象仅在文件内有效

默认情况下，const对象被设定为仅在文件内有效，当多个文件中出现了同名的const变量时，其实等同于不同文件中分别定义了独立的变量。

如果希望在多个文件中都可以使用该const类型对象，而无需编译器为每个文件分别生成独立变量。解决方法便是：对于const变量不管是声明还是定义都添加extern关键字。例如：

```c++
/* file_1.cpp */
extern const int BUFFERSIZE=1024;   /* 初始化常量 */

/* file_1.hpp */
extern const int BUFFERSIZE;        /* 在头文件中也使用extern关键字，作用是指明该常量并非本文件独有，它的定义在别处出现 */
```

> Note
>
> 如果想要在多个文件之间共享const对象，必须在变量的定义之前添加extern关键字

### 2.4.1 const的引用

可以将引用绑定到const对象上，称为**对常量的引用**(reference to const)。。

```cpp
const int i=1024;   
cosnt &r=i;         /* 正确，引用的对象和其引用本身都为常量 */
r=42;               /* 错误，r是对常量i的引用，常量无法被修改 */
int r2=i;           /* 错误，普通引用（对非常量的引用）无法指向常量对象 */
```

> 通俗上讲：我们一般将“对常量（const）的引用”说成“常量引用”。但是严格上讲，并不存放常量引用，因为引用本身就不是一个对象，无法保证引用的不变。

初始化和对const的引用
初始化常量引用时运行用任意表达式作为初始值，只要其表达式能够转换为引用类型。运行常量引用绑定非常量的对象、字面值或一般表达式。（**这也是“引用的类型必须与所引用的类型一致”的两个例外之一**）

```cpp
int i =23;
const int &v1=i;        /* 允许将const int&绑定到一个普通int对象上 */
const int &v2=23;       /* 正确，允许为一个常量引用绑定非常量的对象、字面值，甚至是个一般表达式 */
const int &v3 = v1*2;   /* 正确 */
int &v4=v1*2;           /* 错误，无法将常量赋值给普通变量 */


```

常量引用**仅对引用可参与的操作做出限定**，对于引用的对象本身是不是一个常量未作限定。

```cpp
int i = 23;
int &v1 =i;         
const int &v2=i;    /* 绑定常量 其实际步骤为：const int temporary = i; const int &v2 = temporary; */
v1 = 0;             /* 正确*/
v2=0;               /* 错误，v2作为一个常量引用 */
```

### 2.4.2 指针和const

与引用类似，存在**指向常量的指针**（pointer to const），其不能改变所指对象的值。

```cpp
const double pi = 3.14;    
double &ptr= &pi;           /* 错误，无法将指向常量的指针指向非常量 */
const double *cptr=&pi;     /* 正确 */
*cptr=3.15;                 /* 错误，无法修改常量的值 */
```

由此可以看出：指向常量的指针必定是指针常量。

与对常量的引用一样并未规定所指的对象必须是一个常量。仅作出不能通过该指针修改对象的值。

```cpp
/* 运行指向常量的指针所指向的对象为非常量 */
double dval = 3.14;
cptr = &dval;               /* 合法 */
```

与之“指向常量的指针”（也可以说是“指针常量”）`const *ElemType p`对应的“常量指针”`*ElemType const p`。

```cpp
int number=1;
int *const p=&number;     /* 正确，p将一直指向number变量 */
```

> 在这里指的“常量指针”表示指针定为常量，即存放在指针中的地址为常熟，所以“常量指针”是无法再次指向新的对象。

### 2.4.3 顶层const

在此回答一下：什么是常量引用？什么是指针常量？什么是常量指针？

😆不知道有没有搞糊涂呢？

那么我们来学习一下顶层const吧！深刻立即这三者。

指针本身既是对象，同时它又可以指向对象。那么便存在这两个对象是否是常量的关系：

- 指针是常量，指向对象不是常量。常量指针`ElemType *const pi`
- 指针不是常量，指向对象是常量。指针常量`const ElemType *pi`
- 指针是常量，指向对象也是常量。常指针常量`const ElemType *const pi`

> 至于如何读，则遵从从右向左读，观察首先遇见的是const还是ElemType。

回到标题，何为“顶层const”？从上述我们已经了解const定义位置的不同会产生不同结果，所以我们在此下定义：**顶层const（top-level const)表示指针本身是个常量，底层const(low-level const)表示所指对象是一个常量**。

同样，此定义具有普适性，我们可以说：顶层const可以表示任意对象为常量，例如：指针、类、算术类型等。底层const则与指针和引用等复合类型的基本类型有关。

例如：

```cpp
int i=0;
int *const p1=&i;   /* 不能修改p1值，顶层const */
const int j=0;      /* 不能修改j的值，顶层const */
const int *p2=&j;   /* 可以修改p1值，底层const */
const int &r=j;     /* 用于声明的const都为底层const */
```

说完顶层和底层const，那么它们有什么特点？

当对执行对象进行拷贝操作，顶层const不受影响。

> 我们之前说过，执行拷贝操作并不会改变被拷贝对象的值，所以拷入和拷出对象是否是常量都没什么影响。
>
> 例如：`i=j;`

但是对于底层const来说拷入和拷出对象必须都为底层const才行。

```cpp
int *p3=p2;      /* 错误，p2拥有底层const，但是p3并未拥有 */
/* 非常量可以转换为常量 */
p2=&i;
```

同时，还有一个点：对常量对象取地址其实是一种底层const。

### 2.4.4 constexpr和常量表达式

何谓“constexpr”?

const expression，常量表达式。即在编译过程中就可得到答案，而非在运行过程中且值不会改变的表达式。

例如：

```cpp
const int SIZE=20;      /* 正确 */
const int NEXT=SIZE-1;  /* 正确 */
const int RESULT=getValue();    /* 错误 */
```

C++ 11新特性：`constexpr`：

使用场景：当我们无法立刻确定某个常量的初始值时，可以使用该声明类型由编译器检验此变量的初始值是否为常量表达式。

例如：

```cpp
constexpr int i=0;              /* 正确 */
constexpr int j=i+1;            /* 正确 */
constexpr int k=getValue();     /* 当getValue()函数为constexpr函数时，正确 */
```

> 注意
>
> 当在constexpr声明中定义一个指针，其限定符constexpr仅对指针有效，对所指向的对象无效。
> ```cpp
> constexpr int i=10;
> const int *p=&i;      /* 指向int常量的指针 */
> constexpr int *p=&i;  /* 指向int常量的常量指针 */
> ```

---

## 2.5 处理类型

为什么需要进行类型处理？

由于程序愈发复杂从而导致程序中使用到的类型也愈发繁多，其中不免存在类型名称不明确或难写的情况。这个时候就需要通过对类型处理获取良好地编写和审查代码体验。

### 2.5.1 类型别名

type alias，类型别名。类型别名的作用显而易见，为类型起个别名从而获得更好的体验。

例如：

```cpp
/* 将类型名可视化 */
typedef int ElemType;
/* 将类型名简单化*/
typedef _Size_Of_Aarry AarrySize;
/* 此类型作用是设置全局元素类型 */
typedef double ElementType;     /* 将类型名做注释，助于理解 */
```

当然，除了使用关键字`typedef`这一声明语句的基本数据类型之外，C++ 11也规定一种新方式：使用别名声明（alias
declaration）来定义类型的别名：

例如：

```cpp
using ElemType=int;     /* int的别名为ElemType */
```

其类型别名与类型名等价。

### 2.5.2 auto类型说明符

auto，C++ 11新特性。auto是C++的一个特殊类型声明，它在声明时标识一个自动变量。通过编译器区分析此时表达式所属的类型。类似于JavaScript的`var`。

需要注意的是：
- 编译器推断出来的auto类型有时候和初始值的类型是不完全一样的，编译器会适当地改变结果类型使其更符号初始化规则。
- 当引用被用作初始值时，编译器会以引用对象的类型作为auto的类型。
- auto会一般会忽略顶层const而保留底层const，若需顶层const则要手动加上。

一个典型的例子：

```cpp
const std::string str="hello";
for(auto &c:str){
    /* ... */
}
```
在这个例子中，c的类型是`const char &`，编译器会将引用对象的类型作为auto的类型，所以auto类型为`const char`，不会忽略顶层const。

所以，在一般的定义类型上尽量少用auto语句，我们较难以发现auto的类型。若想要实现类似auto的效果，可以使用下方的decltype类型说明符。


### 2.5.3 decltype类型说明符

decltype，类型说明符。C++ 11引入新特性，它的作用是选择并返回操作数的数据类型。

听起来是不是很模糊🙃，那么直接示例出发：

```cpp
const int i=10,&j=i;
decltype(i) k=100;      /* 我们将i的类型（const int）拿过来给k，那么k的类型为const int */
decltype(j) m=k;        /* m的类型即为const int & */
```

特殊的：

```cpp
int i=10,&j=i,*p=&i;
decltype(j+1) m;    /*int, ① */
decltype(*p) k;     /*int&, ② */
decltype((i)) e;    /*int&, ③ */
```

> 解释
> 
> 语句①： j本身为引用，若语句①中decltype表达式仅有j，则返回值类型为int&，但当将j作为表达式的一部分，那么显然该表达式将会是一个具体值而非引用，所以表达式为j+1其返回值类型为int。通过这种方式我们可以将引用类型转为基本数据类型。
> 
> 语句②： 如果decltype表达式仅有解引用操作，那么decltype得到的为引用类型。原因：解引用指针所得到的是指向对象。
> 
> 语句③：若对decltype添加多层括号，编译器会将其认为是表达式，而非一层括号表示的该变量的类型。具体来讲，decltype(i)得到的是i的类型，decltype((i))得到的是i的引用。

---

## 2.6 自定义数据结构

何谓“数据结构”？

数据结构是指相互之间存在一种或多种特定关系的数据元素的集合。

用本书的理解：数据结构是把一组相关的数据元素组织起来然后使用它们的策略和方法。

常见的基本数据类型并不能满足我们对数据的处理需求，所以特别需要自定义数据结构用于处理特殊的数据。这些自定义的数据结构包括：string、istream、ostream等等。

编写自己的头文件

为了处理类，确保在不同文件中使用到同一个类和类定义一致性。所以类通常被定义在头文件中，而且类所在头文件的名字应该与类的名字一致。

头文件通常包含哪些只能被定义一次的实体，例如：类、const、constexpr变量等。

在某些情况下，一个源程序可能包含多个同样的头文件，例如：`iostream`。为了确保头文件多次包含依旧能够正常工作，常用的技术手段是**预处理器**（preprocessor），同时还会用到**头文件保护符**（header guard）。

例如：

```cpp
#ifndef CPP_PRIMER_2_6_HPP
#define CPP_PRIMER_2_6_HPP

#include <string>
struct Sales_data{
    std::string isbn;
    unsigned long sold_number;
    double sales_revenue;
};

#endif //CPP_PRIMER_2_6_HPP
```

头文件保护符依赖于预处理变量。预处理变量存在两种状态：已定义和未定义。

我们从示例代码中理解：

`#define`指令会将一个名字（比如这里的“CPP_PRIMER_2_6_HPP”，在CLion中该名字的处理方式是项目名称+设定名称）设定为预处理变量。

以下指令检查某个指定的预处理变量是否已定义：
- `#ifdef`指令：当变量已定义时为真。
- `#ifndef`指令：当变量未定义时为真。

回到示例，通过#define指令将“CPP_PRIMER_2_6_HPP”设置为预处理变量，当我们第一次使用该头文件时，#ifndef指令检测为真，那么便会将该头文件中#ifndef指令到#endif指令之间内容拷贝到目标程序中。拷贝结束后该头文件已经被定义，那么后续如果再次包含该头文件，则会忽略#ifndef指令和#endif指令之间内容。
