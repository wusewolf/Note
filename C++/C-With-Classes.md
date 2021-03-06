C-With-Classes
============

C With Classes 是C++的前身，故名思意，这是带类的C，C With Classes 的定位就是C的扩展语言，兼容C的语法。C With Classes的设计参考了Simula的方式。从C With Classes中可以看到一些C++的早期实现的影子，可以学到一些C语言面向对象实现技巧。

#类
很清楚，C With Classes最重要的特征——后来的C++也一样，就是类的概念。以下是一个类的例子：

```
class statck {
    char s[SIZE];
    char* min;
    char* top;
    char* max;
    void new();
public:
    void push(char);
    char pop();
};

char stack.pop()
{
    if(top<=min) error("stack underflow");
    return *(--top);
}

class stack s1, s2;
class stack *p1 = &s2;
class stack *p2 = new stack;
s1.push('h');
p1->push('a');
```

从以上代码能看出C With Classes与现在的C++十分接近。从这里反应出的关键设计决策包括：

1. C With Classes遵循Simula的方式，让程序员去描述类型，通过这些类型建立变量（对象）。
2. 采用编译时访问控制来限制对实际表示的访问。
3. 函数定义通常被写在"其他地方"，以使类声明看起来更像一个接口描述，而不像为了组织代码而提供的一种语法结构。
4. 函数new()是构造函数，这个函数对编译器具有特殊意义。这种函数为类提供了一种保证，这里的保证是指构造函数一定会调用，以初始化它所属的类的每个对象。
5. 同C语言一样，对象的分配可以有3种方式：在堆栈上（作为自动对象），在固定地址（静态对象），或者在自由存储区（在堆），与C不同的是，C with Classes为自由存储的分配和释放提供了特定的运算符new和delete。
...

后期C++的发展完全可以看做是在这些决策上的探索和优化。


#运行时的效率
为了提高运行时效率，提供了inline机制，在C with CLasses中只有成员函数做成inline的，要求函数称为inline只有一种方式，那就是把它的函数放进类声明之中。

关键字inline和允许非成员的inline函数的功能都是后来由C++提供的。

关键字inline只不过是一种提示，编译器可以去做，但也常常会忽略它们。

#连接模型
如何将分别编译的程序片段连接一起，这个问题对任何程序设计语言都是重要的，它也在一定程度上决定了这个语言所能提供的所有特征。

C的意图是采用按名称等价而不是按结构等价。例如：
```c
struct A {int x, y;};
struct B {int x, y;};
```
定义的是两个不相容的类型A和B。进一步说：
```c
struct c {int x, y;};   //in file 1
struct c {int x, y;};   //in file 2
```
定义了两个不同的类型，它们都称为C，而能越过编译单元的边界做检查的编译器应该给出一个"重复定义"的错误。采用这一规则的原因是为了使维护问题减到最少。

作为一个实践性问题，C以及后来的C++都保证，如前述A和B这样的类似结构具有类似的布局，这样就可能在它们之间转换，以明显的方式强制转换。

按名称等价是C++类型系统的基石，而布局相容性规则保证了可以使用显示转换，以便能提供低级的转换服务。

这个认识后来称为"唯一定义规则"：每个函数、变量、类型、常量等都应该在C++中恰好有一个定义。

#对象连接模型
C with CLasses中的对象简单地就是一个C结构。这样，
```c++
class stack {
    char s[10];
    char* min;
    char* top;
    char* max;
    void new();
public: 
    void push();
    char pop();
};
```
的布局与下面的结构完全一样：

```c++
struct stack {
    char s[10];
    char* min;
    char* top;
    char* max;
};
```
与此类似，通过把对成员函数的调用：

```c++
void stack.push(char c)
{
    if(top>max) error("stack overflow");
    *top++ = c;
}

void g(class stack* p)
{
    p->push('c');
}
```
直接映射到所生成的代码中的一个等价的C函数调用：
```c++
void stack_push(this,c) 
struct stack* this;
char c;
{
    if((this->top)>(this->max)) error("stack overflow");
    *(this->top)++ = c;
}

void g(p) struct* p;
{
    stack_push(p, 'c');
}
```

在每个成员函数中有一个称为this的指针，它所引用的就是调用成员函数的那个对象。
有时人们会问这个问题：为什么this是一个指针而不是一个引用？当this被引进C with Classes时，在这个语言中还根本没有引用机制。

#语法问题
C语言的声明有时从左往右，有时从右往左，不利于理解，C++作者认为可以把所有的声明符都变成后缀形式，还能保证声明都可以从左向右读，例如：

    int f(char)->[10]->(double)->;

这意味着函数f返回一个指针，该指针指向数组，这是一种指针的数组，其指针指向返回int指针的函数。但是这个想法一直没有发布实现，与此对应的是人们开始使用typedef逐步构造起复杂的类型。

#没有虚函数时实现多态性，没有模板时的容器类
可以用类似python的实现方法，在基类中为派生类实现一种变体记录，当派生类指针转换成基类指针后，可以从这个变体记录中获得原来的派生类，并还原真正的类型。

在没有提供泛型的时候，宏定义可以简单实现类似功能，但是这是一种最早和粗糙的技术。它被证明太容易引进错误。经过一系列改进后，最终形成了C++语言中的模板机制，以及一些通过模板类和基类去表述实例模板中的各种共同性的技术。

#对象的布局模型
派生类的实现就是简单的将基类成员和派生类成员毗邻放置。例如，给定了：
```c++
class A {
    int a;
public:
    /*member functions*/
};

class B: public A {
    int b;
public:
    /*member functions*/
};
```

类B的一个对象将采用一个结构来表示：
```c++
strcut B { //generated C code
    int a;
    int b;
}
```

#保护机制
C++的保护机制是编译时保护机制，在[ARM]中总结了C++的保护概念：

1. 保护是通过编译时的机制提供的，目标是防止意外事件，而不是防止欺骗或者有意的侵犯。
2. 访问权是由类本身授予，而不是单方面地取用。
3. 访问权控制是通过名称实行的，并不依赖于被命名事物的种类。
4. 保护的单位是类，而不是个别的对象。
5. 受控制的访问权，而不是可见性。

C++访问控制是为了防止意外事件而不是防止欺骗。任何程序设计语言，只要支持对原始存储器的访问，那就会使数据处于一种开放的状态，如果有意按某种违反数据项的原类型规则描述的方式去触动它，其企图总能实行。

#运行时的保证
前面描述的机制是为了防止未经授权的访问。第二类保证则通"特殊的成员函数"提供，例如构造函数，这些函数都是由编译器识别和调用的。这里的基本想法就是希望程序员能建立起某种保证有时也被称为不变式，以便使其他成员函数都能够依赖这些东西。
