---
title: C++ in Caffe
toc: true
tags:
  - C++
  - Caffe
categories:
  - 笔记
date: 2017-01-02 18:24:45
---

阅读Caffe源码过程中的C++学习笔记, 大部分内容来自其他人博客, 趁着学习Caffe源码也学习一下C++的特性

<!--more-->

### **const**

关键字：Const，Const函数，Const变量，函数后面的Const
 
看到`const`关键字，C++程序员首先想到的可能是const 常量。这可不是良好的条件反射。如果只知道用const 定义常量，那么相当于把火药仅用于制作鞭炮。

`const 更大的魅力是它可以修饰函数的参数、返回值，甚至函数的定义体。`

const 是constant 的缩写，“恒定不变”的意思。被const 修饰的东西都受到强制保护，可以预防意外的变动，能提高程序的健壮性。所以很多C++程序设计书籍建议：“Use const whenever you need”。

**用const 修饰函数的参数**

如果参数作输出用，不论它是什么数据类型，也不论它采用“指针传递”还是“引用传递”，都不能加const 修饰，否则该参数将失去输出功能。

const 只能修饰输入参数：
如果输入参数采用“指针传递”，那么加const 修饰可以防止意外地改动该指针，起到保护作用。

例如StringCopy 函数：

`void StringCopy(char *strDestination, const char *strSource);`

其中strSource 是输入参数，strDestination 是输出参数。
给strSource 加上const修饰后，如果函数体内的语句试图改动strSource 的内容，编译器将指出错误。
如果输入参数采用“值传递”，由于函数将自动产生临时变量用于复制该参数，该输入参数本来就无需保护，所以不要加const 修饰。
例如不要将函数void Func1(int x) 写成void Func1(const int x)。同理不要将函数void Func2(A a) 写成void Func2(const A a)。其中A 为用户自定义的数据类型。
对于非内部数据类型的参数而言，象void Func(A a) 这样声明的函数注定效率比较底。因为函数体内将产生A 类型的临时对象用于复制参数a，而临时对象的构造、复制、析构过程都将消耗时间。
为了提高效率，可以将函数声明改为void Func(A &a)，因为“引用传递”仅借用一下参数的别名而已，不需要产生临时对象。但是函数void Func(A &a) 存在一个缺点：
“引用传递”有可能改变参数a，这是我们不期望的。解决这个问题很容易，加const修饰即可，因此函数最终成为void Func(const A &a)。

以此类推，是否应将void Func(int x) 改写为void Func(const int &x)，以便提高效率？完全没有必要，因为内部数据类型的参数不存在构造、析构的过程，而复制也非常快，“值传递”和“引用传递”的效率几乎相当。

**问题是如此的缠绵，我只好将“const &”修饰输入参数的用法总结一下:**
 
对于非内部数据类型的输入参数，应该将“值传递”的方式改为“const 引用传递”，目的是提高效率。例如将void Func(A a) 改为void Func(const A &a)。
 
对于内部数据类型(基本数据类型)的输入参数，不要将“值传递”的方式改为“const 引用传递”。否则既达不到提高效率的目的，又降低了函数的可理解性。例如void Func(int x) 不应该改为void Func(const int &x)。

在caffe中, 

  `const Net* const root_net_;`

  - 左边的底层const表示指针所指向的对象是个常量

  - 右边的顶层const表示指针本身是个常量, 

  ```c++
  template <typename Dtype>
  Net<Dtype>::Net(const string& param_file, Phase phase, const Net* root_net):root_net_(root_net) {
    NetParameter param;
    ReadNetParamsFromTextFileOrDie(param_file, &param);
    param.mutable_state()->set_phase(phase);
    Init(param);
  }
  ```

  - 上面的代码是在创建Net类时候的初始化`数据成员root_net_`动作, Net类的构造函数`Net(const string& param_file, Phase phase, const Net* root_net)`

**用const 修饰函数的返回值**

如果给以“指针传递”方式的函数返回值加const 修饰，那么函数返回值（即指针）的内容不能被修改，该返回值只能被赋给加const 修饰的同类型指针。例如函数
`const char* GetString(void);`
如下语句将出现编译错误：
`char *str = GetString();`
正确的用法是
`const char *str = GetString();`
如果函数返回值采用“值传递方式”，由于函数会把返回值复制到外部临时的存储单元中，加const 修饰没有任何价值。
例如不要把函数int GetInt(void) 写成const int GetInt(void)。同理不要把函数A GetA(void) 写成const A GetA(void)，其中A 为用户自定义的数据类型。

如果返回值不是内部数据类型，将函数A GetA(void) 改写为const A & GetA(void)的确能提高效率。但此时千万千万要小心，一定要搞清楚函数究竟是想返回一个对象的“拷贝”还是仅返回“别名”就可以了，否则程序会出错。

- 函数返回值采用“引用传递”的场合并不多，这种方式一般只出现在类的赋值函数中，目的是为了实现链式表达。

例如：

```c++
class A
{
A & operate = (const A &other); // 赋值函数
};
A a, b, c; // a, b, c 为A 的对象

a = b = c; // 正常的链式赋值
(a = b) = c; // 不正常的链式赋值，但合法
```

如果将赋值函数的返回值加const修饰，那么该返回值的内容不允许被改动。上例中，语句 a = b = c 仍然正确，但是语句 (a = b) = c 则是非法的。

**const 成员函数**

任何不会修改数据成员的函数都应该声明为const 类型。如果在编写const 成员函数时，不慎修改了数据成员，或者调用了其它非const 成员函数，编译器将指出错误，这无疑会提高程序的健壮性。以下程序中，类stack 的成员函数GetCount 仅用于计数，从逻辑上讲GetCount 应当为const 函数。编译器将指出GetCount 函数中的错误。

```c++
class Stack
{
public:
void Push(int elem);
int Pop(void);
int GetCount(void) const; // const 成员函数
private:
int m_num;
int m_data[100];
};
int Stack::GetCount(void) const
{
++ m_num; // 编译错误，企图修改数据成员m_num
Pop(); // 编译错误，企图调用非const 函数
return m_num;
}
```
const 成员函数的声明看起来怪怪的：const 关键字只能放在函数声明的尾部，大概是因为其它地方都已经被占用了。

关于Const成员函数的几点规则：

1. const对象只能访问const成员函数,而非const对象可以访问任意的成员函数,包括const成员函数

2. const对象的成员是不可修改的,然而const对象通过指针维护的对象却是可以修改的

3. const成员函数不可以修改对象的数据,不管对象是否具有const性质.它在编译时,以是否修改成员数据为依据,进行检查

4. 然而加上mutable修饰符的数据成员,对于任何情况下通过任何手段都可修改,自然此时的const成员函数是可以修改它的

### **virtual**

**虚函数**

首先：强调一个概念
定义一个函数为虚函数，不代表函数为不被实现的函数。
定义他为虚函数是为了允许用基类的指针来调用子类的这个函数。
定义一个函数为纯虚函数，才代表函数没有被实现。
定义纯虚函数是为了实现一个接口，起到一个规范的作用，规范继承这个类的程序员必须实现这个函数。

```c++
class A  
{  
public:  
    virtual void foo()  
    {  
        cout<<"A::foo() is called"<<endl;  
    }  
};  
class B:public A  
{  
public:  
    void foo()  
    {  
        cout<<"B::foo() is called"<<endl;  
    }  
};  
int main(void)  
{  
    A *a = new B();  
    a->foo();   // 在这里，a虽然是指向A的指针，但是被调用的函数(foo)却是B的!  
    return 0;  
}  
```

这个例子是虚函数的一个典型应用，通过这个例子，也许你就对虚函数有了一些概念。它虚就虚在所谓“推迟联编”或者“动态联编”上，一个类函数的调用并不是在编译时刻被确定的，而是在运行时刻被确定的。由于编写代码的时候并不能确定被调用的是基类的函数还是哪个派生类的函数，所以被成为“虚”函数。
    虚函数只能借助于指针或者引用来达到多态的效果。

 
**纯虚函数**

一、定义
　
纯虚函数是在基类中声明的虚函数，它在基类中没有定义，但要求任何派生类都要定义自己的实现方法。在基类中实现纯虚函数的方法是在函数原型后加“=0”
　
`virtual void funtion1()=0`

二、引入原因
　　
1.为了方便使用多态特性，我们常常需要在基类中定义虚拟函数。
　　
2.在很多情况下，基类本身生成对象是不合情理的。例如，动物作为一个基类可以派生出老虎、孔雀等子类，但动物本身生成对象明显不合常理。
　　
为了解决上述问题，引入了纯虚函数的概念，将函数定义为纯虚函数（方法：virtual ReturnType Function()= 0;），则编译器要求在派生类中必须予以重写以实现

多态性。同时含有纯虚拟函数的类称为抽象类，它不能生成对象。这样就很好地解决了上述两个问题。
声明了纯虚函数的类是一个抽象类。所以，用户不能创建类的实例，只能创建它的派生类的实例。
纯虚函数最显著的特征是：它们必须在继承类中重新声明函数（不要后面的＝0，否则该派生类也不能实例化），而且它们在抽象类中往往没有定义。
定义纯虚函数的目的在于，使派生类仅仅只是继承函数的接口。
纯虚函数的意义，让所有的类对象（主要是派生类对象）都可以执行纯虚函数的动作，但类无法为纯虚函数提供一个合理的缺省实现。所以类纯虚函数的声明就是在告诉子类的设计者，“你必须提供一个纯虚函数的实现，但我不知道你会怎样实现它”。

抽象类的介绍

抽象类是一种特殊的类，它是为了抽象和设计的目的为建立的，它处于继承层次结构的较上层。
（1）抽象类的定义：  称带有纯虚函数的类为抽象类。
（2）抽象类的作用：
抽象类的主要作用是将有关的操作作为结果接口组织在一个继承层次结构中，由它来为派生类提供一个公共的根，派生类将具体实现在其基类中作为接口的操作。所以派生类实际上刻画了一组子类的操作接口的通用语义，这些语义也传给子类，子类可以具体实现这些语义，也可以再将这些语义传给自己的子类。
（3）使用抽象类时注意：

- 抽象类只能作为基类来使用，其纯虚函数的实现由派生类给出。如果派生类中没有重新定义纯虚函数，而只是继承基类的纯虚函数，则这个派生类仍然还是一个抽象类。如果派生类中给出了基类纯虚函数的实现，则该派生类就不再是抽象类了，它是一个可以建立对象的具体的类。

- 抽象类是不能定义对象的。

总结：

1、纯虚函数声明如下： virtual void funtion1()=0; 纯虚函数一定没有定义，纯虚函数用来规范派生类的行为，即接口。包含纯虚函数的类是抽象类，抽象类不能定义实例，但可以声明指向实现该抽象类的具体类的指针或引用。
2、虚函数声明如下：virtual ReturnType FunctionName(Parameter)；虚函数必须实现，如果不实现，编译器将报错，错误提示为：
error LNK****: unresolved external symbol "public: virtual void __thiscall ClassName::virtualFunctionName(void)"
3、对于虚函数来说，父类和子类都有各自的版本。由多态方式调用的时候动态绑定。
4、实现了纯虚函数的子类，该纯虚函数在子类中就变成了虚函数，子类的子类即孙子类可以覆盖该虚函数，由多态方式调用的时候动态绑定。
5、虚函数是C++中用于实现多态(polymorphism)的机制。核心理念就是通过基类访问派生类定义的函数。
6、在有动态分配堆上内存的时候，析构函数必须是虚函数，但没有必要是纯虚的。
7、友元不是成员函数，只有成员函数才可以是虚拟的，因此友元不能是虚拟函数。但可以通过让友元函数调用虚拟成员函数来解决友元的虚拟问题。
8、析构函数应当是虚函数，将调用相应对象类型的析构函数，因此，如果指针指向的是子类对象，将调用子类的析构函数，然后自动调用基类的析构函数。

- 有纯虚函数的类是抽象类，不能生成对象，只能派生。他派生的类的纯虚函数没有被改写，那么，它的派生类还是个抽象类。
- 定义纯虚函数就是为了让基类不可实例化, 因为实例化这样的抽象数据结构本身并没有意义, 或者给出实现也没有意义
实际上我个人认为纯虚函数的引入，是出于两个目的
1、为了安全，因为避免任何需要明确但是因为不小心而导致的未知的结果，提醒子类去做应做的实现。
2、为了效率，不是程序执行的效率，而是为了编码的效率。

**C++中虚析构函数的作用**

C++开发的时候，用来做基类的类的析构函数一般都是虚函数。可是，为什么要这样做呢？

```c++ 
class ClxBase
{
public:
    ClxBase() {};
    virtual ~ClxBase() {}; //虚析构函数

    virtual void DoSomething() { cout << "Do something in class ClxBase!" << endl; };
};

class ClxDerived : public ClxBase
{
public:
    ClxDerived() {};
    ~ClxDerived() { cout << "Output from the destructor of class ClxDerived!" << endl; }; 

    void DoSomething() { cout << "Do something in class ClxDerived!" << endl; };
};
```
主函数:

```c++
ClxBase *pTest = new ClxDerived;
pTest->DoSomething();
delete pTest;
```
输出结果:

`Do something in class ClxDerived!`
`Output from the destructor of class ClxDerived!`

如果把类ClxBase析构函数前的virtual去掉，那输出结果就是：

`Do something in class ClxDerived!`

也就是说，类ClxDerived的析构函数根本没有被调用！一般情况下类的析构函数里面都是释放内存资源，而析构函数不被调用的话就会造成内存泄漏。所以虚析构函数做是为了`当用一个基类的指针删除一个派生类的对象时，派生类的析构函数会被调用, 从而释放内存空间。
当然，并不是要把所有类的析构函数都写成虚函数。因为当类里面有虚函数的时候，编译器会给类添加一个虚函数表，里面来存放虚函数指针，这样就会增加类的存储空间。所以，只有当一个类被用来作为基类的时候，才把析构函数写成虚函数。

### **explicit**

按照默认规定，只有一个参数的构造函数也定义了一个隐式转换，将该构造函数对应数据类型的数据转换为该类对象，如下面所示：

```c++
class String {
    String ( const char* p ); // 用C风格的字符串p作为初始化值
    //…
}
String s1 = “hello”; //OK 隐式转换，等价于String s1 = String（“hello”）;
```
 
但是有的时候可能会不需要这种隐式转换，如下：
```c++
class String {
    String ( int n ); //本意是预先分配n个字节给字符串
    String ( const char* p ); // 用C风格的字符串p作为初始化值
    //…
}
```
下面两种写法比较正常：
```c++
String s2 ( 10 );   //OK 分配10个字节的空字符串
String s3 = String ( 10 ); //OK 分配10个字节的空字符串
```
下面两种写法就比较疑惑了：
```c++
String s4 = 10; //编译通过，也是分配10个字节的空字符串
String s5 = ‘a’; //编译通过，分配int（‘a’）个字节的空字符串
```
s4 和s5 分别把一个int型和char型，隐式转换成了分配若干字节的空字符串，容易令人误解。
为了避免这种错误的发生，我们可以声明显示的转换，使用explicit 关键字：
```c++
class String {
    explicit String ( int n ); //本意是预先分配n个字节给字符串
    String ( const char* p ); // 用C风格的字符串p作为初始化值
//…
}
```
**加上explicit，就抑制了String ( int n )的隐式转换，**
 
下面两种写法仍然正确：
```c++
String s2 ( 10 );   //OK 分配10个字节的空字符串
String s3 = String ( 10 ); //OK 分配10个字节的空字符串
 
//下面两种写法就不允许了：
String s4 = 10; //编译不通过，不允许隐式的转换
String s5 = ‘a’; //编译不通过，不允许隐式的转换
```
因此，某些时候，explicit 可以有效得防止构造函数的隐式转换带来的错误或者误解

explicit只对构造函数起作用，用来抑制隐式转换。

如：  
 
```c++
class A{   
  A(int a);   
};   
int Function(A a);      

当调用 Function(2)的时候，2会隐式转换为A类型。这种情况常常不是程序员想要的结果，所以，要避免之，就可以这样写：   
  
class A{   
  explicit A(int a);   
};   
int Function(A a);  
```

这样，当调用`Function(2)`的时候，编译器会给出错误信息（除非`Function`有个以`int`为参数的重载形式），这就避免了在程序员毫不知情的情况下出现错误。

**总结：`explicit`只对构造函数起作用，用来抑制隐式转换。**

### **inline**

下面的函数，函数体中只有一行语句： 
```c++
double Average(double total, int number){ 
  return total/number;
}
```

这么简单的语句为什么要定义为一个函数呢?第一，它使程序更可读；第二，它使这段代码可以重复使用。但是，它也有缺点：当它被频繁地调用的时候，由于调用函数的开销，使得应用程序的时间效率很低

为了避免函数调用的开销, 对于上面的函数把它定义为内联函数的形式：
```c++
inline double Average(double total, int number){
  return total/number;
}
```
函数的引入可以减少程序的目标代码，实现程序代码的共享。函数调用需要时间和空间开销，调用函数实际上将程序执行流程转移到被调函数中，被调函数的代码执行完后，再返回到调用的地方。这种调用操作要求调用前保护好现场并记忆执行的地址，返回后恢复现场，并按原来保存的地址继续执行。对于较长的函数这种开销可以忽略不计，但对于一些函数体代码很短，又被频繁调用的函数，就不能忽视这种开销。引入内联函数正是为了解决这个问题，提高程序的运行效率。

在程序编译时，编译器将程序中出现的内联函数的调用表达式用内联函数的本体来进行替换。由于在编译时将内联函数体中的代码替代到程序中，因此会增加目标程序代码量，进而增加空间开销，而在时间开销上不象函数调用(函数调用有栈内存创建和释放的开销)时那么大，可见它是以目标代码的增加为代价来换取时间的节省。

**总结：inline函数是提高运行时间效率，但却增加了空间开销。即inline函数目的是：为了提高函数的执行效率(速度)。**

关键字inline必须与函数定义体放在一起才能使函数真正内联，仅把inline放在函数声明的前面不起任何作用。因为inlin是一种用于实现的关键字，不是一种用于声明的关键字。许多书籍把内联函数的声明、定义体前都加了inline关键字，但声明前不应该加(加不加不会影响函数功能)，因为声明与定义不可混为一谈。

使用内联函数时应注意以下几个问题：
（1） 在一个文件中定义的内联函数不能在另一个文件中使用。它们通常放在头文件中共享。
（2） 内联函数应该简洁，只有几个语句，如果语句较多，不适合于定义为内联函数。 
（3） 内联函数体中，不能有循环语句、if语句或switch语句，否则，函数定义时即使有inline关键字，编译器也会把该函数作为非内联函数处理。
（4） 内联函数要在函数被调用之前声明。

例如：
```c++
#include <iostream.h>
int increment(int i);
inline int increment(int i){
  i++; return i;
}
void main(void){  ……
}
```
如果我们修改一下程序，将内联函数的定义移到main()之后：
```c++
#include <iostream.h>
int increment(int i);
void main(void){  ……
}
内联函数定义放在main()函数之后
inline int increment(int i){
  i++; return i;
}
```

内联函数在调用之后才定义，这段程序在编译的时候编译器不会直接把它替换到main中。也就是说实际上"increment(int i)"只是作为一个普通函数被调用，并不具有内联函数的性质，无法提高运行效率

### **`#`和`##`**

`#`是字符串化的意思，出现在宏定义中的#是把跟在后面的参数转成一个字符串；

```c++ 
#define  strcpy__(dst, src)      strcpy(dst, #src)
     
strcpy__(buff,abc)  //相当于 strcpy__(buff,“abc”)
```

`##`是连接符号，把参数连接在一起:

```c++ 
#define FUN(arg)  my##arg

FUN(ABC)  //等价于FUN(myABC)  
```