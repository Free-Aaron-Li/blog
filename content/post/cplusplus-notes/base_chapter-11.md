---
title: "C++ 基础： 第十一章 关联容器"
date: 2025-04-06
categories: 
    - "C++ 学习笔记"
tags: 
    - "C++"
weight: 100
image: "https://s3.bmp.ovh/imgs/2024/11/23/f8eb19dce3c8aec6.jpg"
---
# 第十一章 关联容器

## 前言

关联容器和顺序容器有着本质的不同：关联容器中的元素是按关键字来保存和访问的。与之相对，顺序容器中的元素是按它们在容器中的位置来顺序保存和访问的。

关联容器支持高效的关键字查找和访问。两个主要的**关联容器**（associative-container）类型是map和set。

map中元素以[关键字-值]（key-vale）方式存在，关键字起到索引的作用，值则表示与索引相关联的数据。

set中元素仅包含一个关键字，其支持高效的关键字查询操作。

标准库提供8个关联容器：

| 按关键字有序保存元素 |        解释        |
|:----------:|:----------------:|
|    map     | 关联数组：保存`关键字-值`对  |
|    set     | 关键字即值，即只保留关键字的容器 |
|  multimp   |   关键字可重复出现的map   |
|  multiset  |   关键字可重复出现的set   |

|        无序集合        |          解释           |
|:------------------:|:---------------------:|
|   unordered_map    |      用哈希函数组织的map      |
|   unordered_set    |      用哈希函数组织的set      |
| unordered_multimap | 用哈希函数组织的map：关键字可以重复出现 |
| unordered_multiset | 用哈希函数组织的set：关键字可以重复出现 |

以上的8个容器的不同体现在三个维度：

1. 每个容器要么是map,要么是set
2. 容器要求要么是不重复的关键字，要么允许重复关键字
3. 容器按照要么按照顺序存储，要么以无序存储

> map与multimap定义在`map`头文件中；set和multiset定义在`set`头文件中；无序容器定义在`unordered_map`和`unordered_set`中。

## 11.1 使用关联容器

简单举例：

map类型通常被称为<b>关联数组</b>（associative array）。与一般数组类似，只不过其下标为关键字。例如：给定一个名字到电话号码的map，我们可以使用一个人的名字作为下标来获取此人的电话号码。

set本质就是关键字的**简单集合**。

#### 使用map

示例：

```cpp
#include <iostream>
#include <string>
#include <map>

int main(){
    std::map<std::string,std::size_t> word_count;
    std::string word;
    while(std::cin>>word)
        ++word_count[word];
    cout<<"\n";
    for(const auto& w : word_count)
        std::cout<<w.first<<" occurs "<<w.second<<((w.second>1)?" times\n":" time\n");
    return 0;
}
```

> RUN
>
> ```cpp
> $ g++ -o main -g main.cpp
> $ ./main
> hello world !
> this is good !
> this is bad !
> 
> ! occurs 3 times
> bad occurs 1 time
> good occurs 1 time
> hello occurs 1 time
> is occurs 2 times
> this occurs 2 times
> world occurs 1 time
> ```

在上述示例代码中，map保存的每个元素中，关键字类型为`std::string`，值类型为`std::size_t`。 该示例中，将一个string作为下标，与此string相关联的size_t类型作为计数器。

当我们从map中**提取**一个元素时，会得到一个pair类型的对象。pair是一个模板类型，保存两个数据成员（first和second）。map所使用的pair用first成员保留关键字，second成员保存对应的值。

#### 使用set

对于上面的扩展：合理忽略一些常见单词，如“the”、“and”、“or”、“is”等。

```cpp
#include <iostream>
#include <string>
#include <map>
#include <set>

int main(){
    std::map<std::string,std::size_t> word_count{};
    std::set<std::string> exclude{  "The","But","And","Or","An","A","Is","!",
                                    "the","but","and","or","an","a","is"};
    std::string word;
    while(std::cin>>word)
        if (exclude.find(word) == exclude.end())
            ++word_count[word];

    std::cout<<"\n";
    for(const auto& w : word_count)
        std::cout<<w.first<<" occurs "<<w.second<<((w.second>1)?" times\n":" time\n");
    return 0;
}
```

> RUN
>
> ```cpp
> $ g++ -o main -g main.cpp
> $ ./main
> hello world !
> this is bad !
> this is good 
> 
> bad occurs 1 time
> good occurs 1 time
> hello occurs 1 time
> this occurs 2 times
> world occurs 1 time
> ```

调用find函数会返回一个迭代器，如果找到word迭代器指向该关键字，否则返回一个尾后迭代器。

## 11.2 关联容器概述

常规的[容器操作](https://www.cnblogs.com/aaroncoding/p/17596720.html) 关联容器能够支持，但是，对于顺序容器位置相关的操作（如：push_front、push_back）、插入操作等不支持。

关联容器的迭代器都是双向的。

### 定义关联容器

每个关联容器都定义了一个默认构造函数，它创建一个指定类型的空容器。

初始化关联容器必须能够将初始化器转换为容器中元素的类型。特别的：初始化map,必须提供关键字类型和值类型，且将每个关键字——值对包围在花括号中。

示例：

```cpp
// 空容器
std::map<std::string,std::size_t> word_count;
// 列表初始化
std::set<std::string> filter = { "The","But","And","Or","An","A","Is","!",
                                 "the","but","and","or","an","a","is"};
// map初始化
std::map<std::string,std::string> authors = {   {"Joyce","James"},
												{"Austen","Jane"},
												{"Dickens","Charles"}};
```

#### 初始化multimap或multiset

对于map和set来说，对于一个给定的关键字，只能有一个元素的关键字等于它。但是，对于multimap和multiset则没有此限制。例如：在统计单词数量时，每个单词最多拥有一个元素。但在词典中，特定单词可能含有多重释义。

示例：

```cpp
#include <iostream>
#include <set>
#include <vector>

int main(){
    std::vector<int> i_vec;
    for(std::vector<int>::size_type i=0;i!=10;++i){
        i_vec.push_back(i);
        i_vec.push_back(i);
    }
    std::set<int> i_set(i_vec.cbegin(),i_vec.cend());
    std::multiset<int> i_mset(i_vec.cbegin(),i_vec.cend());

    std::cout<<"vector size: "<<i_vec.size()<<" set size: "
        <<i_set.size()<<" multiset size: "<<i_mset.size()<<"\n";
}
```

> RUN
>
> ```cpp
> $ g++ -o main -g main.cpp
> $ ./main
> vector size: 20 set size: 10 multiset size: 20
> ```

在上述示例中，multiset允许重复关键字。

### 关键字类型的要求

关联容器对关键字类型有一些限制。对于有序容器（这里及以下均指代关联有序容器）来说关键字类型必须定义元素比较的方法，默认情况下，标准库使用关键字类型的<运算符来比较两个关键字。

#### 有序容器的关键字类型

与算法类似，有序容器也可以通过自定义操作替代关键字的<运算符。但是必须遵循**严格弱序
**（strict weak ordering）。可以将严格弱序看作“小于等于”，其必须具备如下性质：

- 两个关键字不能同时“小于等于”对方；如果k1“小于等于”k2,那么k2绝不能“小于等于”k1。
- 如果k1“小于等于”k2,且k2“小于等于”k3,那么k1必须“小于等于”k3。
- 如果存在两个关键字，任何一个都不“小于等于”另一个，那么我们称这两个关键字是“等价”的。如果k1“等价于”k2,且k2“等价于”k3,那么k1必须“等价于”k3。

如果两个关键字是等价的，那么容器将它们视为相等来处理。

> 在实际编程中，重要的是，如果一个类型定义了“行为正常”的<运算符，则它可以用作关键字类型。

**如果选择自定义操作，则必须在定义关联容器类型时提供此操作的类型**，自定义操作类型必须在尖括号中紧跟着元素类型。

### pair类型

pair：标准库类型，定义在`utility`头文件中

一个pair保存两个数据成员，类似容器。pair作为一个模板用来生成特定类型。创建一个pair,必须提供两个类型名（不要求一致），pair的数据成员将具有对应的类型。

pair的默认构造函数对数据成员进行值初始化，当然可以手动提供初始化器。
示例：

```cpp
std::pair<std::string,std::string> a;
std::pair<std::string,std::size_t> b;
std::pair<std::string,std::string> c{"hello","world"};
```

pair的数据成员为public,其成员分别命名为first和second。

详细操作：

|        pair上的操作        |                       解释                        |
|:----------------------:|:-----------------------------------------------:|
|     pair<T1,T2> p;     |        p是一个pair,两个类型分别为T1和T2的成员都进行了值初始化         |
|  pair<T1,T2> p(v1,v2)  | p是一个成员类型为T1和T2的pair；first和second成员分别用v1和v2进行初始化 |
| pair<T1,T2> p={v1,v2}; |                   等价于p(v1,v2)                   |
|    make_pair(v1,v2)    |     返回一个用v1和v2初始化的pair。pair的类型从v1和v2的类型推断出来     |
|        p.first         |              返回p的名为first的（公有）数据成员               |
|        p.second        |              返回p的名为second的（公有）数据成员              |
|   p1==p2<br/>p1!=p2    | 当first和second成员分别相等时，两个pair相等。相等性判断利用元素的==运算符实现 |

## 11.3 关联容器操作

| 关联容器额外的类型别名 |                              解释                               |
|:-----------:|:-------------------------------------------------------------:|
|  key_type   |                          此容器类型的关键字类型                          |
| mapped_type |                      每个关键字关联的类型；只适用于map                       |
| value_type  | 对于set,与key_type相同<br/>对于map,为pair<const key_type,mapped_type> |

需要注意到：对于value_type,我们不能改变一个元素的关键字，所以这些pair的关键字部分是const的。

```cpp
set<std::string>::key_type v1; // string
set<std::string>::value_type v2; // string
map<std::string,int>::key_type v3; // string
map<std::string,int>::mapped_type v4; // int
map<std::string,int>::value_type v5; // pair<const string,int>
```

### 关联容器迭代器

对于解引用一个关联迭代器时，我们会得到一个类型为容器的value_type的值的引用。

```cpp
map<std::string,std::size_t> word_count;
auto map_iter=word_count.begin(); // 指向pair<const string,size_t>对象的引用
std::cout<<map_iter->first; // string
std::cout<<" "<<map_iter->second; // size_t
map_iter->first="new key"; // error
++map_iter->second; // true
```

虽然set类型同时定义了iterator和const_iterator类型，但是两种类型都只允许只读访问set中的元素。一个set中的关键字也是const。

在进行常规编历时，使用迭代器进行遍历一个map、multimap、set和multiset时，迭代器按照**关键字升序**遍历元素。

示例：

```cpp
#include <iostream>
#include <iterator>
#include <map>

int main(){
    std::map<std::string,int> word_count{{"c",1},{"b",2},{"a",3}};
    auto map_iter=word_count.cbegin();
    while(map_iter!=word_count.cend()){
        std::cout<<map_iter->first<<" "<<map_iter->second<<"\n";
        ++map_iter;
    }
}
```

> RUN
>
> ```cpp
> $ g++ main.cpp
> $ ./a.out
> a 3
> b 2
> c 1
> ```

通常来说，不会对关联容器使用泛型算法。在实际编程中，如果我们真要对一个关联容器使用算法，要么是将它当作一个<b>源序列</b>，要么当作一个<b>目的位置</b>。例如：使用泛型copy算符拷贝一个关联容器到另一个序列，或是使用inserter将一个插入器绑定到一个关联容器，依次将关联容器当作一个目的位置来调用另一个算法。

### 添加元素

关联容器中insert成员会向容器中添加一个元素或一个元素范围。insert有两个版本，「1」接受一对迭代器，「2」接受一个初始化列表。

```cpp
set_1.insert(vec.cbegin(),vec.cend());
set_2.insert({1,2,3,4});
```

对于map、set及其对应的无序列表来说，对于一个给定的关键字，只有<b>第一个</b>带此关键字的元素才能被插入到容器中。

示例：

```cpp
#include <iostream>
#include <iterator>
#include <map>

int main(){
    std::map<std::string,int> word_count{{"c",1},{"b",2},{"a",3}};
    word_count.insert({"c",4});
    auto map_iter=word_count.cbegin();
    while(map_iter!=word_count.cend()){
        std::cout<<map_iter->first<<" "<<map_iter->second<<"\n";
        ++map_iter;
    }
}
```

> RUN
>
> ```cpp
> $ g++ main.cpp
> $ ./a.out
> a 3
> b 2
> c 1
> ```

#### 向map添加元素

向map插入元素，注意元素类型为pair。

```cpp
map_1.insert({1,2});
map_2.insert(std::make_pair(1,2));
map_3.insert(std::pair<int,int>(1,2));
map_4.insert(std::map<int,int>::value_type(1,2));
```

|             关联容器insert操作             |                                                                     解释                                                                      |
|:------------------------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------:|
|             c.insert(v)              |                                                       v是value_type类型的对象；args用来构造一个元素                                                        |
|           c.emplace(args)            | 对于map和set,只有当元素的关键字不在c中时才插入（或构造）元素。函数返回一个pair,包含一个迭代器，指向具有指定关键字的元素，以及一个指示插入是否成功的bool值。<br/>对于multimap和multiset，总会插入（或构造）给定元素，并返回一个指向新元素的迭代器 |
|    c.insert(b,e)<br/>c.insert(il)    |             b和e是迭代器，表示一个c::value_type类型值的范围；il是这种值的花括号列表。函数返回void<br/>对于map和set,只插入关键字不在c中的元素。对于multimap和multiset,则会插入范围中的每个元素              |
| c.insert(p,v) <br/>c.emplace(p,args) |                             类似于insert(v)（或emplace(args)），但将迭代器p作为一个指示，指出从哪里开始搜索新元素应该存储的位置。返回一个迭代器，指向具有给定关键字的元素                              |

#### 检测insert的返回值

insert（或emplace）返回的值依赖于容器类型和参数。

对于不包含重复关键词的容器，添加单一元素返回pair,pair的first成员为迭代器，指向具有给定关键词的元素；second成员是一个bool值，指出元素是否插入成功。

示例：

```cpp
#include <iostream>
#include <iterator>
#include <map>

int main(){
    std::map<std::string,int> word_count{{"c",1},{"b",2},{"a",3}};
    auto result_1=word_count.insert({"c",4});
    auto result_2=word_count.insert({"d",4});
    auto map_iter=word_count.cbegin();
    while(map_iter!=word_count.cend()){
        std::cout<<map_iter->first<<" "<<map_iter->second<<"\n";
        ++map_iter;
    }
    std::cout<<"result_1 is: "<<(result_1.second?"true":"false")
             <<",result_2 is: "<<(result_2.second?"true":"false")<<"\n";
}
```

> ```cpp
> $ g++ main.cpp
> $ ./a.out
> a 3
> b 2
> c 1
> d 4
> result_1 is: false,result_2 is: true
> ```

### 删除元素

关联容器定义了三个版本的erase。

前两个版本与顺序容器的操作非常类似：都是通过一个迭代器或一个迭代器对进行删除一个元素或一个元素范围，函数返回迭代器。关联容器提供了一个额外的erase操作，其接受key_type参数。此版本删除匹配给定关键词的元素，返回实际删除的<b>元素数量</b>。

|  从关联容器删除元素   |                                       解释                                       |
|:------------:|:------------------------------------------------------------------------------:|
|  c.erase(k)  |                   从c中删除每个关键字为k的元素。返回一个size_type值，指出删除的元素的数量                    |
|  c.erase(p)  | 从c中删除迭代器p指定的元素。p必须指向c中一个真实元素，不能等于c.end()。返回一个指向p之后元素的迭代器，若p指向c中的尾元素，则返回c.end() |
| c.erase(b,e) |                            删除迭代器对b和e所表示的范围中的元素。返回e                             |

示例：

```cpp
#include <iostream>
#include <map>

int main(){
    std::multimap<std::string,int> word_count{{"c",1},{"b",2},{"a",3},{"c",4}};
    std::string removeal_word="c";
    auto removeal_num=word_count.erase(removeal_word);
    if(removeal_num)
        std::cout<<"ok: "<<removeal_word<<" removed,removeal num is "<<removeal_num<<"\n";
    else 
        std::cout<<"oops: "<<removeal_word<<"no found!\n";
}
```

> RUN
>
> ```cpp
> $ g++ main.cpp
> $ ./a.out
> ok: c removed,removeal num is 2
> ```

### map的下标操作

map和unordered_map容器提供下标运算符和对应的at函数。

| map和unordered_map的下标操作 |                      解释                      |
|:----------------------:|:--------------------------------------------:|
|          c[k]          | 返回关键字为k的元素；如果k不在c中，**添加**一个关键字为k的元素，对其进行值初始化 |
|        c.at(k)         |  访问关键字为k的元素，带参数检查；若k不再c中，抛出一个out_of_range异常  |

> 需要注意到的是，如果关键字不再map中会创建一个元素并插入到map中，关联值将进行值初始化。
>
> ```cpp
> map<string,size_t> word_count; // 空容器
> word_count["a"]=1; // 创建元素，关键字为a,值为1
> ```
> 详细流程：
> 1. 在word_count中搜索关键字为a的元素，未找到。
> 2. 将一个新的关键字-值对插入到word_count中。关键字是一个const string，保存a。值进行值初始化，上述例子中为0
> 3. 提取出新插入的元素，并将1赋予它。
>
> 注意：由于下标运算符可能插入一个新元素，所以仅可以对非const的map使用下标操作。

通常情况，解引用一个迭代器所返回的类型与下标运算符返回的类型一样。但对map不然：<b>当对一个map进行下标操作时，会获得一个mapped_type对象；但当解引用一个map迭代器时，会得到一个value_type对象</b>。

### 访问元素

关联容器提供多钟查找一个指定元素的方法。

| 在一个关联容器中查找元素的操作  |                                   解释                                    |
|:----------------:|:-----------------------------------------------------------------------:|
|    c.find(k)     |                 返回一个迭代器，指向第一个关键字为k的元素，若k不再容器中，则返回为后迭代器                  |
|    c.count(k)    |                 返回关键字等于k的元素的数量。对于不允许重复关键字的容器，返回值永远是0或1                  |
| c.lower_bound(k) |                         返回一个迭代器，指向第一个关键字不小于k的元素                         |
| c.upper_bound(k) |                         返回一个迭代器，指向第一个关键字大于k的元素                          |
| c.equal_range(k) | 返回一个迭代器pair,表示关键字等于k的元素的范围。若k不存在，pair的两个成员均等于c.end()（准确来说：指向关键字可以插入的位置） |

> 注意：lower_bound和upper_bound不适用于无序容器。下标和at操作只适用于非const的map和unordered_map。

这么多查找方式，应该根据解决问题思考。如果我们关心只不过是一个特定元素是否已在容器中，可能find是最佳选择。对于count和find,如果不需要计数，最好使用find。

#### 对map使用find代替下标操作

由于下标操作的严重副作用，除非使用下标符合你的预期，最好使用find查找map中的元素。

#### 在multimap或multiset中查找元素

如果一个multimap或multiset中有多个元素具有给定关键字，而这些元素在容器中会<b>相邻存储</b>。

lower_bound返回的迭代器指向第一个具有给定关键词的元素，upper_bound返回的迭代器指向最后一个匹配给定关键词的元素。

如果关键字不再multimap中，则lower_bound和upper_bound会返回相等的迭代器——指向一个不影响排序的关键字插入位置。这两个操作返回的迭代器可能是容器的尾后迭代器。如果关键字不存在且大于容器中任何关键字，则lower_bound返回尾后迭代器；如果查找的元素最大，upper_bound返回的也是尾后迭代器。

> 注意：<b>lower_bound和upper_bound都不会报告关键字是否存在</b>。如果不存在,lower_bound将会指向第一个关键字大于给定关键字的元素，有可能是尾后迭代器；upper_bound指向最后一个匹配给定关键字的元素之后的元素，有可能是尾后迭代器。
>
> 如果lower_bound和upper_bound返回相同的迭代器，则给定关键字不存在容器中。

### 一个单词转换的map

查看：[test](https://github.com/Free-Aaron-Li/Cpp_Study-program/blob/master/Cpp_Primer_Studying/PartII_STL/ChapterEleven_AssociativeContainer/src/exercise/11_3.cpp#L175)

## 11.4 无序容器

C++11中定义了4个<b>无序关联容器</b>（unordered associative container)。这些容器不是使用比较运算符来组织元素，而是使用一个<b>哈希函数</b>（hash function）和关键字类型的==运算符。在关键字类型的元素没有<b>明显</b>的序关系的情况下，无序容器是非常有用的。在某些应用中，维护元素的序代价非常高昂，此时无序容器也很有用。

虽然理论上哈希技术能获得更好的平均性能，但在实际中想要达到很好的效果还需要进行一些性能测试和调优工作。因此，使用无序容器通常更为简单（通常也会有更好的性能）。

> 如果关键字类型固有就是无序的，或者性能测试发现问题可以用哈希技术解决，就可以使用无序容器。

无序容器在存储上组织为一组桶，每个桶保存零个或多个元素。无序容器使用一个哈希函数将元素映射到桶。为了访问一个元素，容器首先计算元素的哈希值，它指出应该搜索哪个桶。容器将具有一个特定哈希值的所有元素都保存在相同的桶中。因此，无序容器的性能依赖于哈希函数的质量和桶的数量和大小。

对于相同的参数，哈希函数必须总是产生相同的结果。理想情况下，哈希函数还能将每个特定的值映射到唯一的桶。但是，将不同关键字的元素映射到相同的桶也是允许的。当一个桶保存多个元素时，需要顺序搜索这些元素来查找我们想要的那个。计算一个元素的哈希值和在桶中搜索通常都是很快的操作。但是，如果一个桶中保存了很多元素，那么查找一个特定元素就需要大量比较操作。

|    无序容器管理操作——桶接口     |      解释       |
|:--------------------:|:-------------:|
|   c.bucket_count()   |   正在使用的桶的数目   |
| c.max_bucket_count() | 容器能容纳的最多的桶的数量 |
|   c.bucket_size()    |  第n个桶中有多少个元素  |
|     c.bucket(k)      | 关键字为k的元素在哪个桶中 |

|     无序容器管理操作——桶迭代     |                解释                |
|:---------------------:|:--------------------------------:|
|    local_iterator     |         可以用来访问桶中元素的迭代器类型         |
| const_local_iterator  |           桶迭代器的const版本           |
|  c.begin(n),c.end(n)  |         桶n的首元素迭代器和尾后迭代器          |
| c.cbegin(n),c.cend(n) | 与前两个函数类似，但返回const_local_iterator |

|    无序容器管理操作——哈希策略    |                                 解释                                 |   
|:--------------------:|:------------------------------------------------------------------:|
|   c.local_factor()   |                        每个桶的平均元素数量，返回float值                         |
| c.max_local_factor() | c试图维护的平均桶大小，返回float值。c会在需要时添加新的桶，以使得local_factor<=max_local_factor |  
|     c.rehash(n)      |      重组存储，使得bucket_count>=n且bucket_count>size/max_load_factor      |
|     c.reserve(n)     |                     重组存储，使得c可以保存n个元素且不必rehash                      |

#### 无序容器对关键字类型的要求

默认情况下，无序容器使用关键字类型的==运算符来比较元素，它们还使用一个`hash<key_type>`类型的对象来生成每个元素的哈希值。标准库为内置类型（包括指针）、string、智能指针类型提供了hash模板。所以对上述类型可以定义对应的无序容器。

对于自定义的类类型必须定义自己的hash模板才能使用无序容器。

## 总结

关联容器支持通过<b>关键字</b>进行高效<b>查找</b>和<b>提取</b>元素。对于关键字的使用将关联容器和顺序容器区分开来。

标准库定义了8个关联容器，每个容器

- 是一个map或者是一个set。map保存关键字-值对；set仅保存关键字
- 要求关键字唯一或不要求。
- 保持关键字有序或不保证有序。

有序容器使用比较函数来比较关键字，从而将元素按顺序存储。默认情况下，比较操作采用关键字类型的<运算符；无序容器使用关键字类型的==运算符和一个hash<key_type>类型的对象来组织元素。

有序容器的迭代器通过关键字有序访问容器中的元素。无论在有序容器还是在无序容器中，具有相同关键字的元素都是相邻存储的。