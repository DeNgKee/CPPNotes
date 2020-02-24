# 1.变量

*C/C++*程序内存分为堆区，栈区，全局区（静态区），常量区，代码区。

ELF文件分为代码段，*.data段*，*.bss段*，*.rodata*段以及自定义段。

已经初始化的全局变量以及静态局部变量存放在*.data*段，没有初始化的存放在*.bss*段。由于没有初始化数据，所以其实不占用空间，因此在*ELF*文件中，*.bss*只是一个占位符，只有当程序真正运行起来之后才会在内存上真正的开辟*.bss*空间，并且在*.bss*空间中开辟空间，并且自动初始化为0。程序运行之后存放在全局静态区。

常量（字符串常量，*const*变量）存放在*.rodata*段，程序运行后存放在常量区

代码存放在代码段，程序运行后存放在代码区。

当程序被加载到内存中后，操作系统负责加载上述各段，并为程序分配堆和栈。堆存放局部变量以及函数形参，有操作系统自行分配和释放；栈存放由*malloc*函数申请的空间，由程序员显示地分配和释放。

## 1.1 局部变量

定义在某个函数内部或者某个*statement*内部的变量，生命周期为函数或者*statement*开始到结束。

## 1.2 全局变量

定义在所有函数之外（包括*main*函数），在*main*函数执行之前进行初始化。

若想实现在*main*函数之前执行某个函数，可以定义一个全局变量，变量值为某个函数的返回值即可。

若想全局变量跨多个源文件，需要在一个文件里进行定义，其他文件进行*extern*声明。

## 1.3 静态变量

静态局部变量只定义一个，存放在全局静态区，生命周期同全局变量，但是仅局部可见；静态全局变量只在该文件可见，不能跨文件使用。

可执行二进制程序 = *text + bss(0) + data + rodata*

正在运行的程序 = *text + bss + data + rodata + stack + heap*

# 2.字符、字符串

`char a = ‘a’;`

`char *b = “abc”;`

上述两个类型为字符及字符串常量，存储在ELF文件的*.rodata*段，程序运行后加载到内存空间的常量区。

`char c[] = “abc”;`此为*char*型数组，存放在堆区。

*string*类型是模板参数为*char*的*basic_string*模板类的类型定义：

`typedef basic_string<char, char_traits<char>, allocator<char>> string;`

# 3.浮点类型

定义在*IEEE754*标准中，规约化计算方式为$(-1)*sign*(2^exponent-2^{e-1}+1)*1.(fraction)$,

非规约化计算方式为$(-1)*sign*(2^(-126))*0.(fraction)$;

规约和非规约的区别为指数位是否大于0，若为0则为非规约数，指数位默认大小为$2^(-126)$；

若大于0则为规约数，指数为大小为$2^exponent-2^(e-1)+1$。

*fp16*:

 ![IEEE 754r Half Floating Point Format.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/IEEE_754r_Half_Floating_Point_Format.svg/175px-IEEE_754r_Half_Floating_Point_Format.svg.png) 

*float*:

 ![Float example.svg](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/Float_example.svg/590px-Float_example.svg.png) 

*double*:

 ![General double precision float.png](https://upload.wikimedia.org/wikipedia/commons/7/76/General_double_precision_float.png) 

# 4.类型提升

表达式中可以使用整数的地方，就可以使用枚举类型，或有符号或无符号的字符、短整数、整数位域。如果一个*int*可以表示上述类型，则该值被转化为*int*类型的值；否则，该值被转化为*unsigned int*类型的值。这一过程被称作*integral promotion*。

整型提升的意义在于：表达式的整型运算要在CPU的相应运算器件内执行，CPU内整型运算器(ALU)的操作数的字节长度一般就是*int*的字节长度，同时也是CPU的通用寄存器的长度。因此，即使两个*char*类型的相加，在CPU执行时实际上也要先转换为CPU内整型操作数的标准长度。通用CPU（*general-purpose CPU*）是难以直接实现两个8比特字节直接相加运算（虽然机器指令中可能有这种字节相加指令）。所以，表达式中各种长度可能小于`int`长度的整型值，都必须先转换为*int*或*unsigned int*，然后才能送入CPU去执行运算。

# 5.函数

函数可分为匿名函数和有名函数，匿名函数使用*lambda*表达式实现，匿名函数一般使用在该函数只使用一次的情况下，比如为*std::sort*定义一个比较函数：

```cpp
using itype = vector<int,int>;
itype a;
itype b;
```

我们希望通过比较第二个*int*的值来排序，那么可以像这样调用

```cpp
std::sort(a,b,[&](itype a, itype b ){
If(a[1]>b[1])
    return true;
return false;
});
```

function模板类可包装其他任何*callable*目标——函数、*lambda* 表达式、 *bind* 表达式或其他函数对象，还有指向成员函数指针和指向数据成员指针。我们可将匿名函数包装与function中使其成为有名函数，可使用function接受*bind*返回使成员函数作为线程入口函数：

```cpp
using namespace std;
using namespace std::placeholder;
Class MyClass{
public:
void MyFunc(int a, int b)
{
    count = a + b;
}
private:
    int count;
};
MyClass mc;
auto func = bind(MyClass::MyFunc,mc,_1,_2);
int a = 10;
int b = 100;
Thread T(func, a, b);
T.join();
cout<<”mc.count=”<<mc.count<<endl;
```

*std::bind*的返回值类型为*unspecified*，所以不能比较两个*bind*后的函数是否相等。

## 5.1 函数声明

函数声明不需要函数实现，不需要形参名。

## 5.2 函数定义

函数定义需要声明加实现

## 5.3 函数调用

第一个进栈的是主函数中函数调用后的下一条指令（函数调用语句的下一条可执行语句）的地址，然后是函数的各个参数，在大多数的*C*编译器中，参数是由右往左入栈的，然后是函数中的局部变量。注意静态变量是不入栈的。当本次函数调用结束后，局部变量先出栈，然后是参数，最后栈顶指针指向最开始存的地址，也就是主函数中的下一条指令，程序由该点继续运行。

## 5.4 函数副作用(*side effect*)

函数副作用指当调用函数时，除了返回函数值之外，还对主调用函数产生附加的影响。例如修改全局变量（函数外的变量），修改参数或改变外部存储。函数副作用会给程序设计带来不必要的麻烦，给程序带来十分难以查找的错误，并降低程序的可读性。严格的函数式语言要求函数必须无副作用。没有副作用的函数为纯函数。

## 5.5 函数参数传递

分为值传递和引用传递，引用传递形参就是实参，不会新构造一个副本。

## 5.6 缺省参数

函数参数设置默认值之后调用时可缺省参数，但是只可以从右往左设置默认值，不能够中间空没有设置默认值的参数。

## 5.7 *inline*函数

编译器会选择性在编译时将*inline*函数在调用的地方直接替换成函数体，省去了函数调用的开销。和宏定义的区别在于：

+ 1.*inline*函数会进行参数检查

+ 2.*inline*函数在编译时展开，宏定义在预编译时替换

+ 3.*inline*函数可以进行*debug*

*inline*函数的函数体若小于调用函数产生的压栈出栈的代码大小则会是的程序占用内存变小，若函数体过大则会增大程序占用的内存大小。

由于函数体直接被替换到了调用处，编译器可根据上下文信息进行进一步的优化。

若没有使用*inline*函数，程序至函数调用处需要跳转去函数体所在位置的代码，一般函数调用位置和函数体位置并不相近，这样容易形成缺页中断。

## 5.8 函数重载

函数名称必须相同。

参数列表必须不同（个数不同、类型不同、参数排列顺序不同等）。

函数的返回类型可以相同也可以不相同。

仅仅返回类型不同不足以成为函数的重载。

*C++*通过*name mangling*来实现函数重载，以及域名空间等作用域的功能。

# 6.类

在*C++*中，结构体是由关键词*struct*定义的一种数据类型。他的成员和基类默认为公有的（*public*）。由关键词*class*定义的成员和基类默认为私有的（*private*）。这是*C++*中结构体和类仅有的区别。

## 6.1 访问控制 

*C++*类的重要属性就是封装和继承。因此，最关键的问题就是权限 的问题，*public*，*protected*，*private*控制的就是访问权限。

|                            | public | protected | private |
| -------------------------- | ------ | --------- | ------- |
| 类成员是否可以访问         | Y      | Y         | Y       |
| 友元函数是否可以访问       | Y      | Y         | Y       |
| 子类是否可以访问           | Y      | Y         | N       |
| 类的实例化对象是否可以访问 | Y      | N         | N       |

三种继承方式导致的权限变化：

|               | public    | protected | private |
| ------------- | --------- | --------- | ------- |
| public继承    | public    | protected | private |
| protected继承 | protected | protected | private |
| private继承   | private   | private   | private |

通过对象我们可以直接访问对象的成员函数以及成员变量，我们也可以通过*pointer to member*来对成员进行访问：

```cpp
class MyClass{private: int value;};
int Myclass::*ptr = MyClass::value;
MyClass mc;
mc.value = 10;
cout<<mc.*ptr<<endl;
```

## 6.2 对象

+ 1)    对象是类实例化（调用构造函数）之后的结果，仅对`public`成员有访问权限，释放时会自动调用析构函数。

+ 2)    对象模型
  + a) C++*中虚函数的作用主要是为了实现多态机制。多态，简单来说，是指在继承层次中，父类的指针可以具有多种形态——当它指向某个子类对象时，通过它能够调用到子类的函数，而非父类的函数。
  + b) 当一个类本身定义了虚函数，或其父类有虚函数时，为了支持多态机制，编译器将为该类添加一个虚函数指针（*vptr*）。虚函数指针一般都放在对象内存布局的第一个位置上，这是为了保证在多层继承或多重继承的情况下能以最高效率取到虚函数表。
  + c) 当*vprt*位于对象内存最前面时，对象的地址即为虚函数指针地址。我们可以取得虚函数指针的地址：

```cpp
Base b;
int * vptrAdree = (int *)(&b); 
cout << "虚函数指针（vprt）的地址是：\t"<<vptrAdree << endl;
```

我们强行把类对象的地址转换为*int\** 类型，取得了虚函数指针的地址。虚函数指针指向虚函数表,虚函数表中存储的是一系列虚函数的地址，虚函数地址出现的顺序与类中虚函数声明的顺序一致。对虚函数指针地址值，可以得到虚函数表的地址，也即是虚函数表第一个虚函数的地址:

```cpp
typedef void(*Fun)(void);
Fun vfunc = (Fun)*( (int *)*(int*)(&b));
cout << "第一个虚函数的地址是：" << (int *)*(int*)(&b) << endl;
cout << "通过地址，调用虚函数Base::print()：";
vfunc();
```

我们把虚表指针的值取出来： *\*(int\*)(&b)*，它是一个地址，虚函数表的地址

把虚函数表的地址强制转换成 *int\** :*(int\*) \*(int\*)(&b)*

再把它转化成我们Fun指针类型： *(Fun)\*(int \*)\*(int\*)(&b)*

这样，我们就取得了类中的第一个虚函数，我们可以通过函数指针访问它。

同理,第二个虚函数的地址为：*(int\*)(\*(int\*)(&b)+1)*

+ + d) 子类若*override*了一个父类的虚函数，其虚函数表中对应被*override*的虚函数指针会替换成自己*override*的函数，若有新增虚函数则在虚函数表后面累加新的虚函数指针。

+ 3)    对象大小
  + a) 空类的大小为1；
  + b) 类的（静态）成员函数，静态成员变量不占用类的空间；
  + c) 若有虚函数增加一个虚函数表指针的大小；
  + d) 虚继承的子类也需要加上n个父类的虚函数表指针；

## 6.3 构造函数/析构函数

+ 1)    构造函数在生成对象时调用，分为默认构造函数，拷贝构造函数，移动构造函数，赋值构造函数和移动赋值构造函数。

+ 2)    在构造函数后面加*default*关键字可将某个构造函数设置成默认的构造函数，比如若果我们没有定义构造函数，编译器会帮助我们生成默认无参且函数体为空的默认构造函数；但如果我们定义了一个有参数的构造函数，编译器便不会帮助我们生成默认构造函数，此时我们不能够像这样定义对象*MyClass mc*，但是我们可以通过添加*default*关键字恢复：
+ `MyClass() = default;`

+ 3)    在构造函数后面加*delete*关键字可将某个构造函数设为禁用，比如我们不希望对象进行拷贝构造而希望其进行移动构造这样能避免不必要的内存分配和拷贝，这样我们可在拷贝构造函数和赋值拷贝构造函数后面添加*delete*关键字。

```cpp
class A
{
public:
A(int x):x(x)
{
    n = new int[x];
    cout << "A default construction" << endl;
}
A(A &a)
{
    this->n = new int[x];
    memcpy(this->n, a.n, x * sizeof(typeid(n)));
    cout << "A copy construction" << endl;
}
A(A &&a)
{
    this->n = a.n;
    a->n = nullptr;
    cout << "A move construction" << endl;
}
A& operator=(A &a)
{
    this->n = new int[x];
    memcpy(this->n, a.n, x * sizeof(typeid(n)));
    cout << "A copy assignment construction" << endl;
    return *this;
}
A& operator=(A &&a)
{
    this->n = a.n;
    a->n = nullptr;
    cout << "A move assignment construction" << endl;
    return *this;
}
virtual void foo() {}
~A()
{
    if (this->n != nullptr) {
        delete this->n;
        this->n = nullptr;
    }
    cout << "A destruction" << endl;
}
private:
int x = 0;
int *n;
};
```

+ 1) 默认构造函数为编译器为我们默认生成的构造函数，实际上没有任何操作。

+ 2) 拷贝构造函数传入形参为对象的引用，声明对象时进行赋值或隐式赋值会调用拷贝构造函数

+ 3) 移动构造函数传入形参为对象的右值引用，将赋值对象使用*move*函数强转成右值引用之后再进行如同拷贝构造函数的调用方式时会调用移动构造函数。

+ 4) 函数返回对象会先将局部对象赋给一个临时变量，由于函数返回值为右值，所以此时调用的是移动构造函数，等局部变量析构后再将临时对象赋给函数外的对象，最后再将临时对象析构。

+ 5) 析构函数在对象到达生命周期时会自动调用，比如局部对象在函数结束时，对象指针被*delete*时。通过这种特性我们可以实现*RAII(resource acquisition is initialization)*，比如*c++11*中的*mutex_guard*:

```cpp
template <class Mutex> class lock_guard {
private:
    Mutex& mutex_;
public:
    lock_guard(Mutex& mutex) : mutex_(mutex) { mutex_.lock(); }
    ~lock_guard() { mutex_.unlock(); }
    lock_guard(lock_guard const&) = delete;
    lock_guard& operator=(lock_guard const&) = delete;
};
```

## 6.4 类的静态成员函数和变量

+ 1) 静态成员函数不能直接访问非静态成员变量，可以以传入对象的方式间接访问。

+ 2) 非静态成员函数可以调用静态成员变量，因为静态成员变量属于成哥类而非某个特定的对象，所有对象都共享该变量，在对象产生之前就有了，存储在全局静态存储区。

+ 3) 使用静态成员变量实现多个对象之间的数据共享不会破坏隐藏的原则，保证了安全性还能节省内存。

+ 4) 静态成员变量使用之前必须初始化（如`int MyClass::m_Number = 0;`）,否则链接会出错。

## 6.5 友元

+ 1) 友元函数是可以直接访问类的私有成员的非成员函数。它是定义在类外的普通函数，它不属于任何类，但需要在类的定义中加以声明，声明时只需在友元的名称前加上关键字*friend*。

+ 2) 友元函数的声明可以放在类的私有部分，也可以放在公有部分，它们是没有区别的，都说明是该类的一个友元函数。

+ 3) 一个函数可以是多个类的友元函数，只需要在各个类中分别声明。

+ 4) 友元函数的调用与一般函数的调用方式和原理一致。

+ 5) 友元类的所有成员函数都是另一个类的友元函数，都可以访问另一个类中的隐藏信息（包括私有成员和保护成员）。    

+ 6) 当希望一个类可以存取另一个类的私有成员时，可以将该类声明为另一类的友元类。

```cpp
class A
{
public:
friend class B;
};
```

## 6.6 操作符重载

+ 1) 在类中声明的成员函数操作符重载只能重载单目运算，意味着只能传入一个参数，而友元函数和普通函数则可重载双目运算符，此时可以传入两个参数。

+ 2) 使用*ostream*对象进行*<<*操作只能定义为普通函数或者友元函数，因为无法修改*ostream*对象的*<<*代码实现，此时需要传入两个参数：

```cpp
ostream& operator << (ostream& os, Test & test)
{
    os << test.a;
    return os;
}
```

## 6.7 基类

## 6.8 派生类

## 6.9 多态机制

+ 1) 多态通过虚函数来实现，在运行时根据基类指针指向的具体子类调用特定被*override*的函数，具体虚函数表原理可查看*6.2*章节。
+ 2) 若子类在需要重写的虚函数结尾加了*override*关键字则该函数一定是父类中存在可被重写的虚函数，否则在编译时会报错。

## 6.10 派生类的构造函数

+ 1) 当创建一个派生类对象时，派生类的构造函数必须首先通过调用基类的构造函数来对基类的数据成员进行初始化，然后再执行派生类构造函数的函数体，对派生类新增的数据成员进行初始化。当派生类对象的生存期结束时，析构函数的调用顺序相反。

+ 2) 派生类构造函数调用基类构造函数

隐式调用：不指定基类的构造函数，默认调用基类默认构造函数（不带参数或者带默认参数值的构造函数）

显式调用：指定调用基类的某个构造函数。除非基类有默认构造函数，否则都要用显示调用。

<派生类名>::<派生类名>(<形参声明>) : <基类名>(<参数表>)
{
<派生类构造函数的函数体>
} 

## 6.11 抽象类

抽象类即为声明了纯虚函数的类，这种类不能实例化为对象，继承这种类的派生类需要实现对应的纯虚函数才能够进行实例化，否在在编译时会报错。通过这种方式可以提供一个单纯只提供接口的父类，而不需要对对应的接口进行某种特定的实现。

`virtual void MyFunc() = 0;`

# 6.命名空间

`namespace`分为有名命名空间和无名命名空间，无名的由于没有名字所以其他文件无法引用，相当于改文件里面的`static`。`namespace`中的变量或者函数通过作用域符进行访问：

```cpp
namespace MyNamespace{
int value = 10;
void MyFunc(int value){
    cout<<”value:”<<value<<endl;
}
}
MyNamespace::MyFunc(MyNamespace::value);
```

若作用域符前面没有任何`namespace`或者类名，则表示访问的是全局变量。

# 7.Name Mangling

*mangling*的目的就是为了给重载的函数不同的签名，以避免调用时的二义性调用。如果希望*C++*编译出来的代码不要被*mangling*，可以使用*extern "C" {}*来讲目标代码包含起来，这样能使得*C++*编译器编译出的二进制目标代码中的链接符号是未经过*C++*名字修饰过的，就像*C*编译器一样。

如果想将*mangling*的符号恢复成可读的，可以使用*linux*下的*c++filt*。

如果有一些编译器*mangling*的例子：

|                            编译器                            |     `void h(int)`     |  `void h(int, char)`   |    `void h(void)`     |
| :----------------------------------------------------------: | :-------------------: | :--------------------: | :-------------------: |
|                   Intel C++ 8.0 for Linux                    |        `_Z1hi`        |        `_Z1hic`        |        `_Z1hv`        |
|                    HP aC++ A.05.55 IA-64                     |        `_Z1hi`        |        `_Z1hic`        |        `_Z1hv`        |
|                    IAR EWARM C++ 5.4 ARM                     |        `_Z1hi`        |        `_Z1hic`        |        `_Z1hv`        |
|                     GCC 3.*x* and 4.*x*                      |        `_Z1hi`        |        `_Z1hic`        |        `_Z1hv`        |
| [GCC](https://zh.wikipedia.org/wiki/GNU_Compiler_Collection) 2.9*x* |        `h__Fi`        |        `h__Fic`        |        `h__Fv`        |
|                   HP aC++ A.03.45 PA-RISC                    |        `h__Fi`        |        `h__Fic`        |        `h__Fv`        |
| [Microsoft Visual C++](https://zh.wikipedia.org/wiki/Microsoft_Visual_C%2B%2B) v6-v10 |     `?h@@YAXH@Z`      |     `?h@@YAXHD@Z`      |      `?h@@YAXXZ`      |
| [Digital Mars](https://zh.wikipedia.org/w/index.php?title=Digital_Mars&action=edit&redlink=1) C++ |     `?h@@YAXH@Z`      |     `?h@@YAXHD@Z`      |      `?h@@YAXXZ`      |
|                       Borland C++ v3.1                       |        `@h$qi`        |       `@h$qizc`        |        `@h$qv`        |
|                 OpenVMS C++ V6.5 （ARM模式）                 |        `H__XI`        |        `H__XIC`        |        `H__XV`        |
|                OpenVMS C++ V6.5 （ANSI模式）                 | `CXX$__7H__FI0ARG51T` | `CXX$__7H__FIC26CDH77` | `CXX$__7H__FV2CB06E8` |
|                    OpenVMS C++ X7.1 IA-64                    |  `CXX$_Z1HI2DSQ26A`   |  `CXX$_Z1HIC2NP3LI4`   |  `CXX$_Z1HV0BCA19V`   |
|                          SunPro CC                           |    `__1cBh6Fi_v_`     |    `__1cBh6Fic_v_`     |     `__1cBh6F_v_`     |
|                  Tru64 C++ V6.5 （ARM模式）                  |        `h__Xi`        |        `h__Xic`        |        `h__Xv`        |
|                 Tru64 C++ V6.5 （ANSI模式）                  |      `__7h__Fi`       |      `__7h__Fic`       |      `__7h__Fv`       |
|                       Watcom C++ 10.6                        |      `W?h$n(i)v`      |      `W?h$n(ia)v`      |      `W?h$n()v`       |

# 8.new/delete重载

*new operator*：指我们在C++里通常用到的关键字，比如`A* a = new A;`

*operator new*：它是一个操作符，并且可被重载(类似加减乘除的操作符重载)

当我们调用*new operator*时会调用*operator new*来分配内存，其中会调用`malloc`函数，接着会在分配内存上调用对象的构造函数，最后再返回这个指针。

关于这两者的关系，我找到一段比较经典的描述（来自于www.cplusplus.com 见参考文献：

> operator new can be called explicitly as a regular function, but in C++, new is an operator with a very specific behavior: An expression with the new operator, first calls function operator new (i.e., this function) with the size of its type specifier as first argument, and if this is successful, it then automatically initializes or constructs the object (if needed). Finally, the expression evaluates as a pointer to the appropriate type.

通过重载这两个运算符我们可以记录内[存分配删除信息来构建内存池，或者添加一些打印信息的功能。

```cpp
#include <iostream>
using std::cout;
using std::endl;

void* operator new(size_t size, const char *file, unsigned int line) {
	cout << "file:" << file << " line:" << line << " size:" << size << " Call ::operator new" << endl;
	void* tmp = malloc(size);
	return tmp;
}
void operator delete(void *ptr, const char *file, unsigned int line) {
	cout << "Call ::operator delete" << endl;
	free(ptr);
}
class A {
public:
	A() {
		cout << "A constructor" << endl;
	}
	~A()
	{
		cout << "A destructor" << endl;
	}
private:
	int a[20];
};

//#define new new(__FILE__,__LINE__)

int main()
{
#ifndef new
	A *a = reinterpret_cast<A*>(operator new(sizeof(A), __FILE__, __LINE__));
	new(a) A;
	delete a;

	A *b = new(__FILE__, __LINE__) A;//这么写会调用重载的operator new 再调用A的构造函数
	delete b;
#else
	A *b = new A;
	delete b;
#endif
	system("pause");
	return 0;
}

```

# 9.类型转换

*C++*同样支持*C*风格的强制转换(*Type Cast*):`TypeName b = (TypeName)a;`

*C++*的四种类型转换：

* 1.*const_cast*

  * 常量指针被转化成非常量的指针，并且仍然指向原来的对象

  * 常量引用被转换成非常量的引用，并且仍然指向原来的对象
  * 常量指针是指向常量的指针，指指针指向的内容不能改变，声明为`const int *p;`或者`int const *p`;
  * 指针常量是指针的值为常量，指指针指向的地址不能改变，声明为`int * const p = &a;`

* 2.*static_cast*

  * *static_cast* 作用和C语言风格强制转换的效果基本一样，由于没有运行时类型检查来保证转换的安全性，所以这类型的强制转换和C语言风格的强制转换都有安全隐患。
  * 用于类层次结构中基类（父类）和派生类（子类）之间指针或引用的转换。注意：进行上行转换（把派生类的指针或引用转换成基类表示）是安全的；进行下行转换（把基类指针或引用转换成派生类表示）时，由于没有动态类型检查，所以是不安全的。
  * 用于基本数据类型之间的转换，如把*int*转换成*char*，把*int*转换成*enum*。这种转换的安全性需要开发者来维护。*
  * *static_cast*不能转换掉原有类型的*const*，*volatile*、或者 *__unaligned*属性。(前两种可以使用*const_cast* 来去除)
  * 在*c++ primer*中说道：*C++* 的任何的隐式转换都是使用 *static_cast* 来实现。

* 3.*dynamic_cast*

  * 该转换涉及面向对象的多态性和程序运行时的状态,也与编译器的属性设置有关.所以不能完全使用C语言的强制转换替代

  * 从子类到基类不会有任何问题

  * 从基类到子类会对*RTTI*(*runtime type information*)进行检查，由于需要*RTTI*，所以需要有虚函数表，所以需要有虚函数，若子类拥有父类没有的成员，则函数会返回*nullptr*。*typeid*运算符也是读取*RTTI*的机制，同样需要有虚函数，在生成虚表之后上面会挂一个*type_info*结构体，通过这个结构体父类指针在运行时可以读取出相关的信息进行判断。而*sizeof*运算符则是在编译时求值，只关心静态声明的类型，不过也可以有运行时的语义，当*sizeof*的参数是[Variable-Length Array](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Variable-length_array)时。

    > a) If expression is a [glvalue expression](https://link.zhihu.com/?target=http%3A//en.cppreference.com/w/cpp/language/value_category) that identifies an object of a polymorphic type (that is, a class that declares or inherits at least one [virtual function](https://link.zhihu.com/?target=http%3A//en.cppreference.com/w/cpp/language/virtual)), the typeid expression evaluates the expression and then refers to the [std::type_info](https://link.zhihu.com/?target=http%3A//en.cppreference.com/w/cpp/types/type_info) object that represents the dynamic type of the expression. If the glvalue expression is obtained by applying the unary * operator to a pointer and the pointer is a null pointer value, an exception of type [std::bad_typeid](https://link.zhihu.com/?target=http%3A//en.cppreference.com/w/cpp/types/bad_typeid) or a type derived from [std::bad_typeid](https://link.zhihu.com/?target=http%3A//en.cppreference.com/w/cpp/types/bad_typeid) is thrown.

* 4.*reinterpret_cast*

  * *reinterpret_cast*是强制类型转换符用来处理无关类型转换的，通常为操作数的位模式提供较低层次的重新解释。但是他仅仅是重新解释了给出的对象的比特模型，并没有进行二进制的转换。他是用在任意的指针之间的转换，引用之间的转换，指针和足够大的*int*型之间的转换，整数到指针的转换。

# 10.异常机制

# 11.*auto*

* 对于*auto*而言，其意义在于*type deduce*，所以它不会允许没有初始化的声明，这样对于开发者来说是一个很好的习惯；

* 另外一个意义是简化代码，尤其是经常使用容器迭代器初始化；

* 如果想保存*lambda*表达式，不需要自己推导一个函数指针或者*function*类型变量，直接使用*auto*即可；

  ```cpp
  auto closure = [](const int&, const int&) {}
  ```

* 在*C++14*的泛型*lambda*表达式中，我们还可以将参数定义成*auto*类型，通过这个我们可以实现*ChurchNumber*，后续在13章作介绍；

  ``` cpp
  auto closure = [](auto x, auto y) { return x * y;}
  ```

* 在*C++1X*中，配合*decltype*可以解决原来难以让编译器自动推导模板函数返回值的问题，原来我们需要先转换成万能的0，再转成指针，最后再取指针：

  ```cpp
  template<class T,class U>
  decltype(*(T*)(0) * *(U*)(0)) mul(T x, U y)
  {
      return x*y;
  }
  ```

  *C++11*可以直接这样：

  ```cpp
  template<class T,class U>
  auto mul(T x, U y)->decltype(x * y)
  {
      return x*y;
  }
  ```

  而在*C++14*中我们可以直接这样：

  ```cpp
  template<class T,class U>
  decltype(auto) mul(T x, U y)
  {
      return x*y;
  }
  ```

  其中的*auto*在编译时会替换成函数返回的表达式，再通过*decltype*来推导其类型。

* 当不声明为引用类型时，*auto*的初始化表达式即使是引用，编译器也并不会将该变量推导为对应类型的引用，*auto*的推导结果等同于初始化表达式去除引用和*const qualifier*，所以在实际编码中我们需要显示的声明引用和指针。当声明为引用或指针时*auto*的推导结果能推导出正确的类型，其结果能保留初始化表达式的*qualifier*：

  ```cpp
  #include <iostream>
  using namespace std;
  int main()
  {
      int x = 0;
      auto *a = &x;//类型为int*，auto为int
      ++(*a);
      cout << "after ++(*a) *a:" << *a << " x:" << x << endl;
      auto b = &x;//类型为int*，auto为int*
      ++(*b);
      cout << "after ++(*b) *b:" << *b << " x:" << x << endl;
      auto &c = x;//类型为int&，auto为int
      ++c;
      cout << "after ++c c:" << c << " x:" << x << endl;
      auto d = c;//类型为int，auto为int
      ++d;
      cout << "after ++d d:" << d << " x:" << x << endl;
  
      const auto e = x;//类型为const int，auto为int
      //++e; error C3892: “e”: 不能给常量赋值
      auto f = e;//类型为int，auto为int
      ++f;
      cout << "after ++f f:" << f << " x:" << x << endl;
      const auto &g = x;//类型为const int&，auto为int
      //++g; error C3892: “g”: 不能给常量赋值
      auto& h = g;//推导为const int&,auto为const int
      //++h; error C3892: “g”: 不能给常量赋值
      auto *i = &e; //推导为 const int*，常量指针，不能改变指针指向的内容
      //++(*i);  error C3892: “i”: 不能给常量赋值
      auto j = i;
      //++(*j); error C3892: “j”: 不能给常量赋值
      return 0;
  }
  ```

* *auto*有时候可能得不到我们预想的类型，其实主要是因为基础不够扎实，比如*vector<bool>*。*vector<bool>*是*vector<T>*的一个特例化(*template specialization*)，这个特例化要解决的问题是存储容量的问题。

  *To optimize space allocation, a specialization of vector for bool elements is provided.*

  所以，它一般来说是以位的方式来存储*bool*的值。从这里我们可以看出，如果使用位来提升空间效率可能引出的问题就是时间效率了，因为我们的计算机地址是以字节为单位的，根据网友的实验遍历访问*vector<bool>*要比其他*vector<int>*耗时40倍以上。

  对于*vector<bool>*来说，它的*operator[]*返回的不是对应元素的引用，而是一个*member class std::vector< bool>::reference* 。对于普通*vector<T>*来说，若是使用*auto*声明变量对保存*opertor[]*返回的值，会如同上一段所说的去除引用，而对于*vector<bool> operator*返回的*std::vector< bool>::reference*是一个*member class*：

  > *This embedded class is the type returned by members of non-const [vector](http://www.cplusplus.com/vector) when directly accessing its elements. It accesses individual bits with an interface that emulates a reference to a `bool`.*

  根据定义这个类可以以*bit*为单位访问对应的*bool*，并且是以引用的方式，所以*auto*声明得到的*vector<bool>::operator[]*返回值是可以改变*vector<bool>*对应元素的内容的。如果我们试图用*auto&*来定义一个*vector<bool>::operator[]*返回的值，会出现如下编译错误：

  > error: invalid initialization of non-const reference of
  >  type 'std::_Bit_reference&' from an rvalue of type 'std::_Bit_iterator::referen
  > ce {aka std::_Bit_reference}'

  这是因为对于*std::vector< bool>::reference*这种*proxy reference*来说，我们对其*dereference*并不会得到一个普通的*bool &*，取而代之的是一个临时对象，所以取引用会导致编译错误。这时候我们使用右值引用来定义这个变量可以解决*vector<bool>*想遍历修改的问题，右值引用对*vector<T>*的其他类型同样适用：

  ```cpp
  vector<bool> v = {true, false, false, true};
  // Invert boolean status
  for (auto&& x : v)  // <-- note use of "auto&&" for proxy iterators
      x = !x;
  ```

# 12.移动语义和右值引用

在*6.2*章节构造函数章节中我们介绍了移动构造函数，入参中的*A &&a*即为右值引用，当我们使用*move*函数将变量转换成右值引用之后再进行构造函数便会调用我们的移动构造函数。我们一般会在移动构造函数中实现移动语义的功能，就是不会新分配一块内存，而是将旧的内存直接赋给新的对象，并将旧的对象的指针赋成*nullptr*：

```cpp
A(A &&a)
{
    this->n = a.n;
    a->n = nullptr;
    cout << "A move construction" << endl;
}
```

右值引用使*C++*标准库的实现在多种场景下消除了不必要的额外开销（如*std::vector*,*std::string*），也使得另外一些标准库（如*std::unique_ptr*,*std::function*）成为可能。右值引用的意义通常为两大作用：移动语义和完美转发。移动语义即为上述所示的移动构造函数，*std::vector*和*std::string*也可以通过移动构造函数来构造，这样就避免了重新给vetor或者string分配空间的开销：

```cpp
std::vector<int> vec(100,1);
std::vector<int> vec1 = move(vec);
```

而对于模板函数来说，`void func(T&& param)`其中的*T&&*并不一定表示的是右值，它绑定的类型是未知的，既可能是右值，也可能是左值，需要类型推导之后才会知道，比如：

```cpp
template<typename T>
void func(T&& param){}
func(10); // 10是右值
int x = 10;
func(x);  // x是左值
```

这种未定的引用类型称为万能引用(*universal reference*)，这种类型必须被初始化，具体是什么类型取决于它的初始化。由于存在*T&&*这种未定的引用类型，当它作为参数时，有可能被一个左值引用或右值引用的参数初始化，这是经过类型推导的*T&&*类型，相比右值引用(*&&*)会发生类型的变化，这种变化就称为引用折叠，引用折叠规则如下：

* 1.所有右值引用折叠到右值引用上仍然是一个右值引用。（*T&& &&*变成 *T&&*）
* 2.所有的其他引用类型之间的折叠都将变成左值引用。 （*T& &* 变成 *T&*; *T& &&* 变成 *T&*; *T&& &* 变成 *T&*）

对于万能引用，我们可能需要知道它什么时候是右值引用什么时候是左值引用，这时候我们就需要完美转发*std::forward<T>()*。如果传进来的参数是一个左值，*enter*函数会将T推导为*T&*，*forward*会实例化为*forward<T&>*，*T& &&*通过引用折叠会成为*T&*，所以传给*func*函数的还是左值；如果传进来的是一个右值，*enter*函数会将*T*推导为*T*，*forward*会实例化为*forward<T>*，*T&&*通过引用折叠还是*T&&*，所以传给*func*函数的还是右值：

``` cpp
template<typename T>
T&& forward(typename remove_reference<T>::type& param)
{
    return static_cast<T&&>(param); //会发生引用折叠
}

template <typename T>
void func(T t) {
    cout << "in func " << endl;
}

template <typename T>
void enter(T&& t) {
    cout << "in enter " << endl;
    func(std::forward<T>(t));
}
```

# 13.*lambda*表达式

*lambda*表达式对于*C++*的意义有两条：

* 第一条是可以在表达式中直接定义一个函数，而不需要将定义函数和表达式分开。
* 第二条是引入了闭包。闭包是指将当前作用域中的变量通过值或者引用的方式封装到*lambda*表达式当中，成为表达式的一部分，它使*lambda*表达式从一个普通的函数变成一个带隐藏参数的函数。

基本语法我们在函数章节有所介绍：

```cpp
[capture list] (params list) -> return type {function body};
```

[]中我们可以定义好闭包变量的捕获方式：

```cpp
[]        //未定义变量.试图在Lambda内使用任何外部变量都是错误的.
[x, &y]   //x 按值捕获, y 按引用捕获.
[&]       //用到的任何外部变量都隐式按引用捕获
[=]       //用到的任何外部变量都隐式按值捕获
[&, x]    //x显式地按值捕获. 其它变量按引用捕获
[=, &z]   //z按引用捕获. 其它变量按值捕获
```

()中我们定义了需要直接直接传入*lambda*表达式的形参；

->之后我们定义的是返回值类型，当然我们可以将其隐藏，编译器会根据 *return*表达式进行类型推导。除了返回值可以类型推导，在*C++14*中我们使用generic lambda还能对形参进行类型推导；

通过闭包和返回值以及形参的类型推导我们可以进行函数式编程，比方说构造*Church Number*：

```cpp
#include <iostream>
using namespace std;
//定义0函数
auto zero = [](auto f){
    return [=](auto x){
        return x;
    };
};
//定义后继函数
auto succ = [](auto num){
    return [=](auto f){
        return [=](auto x){
        	return f(num(f)(x));
        };
    };
};
//定义加法函数
auto add = [](auto m, auto n){
    return [=](auto f){
        return [=](auto x){
            return m(f)(n(f)(x));
        };
    };
};
//定义乘法函数
auto mul = [](auto m, auto n){
    return [=](auto f){
        return [=](auto x){
            return m(n(f))(x);
        };
    };
};
int main()
{
    auto two = succ(one);//通过后继函数得到2
    auto three = add(one, two);//通过加法函数得到3
    auto six = mul(two, three);//通过乘法函数得到6
    auto f = [](auto x){cout<<"x:"<<x<<endl; return x+1;}//为了能具象数字，我们定义f
    six(f)(0);
    return 0;
}
```

# 14.智能指针

# 15.算法增强

