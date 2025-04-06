---
title: "C++ 基础： 第五章 语句"
date: 2025-04-06
categories: 
    - "C++ 学习笔记"
tags: 
    - "C++"
weight: 100
image: "https://s3.bmp.ovh/imgs/2024/11/23/f8eb19dce3c8aec6.jpg"
---
# 第五章 语句

## 前言

通常情况下，程序是按照顺序执行。但是仅仅依靠顺序执行无法解决一些复杂问题，所以C++提供一组*控制流*
（flow-of-control）语句支持更加复杂的执行路径。

---

## 5.1 简单语句

简单语句的三种语句：

```cpp
value+5;            /* 表达式语句 */
;                   /* 空语句 */
while(value<10){    /* 复合语句（块） */
    cout<<value<<" ";   
    ++value;
}
```

有一个很好的例子：

```cpp
/* 重复读取数据直到到达文件末尾或者遇到换行 */
while(cin>>s && s!='\n')
    ;   /* 空语句 */
```

当语法上只需要一条语句，但是逻辑上需要多条语句时便可以使用复合语句。

## 5.2 语句作用域

定义在控制结构当中的变量只在相应语句的内部可见。

## 5.3 条件语句

C++提供两种按条件执行的语句。一种是if语句，另一种是switch语句。

### if语句

if语句的形式：

```cpp
if(condition)
    statement
    
/* if-else形式 */
if(condition)
    statement
else
    statement2 
```

condition可以是一个表达式，也可以是一个初始化了的变量声明。但都必须用括号括起来。

### switch语句

当对多个固定选项进行选择时，使用if-else语句就显得冗余，所以C++提出switch语句。

switch语句的形式：

```cpp
switch(condition){
    case option1:
            statement1;
    case option2:
            statement2;
    case optionn:
            statement3;
    default:
            statement4; 
}
```

注意点：case标签必须为**整型常量表达式**，且任意的两个case标签不能相同；default属于特殊的case标签。

有关case的一个特点，可以将几个case标签写在同一行，用于强调这些case代表的是某个范围的值：

```cpp
char character;
int  number = 0;
std::cin >> character;
switch (character) {
    case 'a':
    case 'b':
    case 'c': number++; break;
    default: std::cout << "error!\n"; break;
}
/* 如果输入的是“a”、“b”、“c”，则返回1，否则返回0 */
std::cout << "number is " << number << "\n";    
```

> 建议
>
> 一般不要遗忘在最后一个case分支上添加break语句，如果有需要，请用注释注明。
>
> 即便不使用default语句，也请定义一个default标签。这样做的目的：表明对switch语句的case分支情况我们已经考虑全面。
>
> 无论是case还是default标签，我们都不应该让其内容为空，哪怕这个分支并未起作用，我们也应该使用空语句或者空块填充内容。


还有个关于在switch内部的变量定义讨论：

例如：

```cpp
bool index = false;
switch (index) {
    case true:
        std::string file_name;
        int         ival = 0;
        int         jval;
        break;
    case false: /* 错误！ */
        jval = 0;
        if (file_name.empty()) {};
        break;
    default: break;
}
```

C++规定：不允许跨过变量的初始化语句直接跳转到该变量作用域内的另一个位置。

## 5.4 迭代语句

迭代语句又叫循环语句，在之前我们了解过三种循环语句：

- while()
- for()
- do-while()

---

### while语句

其形式：

```cpp
while(condition)
    statement
```

其条件部分与if语句类型：可以是一个表达式，也可以是一个初始化了的变量声明。

### 传统for语句

其形式：

```cpp
for(init-statement;condition;expression)
    statement
```

注意点：

1. for循环语句中定义的变量仅在该循环语句中有效
2. init-statement仅存在一条声明语句（注意不是仅能声明一个对象），但可以有多个对象
3. for语句头能够省略任何一个语句，即便是全部省略也可以

### 范围for语句

其形式：

```cpp
for (declaration:expression)
    statement
```

### do while语句

其形式：

```cpp
do{
    statement
} while(condition);
```

相较于while语句，

- do-while语句会在进行判断语句之前执行一次statement
- do-while语句在括号后会添加一个分号

## 5.5 跳转语句

跳转语句用于**中断当前的执行过程**，C++语言提供了四种跳转语句：`break`、`continue`、`goto`和`return`。

### break语句

break语句负责终止**离它最近**的while、do-while、for或switch语句，并从这些语句**之后的第一条语句开始继续执行**。

### continue语句

continue语句负责终止**最近循环中当前迭代并立即开始下一次迭代**。其与break不同点在于：continue仅能出现在for、while和do-while语句中，除非switch语句被嵌套在前三者之中，否则不能在switch语句中使用continue。

其中断迭代及下次迭代过程：

- 对于while和do-while：终止当前迭代，执行判断条件语句。
- 对于传统for：终止当前迭代，执行for语句头的expression（注意：不是init-statement）。
- 对于范围for：终止当前迭代，使用下一个元素初始化循环控制变量。

### goto语句

goto语句的作用是从goto语句**无条件**跳转到同一函数内的另一个语句。

> 建议
>
> goto用时很爽，用完火葬场💀。由于goto的无条件，在程序开发中如果随时使用goto语句，那么难以对对象进行追踪。建议使用goto语句时其标签与调用点代码行数不超过30行。

goto语句其形式为：

```cpp
goto label;
```

label是用于标识一条语句的标识符。

**带标签语句**为一种特殊语句，其形式为在普通语句之前有一个标示符以及一个冒号。

```cpp
label: do{

}while(/*...*/);
```

> 注意
>
> 标签标示符独立与变量名和其他标示符名，所以标签标示符可以和其他实体的标示符使用同一个名字，但是建议不要这么做，容易导致混乱（除非你对你的代码排版有信心）。
>
> goto语句和控制权转向指定的带标签语句必须在同一个函数内。并且，goto语句无法将程序的控制权从变量作用域之外转到作用域之内。

这里需要考虑goto语句是向前跳还是向后跳：

如果向前跳，则之前声明的变量定义将会被销毁并重新创建，如果是向后跳，如果后续的变量已经被定义则合法，否则不合法。

## 5.6 try语句块和异常处理

异常是指*存在于运行时的反常行为*。

异常处理机制为程序中异常检测和异常处理这两部分的协作提供支持：

- throw表达式，throw用于异常检测。当出现异常，那么throw将会将异常“抛出”
- try语句块，try-catch语句用于异常处理，当异常被try“抛出”后，执行catch字句用于处理“异常”。因此也被称为“*异常处理代码*”（exception handler）

### throw表达式

throw表达式包括：关键字throw和紧跟的表达式。其表达式的类型就是抛出的异常类型。

例如：

```cpp
int i=2,j=0;
if(i/j) throw.runtime_error("被除数不能为零！");
/* runtime_error是标准库异常类型的一种，定义在stdexcept头文件中。
 * 使用该类型，必须初始化。初始化方式为为它提供一个string对象或者C风格字符串。
 */
```

抛出异常后将终止当前函数，并把控制权转移给能够处理该异常的代码。

### try语句块

其形式：

```cpp
try{
    program-statement
} catch(exception-declaration){
    handler-statements
} catch(exception-declaration){
    handler-statements
} // ...
```

catch字句包括三部分：关键字catch、括号内一个对象的声明（称作**异常声明**，exception declaration）以及一个块。

try语句块内的声明变量无法在块外部访问。

---

对执行catch语句的讨论：

当异常被抛出时，首先搜索抛出该异常的函数，如果没有与之匹配的catch字句，该函数将被终止，并在调用该函数的函数中寻找，如果依旧没有找到与之匹配的catch字句，该新函数也将被终止，继续向上搜索，直到找到适合类型的catch字句。

如果最终依旧没有找到与之匹配的catch字句，程序将转到**terminate**标准库，该函数的行为和系统有关，一般情况下，执行该函数将导致程序的非正常退出。

对于没有使用try语句块但是出现异常的函数，系统会调用terminate函数并终止当前程序的执行。

### 标准异常

C++标准库定义了一组类，其分别定义在4个头文件中：

- exception 头文件定义了最通用的异常类exception。它只报告异常的发生，不提供任何额外信息
- stdexcept 头文件定义了几种常用的异常类
- new 头文件定义了bad_alloc异常类型
- type_info 头文件定义了bad_cast异常类型

<stdexcept>定义的异常类：

|        类名        | 解释                      |
|:----------------:|:------------------------|
|    exception     | 最常见的问题                  |
|  runtime_error   | 只有在运行时才能检测出的问题          |
|   range_error    | 运行时错误：生成的结果超出了有意义的至于范围  |
|  overflow_error  | 运行时错误：计算上溢              |
| underflow_error  | 运行时错误：计算下溢              |
|   logic_error    | 程序逻辑错误                  |
|   domain_error   | 逻辑错误：参数对应的结果值不存在        |
| invalid_argument | 逻辑错误：无效参数               |
|   length_error   | 逻辑错误：试图创建一个超出该类型最大长度的对象 |
|   out_of_range   | 逻辑错误：使用一个超出有效范围的值       |

对于exception、bad_alloc和bad_cast对象，我们只能采用默认初始化方式，不允许为这些对象提供初始值。其他的异常类型必须使用string对象或
C风格字符串对象作为初始值初始化，不允许使用默认初始化方式。

同时，异常类型只定义了一个名为what的成员函数，该函数没有任何参数，返回值是一个指向*C风格字符串*的const char*。该字符串的目的是提供关于异常的文本信息。what函数返回的C风格字符串内容与异常对象有关，如果该异常类型存在初始值，则what返回该初始值字符串，如果无初始值，其返回内容由编译器决定。


