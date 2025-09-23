---
title: 春招寄语
date: 2023-03-04 15:14:23
tags:
math: true
typora-root-url: ./..
---

图形学全忘光了，技美笔试直接寄了。  
把面试题整理一下吧……  
对于**多线程**和**图形API**了解不够需要继续学习

<!--more-->

# C++ 部分

## [new 和 malloc 区别](https://www.cnblogs.com/ywliao/articles/8116622.html)

### 1.申请内存所在位置

* new 从 free store 动态分配内存空间，free store 能否是堆取决于 new 的实现其可以是堆，还可以是静态存储区。
  
* malloc 从堆上分配内存
  
### 2.返回类型安全性
* new 分配内存成功时，返回对象类型的指针，符合类型安全。
  
* malloc 返回 void* ，需要强制类型转换。
  
### 3.内存分配失败时返回值
* new 失败返回 bac_alloc 异常，不会返回 NULL；  
* malloc 失败返回 NULL；
  
### 4.是否需要指定内存大小
* new 无需指定，编译器自己计算
  
* malloc 需要自己显示指定。
  
  ```c  
  A *ptr = (A*)malloc(sizeof(A));  
  ```
  
### 5.是否调用构造函数/析构函数
* new 会经过三步分配内存  
  * 调用 operator new (数组是 operator new[] )，分配足够大、原始、未命名的空间。  
  * 编译器运行对应构造函数，传入初值。  
  * 构造完成，返回一个指向该对象的指针。  
* 在 delete 会  
  * 调用析构函数  
  * 编译器调用 operator delete / operator delete[]
  
### 6.对数组处理
* C++ 有 new[] 和 delete [] 处理数组
  
* 而 malloc 不知道你存放什么，只会给一块原始的内存，所以需要我们自己指定
  
  ```c  
  int *ptr = (int*)malloc(10*sizeof(int)); // 分配10个int，作为数组  
  ```
  
### 7. new 和 malloc 是否可以相互调用

new delete可以基于 malloc 实现，但是 malloc 不能调用 new！！！

### 8.是否可以重载

* operator new / operator delete 可以重载  
* malloc / free 不能重载
  
### 9.能够直观地重新分配内存

malloc 分配的内存如果不够了可以 realloc

但是 new 不行！

### 10.客户处理内存分配不同

operator new 抛出异常前会调用一个用户指定的错误处理函数 new-handler，指向一个错误处理函数。

malloc 异常用户只能看着他 NULL

## 类和接口

如果把类比作一把枪，接口就是枪的配件，比如握把，枪托等等。我们对于类重点在于描述一个对象，而对于接口重点在于描述接口能提供的功能，所以接口内没有数据成员，只有成员函数。



## C++ 类

* 子类和父类有同名成员数据，是通过"::"区分  
  ~~~c++  
  class A  
  {  
  public:  
      int num;  
  }  
  class B:public A  
  {  
  public:  
      int num;  
  }  
  ~~~
  
  其实这两个`num`实际的名字是`A::num`和`B::num`
  
* 类的成员函数调用  
  非虚函数，根据调用对象类型决定，虚函数根据实际类型决定。
  
  ~~~c++  
  class T  
  {  
  public:  
      virtual void Fun1() { std::cout << "T" << std::endl; }  
  };
  
  class A  
  {  
  public:  
      int num;  
  public:  
      A() { num = 2; std::cout << "A::A " << num++<<std::endl; }  
      virtual void Fun1() { std::cout << "A::Fun1 " << ++num<< std::endl; }  
      void Fun2(){ std::cout << "A::Fun2 " << --num << std::endl; }  
      void Fun3(){ std::cout << "A::Fun3 " << num-- << std::endl; }  
      ~A() { std::cout << "~A() " << --num << std::endl; }  
  };
  
  class B :public A  
  {  
  public:  
      int num;  
  public:  
      B() { num = 100; std::cout << "B::B " << num++ << std::endl; }  
      void Fun1() { std::cout << "B::Fun1 " << ++num << std::endl; }  
      void Fun2() { std::cout << "B::Fun2 " << --num << std::endl; }  
      virtual void Fun3() { std::cout << "B::Fun3 " << num-- << std::endl; }  
      ~B() { std::cout << "~B() " << --num << std::endl; }  
  };
  
  int main()  
  {  
      A* ptr = new B;  
      ptr = dynamic_cast<A*>(ptr);  
      ptr->Fun1();  
      ptr->Fun2();  
      ptr->Fun3();  
  }  
  ~~~
  
  ~~~c++  
  A::A 2  
  B::B 100  
  B::Fun1 102  
  A::Fun2 2  
  A::Fun3 2  
  ~B() 101  
  ~A() 0  
  ~~~
  
  `A* ptr = new B;`  
  `ptr`类型为`A*`，但是实际指向对象类型其实为`B`  
  那么其实在内存空间中有如下分配
  
  ~~~  
  成员数据:  
  n_ptr  
  A::num  
  B::num  
  一张A的虚函数表？  
  一张B的虚函数表  
  函数映射：  
  直接查找的  
  A::Fun2  
  A::Fun3  
  B::Fun1  
  B::Fun2  
  ~~~

  
  
## 构造函数链和析构函数链

### 构造函数链

一个类中有父类成员，成员变量。对于构造函数我们

1. 先调用父类，  
2. 再调用成员变量的构造函数，  
3. 最后调用子类构造函数在构造过程中

子类是继承父类的成员

~~~c++
class A
{
    int a;
}

class B : public A
{
	int b;    
}
~~~

这里B有成员b，和从父类继承的a。  
因为 a 是父类成员，所以交由父类初始化，如果先调子类，子类成员有a，b，对a，b初始化，再调父类，a又初始化初始化两次。  
对于成员变量，成员变量有自己的构造函数，是当前子类无需关心的，对于他们先交给他们自己初始化即可。

## 析构函数链

对于析构函数链，我们把子类可以看成一个套娃。  
我们装套娃会从小到大(从父到子)

## C++的public、protected、private和三种继承

public 表示大家谁都能来访问，包括类对象自己和外部。  
protected,外部不能访问，但是自己和子类可以访问。  
private，表示只能自己访问，自己的子类都不能访问。  
public 继承，不改变父类成员的可见性。  
protected 继承，父类成员除private外，全部变为protect  
private 继承全部变为private
>假如把父类比喻一个门派，期内对象就是武功秘籍  
>public 就像比较普通的武功，比如太祖长拳，大家都可以来学。  
>protect 就像门派内的高深强大的武功秘籍，比如降龙十八掌，只有你是下一任掌门才能学。  
>private 就像掌门自己偷学了邪门歪道，比如岳不群学《辟邪剑谱》，只能自己知道，外人不能晓得。除了自己谁也不让看。  
>  
>public 继承就像是普普通通比较开明的继承者，子承父业，原来掌门规定的哪些武功可以交给外人，哪些武功自己人学，都不变，但是掌门私底下学的歪门武功他也不知道是什么，对他而言不可见，想知道是什么得返回去用上一任掌门的去查。  
>protect 继承就像垄断或者有点小心思的人继承，我不让所有人随便学了，现在你们想从我这学你得是我的关门弟子下一任掌门。  
>private 继承就像断代了，诶，我谁也不教，全部变成private，只有我自己看，但是无论这三种哪一种都看不到父类的private。

## static关键字

static关键字表示静态元素，其在程序的生存周期中仅在**静态存储区**分配一次存储空间，直到程序的生存周期结束。  
static常用于

* 函数中的静态变量  
* 静态类对象  
* 类中的静态成员变量  
* 类中的静态方法
  
## **static 和const分别怎么用，类里面static和const可以同时修饰成员函数吗**
- **static**
  
  - static对于变量
  
    1. 局部变量
  
       在局部变量之前加上关键字static，局部变量就被定义成为一个局部静态变量。
  
       内存中的位置：静态存储区
  
       初始化：局部的静态变量只能被初始化一次
  
       作用域：作用域仍为局部作用域，当定义它的函数或者语句块结束的时候，作用域随之结束。
  
       > 当static用来修饰局部变量的时候，它就改变了局部变量的存储位置（从原来的栈中存放改为静态存储区）及其生命周期（局部静态变量在离开作用域之后，并没有被销毁，而是仍然驻留在内存当中，直到程序结束，只不过我们不能再对他进行访问），但未改变其作用域。
  
    2. 全局变量
  
       在全局变量之前加上关键字static，全局变量就被定义成为一个全局静态变量。
  
       内存中的位置：静态存储区（静态存储区在整个程序运行期间都存在）
  
       初始化：未经初始化的全局静态变量会被程序自动初始化为0
  
       作用域：全局静态变量在声明他的文件之外是不可见的。准确地讲从定义之处开始到文件结尾。(只能在本文件中存在和使用)
  
       > 全局变量本身就是静态存储方式， 静态全局变量当然也是静态存储方式。两者的区别在于非静态全局变量的作用域是整个源程序， 当一个源程序由多个源文件组成时，非静态的全局变量在各个源文件中都是有效的（在其他源文件中使用时加上extern关键字重新声明即可）。 而静态全局变量则限制了其作用域， 即只在定义该变量的源文件内有效， 在同一源程序的其它源文件中不能使用它。
  
- static对于函数
  
  修饰普通函数，表明函数的作用范围，仅在定义该函数的文件内才能使用。在多人开发项目时，为了防止与他人命名空间里的函数重名，可以将函数定位为 static。（和全局变量一样限制了作用域而已）
  
- static对于类
  
  1. 成员变量
  
     用static修饰类的数据成员实际使其成为类的全局变量，会被类的所有对象共享，**包括派生类的对象**。
  
     因此，static成员必须在类外进行初始化，而不能在构造函数内进行初始化。不过也可以用const修饰static数据成员在类内初始化 。
  
  2. 成员函数
  
     用static修饰成员函数，使这个类只存在这一份函数，所有对象共享该函数，**不含this指针。**
  
     静态成员是可以独立访问的，也就是说，无须创建任何对象实例就可以访问。
  
     **不可以同时用const和static修饰成员函数。**
  
- **const**
  
  1. 限定变量为不可修改。  
  2. 限定成员函数不可以修改任何数据成员
  
- static和const可以同时修饰成员函数吗?
  
  答：不可以。C++编译器在实现const的成员函数的时候为了确保该函数不能修改类的实例的状态，会在函数中添加一个隐式的参数const this*。但当一个成员为static的时候，该函数是没有this指针的。也就是说此时const的用法和static是冲突的。两者的语意是矛盾的。**static的作用是表示该函数只作用在类型的静态变量上，与类的实例没有关系；而const的作用是确保函数不能修改类的实例的状态**，与类型的静态变量没有关系。因此不能同时用它们。
  
## 拷贝构造函数

**只有这三种情况！！！！！**

### 1.用一个对象初始化另一个对象。

```c++  
Point p2(p1);  
Point p3 = p1  
```

这两种情况一样。

### 2.若一个函数的形参是一个 Class 对象，当 F 被调用时，Class 拷贝构造函数调用。

```c++  
void Func(Point a){};
  
int mian()  
{  
    Point a;  
    Func(a);    // 调用拷贝构造函数  
}  
```

### 3.如果函数的返回值是 Class 对象，函数返回时，调用 Class 拷贝构造函数

即返回值对象由拷贝构造函数初始化。

## 模板函数

### 模板函数的声明实现为什么必须在一个文件内

[CSDN : 模板函数的声明和定义为何不能分开放在两个文件中?](https://blog.csdn.net/chigusakawada/article/details/78752668)

##  C++ 11 特性

### 左值右值

[四行代码！看懂右值引用](https://www.cnblogs.com/qicosmos/p/4283455.html)

#### 右值引用的特点：

1. 通过右值引用的声明，右值又“重获新生”，其生命周期与右值引用类型变量的生命周期一样长，只要该变量还活着，该右值临时量将会一直存活下去。  
   ~~~c++  
   A GetA()  
   {  
       return A();  
   }  
   A&& a = GetA();	// 这里只会调用一次拷贝构造函数，就是return A()，  
   // 因为A&& 延长了 GetA的生命周期无需拷贝函数  
   ~~~
  
2. 右值引用独立于左值和右值。意思是右值引用类型的变量可能是左值也可能是右值。
  
3. T&& t在发生自动类型推断的时候，它是未定的引用类型（universal references），如果被一个左值初始化，它就是一个左值；如果它被一个右值初始化，它就是一个右值，它是左值还是右值取决于它的初始化。  
   正是因为右值引用可能是左值也可能是右值，依赖于初始化，并不是一下子就确定的特点，我们可以利用这一点做很多文章，比如后面要介绍的移动语义和完美转发。
  
4. 

移动语义的实现：

~~~c++
A GetA()
{
    return A;
}

A(A&& a):m_ptr(a.m_ptr){}

A a = GetA();
~~~

利用右值引用作为参数，因为`GetA`返回值是右值，而`A&&`类型确定不发生自动推断即确定的右值，拷贝函数会匹配到`A(A&& a)`，这样就省去了重新`new m_ptr`。

##### 引用折叠

- 所有的右值引用叠加到右值引用上仍然还是一个右值引用；  
- 所有的其他引用类型之间的叠加都将变成左值引用。
  
#### 一个移动语义应用
实现移动构造和移动赋值  
为什么在 移动构造中将 f.ptr 赋值为 nullptr  
因为我们的移动构造函数参数是一个右值，在完成初始化的任务后就应该消失，所以把他所指内存交给新对象，并指向了nullptr，移动赋值也是同理。  
移动赋值和移动复制通常联合使用，目的是优化对象的复制和赋值操作，提高程序的性能。  
移动赋值会在这时调用`f1=Foo()`，f1是之前声明过的，Foo()返回右值，匹配到`=(Foo&&)`函数。
~~~c++
class Foo
{
public:
    int a;
    int *ptr;

public:
    Foo(Foo&& f):ptr(f.ptr),a(f.a)
    {
        f.ptr = nullptr;
    }
    
    Foo& operator =(Foo&& f)
    {
        if(this != &f)
        {
            this->ptr = f.ptr;
            f.ptr = nullptr;
            a = f.a;
        }
        return *this;
    }
};
~~~

### 一些补充

~~~c++
vec.push(Foo());
~~~

首先，我们知道函数的返回值是右值，所以Foo() 这部分是个右值  
当我们没有右值引用的移动构造函数时，我们会发生 右值转换的过程——把右值转化为左值，调用复制构造函数。  
若我们有参数为右值引用的移动构造函数，右值会直接匹配到这个函数，不会发生一次转换去匹配参数为左值的复制构造函数。  
这里的知识点是右值可以转换为左值。

### 智能指针



### 类型推断



### lambda表达式



### 类型转换

[九阳神功！不会C++就看这个！](https://zhuanlan.zhihu.com/p/417640759)（1）const_cast: 把const属性去掉，即将const转换为非const（也可以反过来），const_cast只能用于指针或引用，并且只能改变对象的底层const（顶层const，本身是const，底层const，指向对象const）；

（2）static_cast: 隐式类型转换，可以实现C++中内置基本数据类型之间的相互转换，enum、struct、 int、char、float等，能进行类层次间的向上类型转换和向下类型转换（向下不安全，因为没有进行动态类型检查）。它不能进行无关类型(如非基类和子类)指针之间的转换，也不能作用包含底层const的对象；

（3）dynamic_cast：动态类型转换，用于将基类的指针或引用安全地转换成派生类的指针或引用（也可以向上转换），若指针转换失败返回NULL，若引用返回失败抛出bad_cast异常。dynamic_cast是在运行时进行安全性检查；使用**dynamic_cast父类一定要有虚函数，否则编译不通过；**

（4）reinterpret_cast：reinterpret是重新解释的意思，此标识符的意思即为将数据的二进制形式重新解释，但是不改变其值，有着和C风格的强制转换同样的能力。它可以转化任何内置的数据类型为其他任何的数据类型，也可以转化任何指针类型为其他的类型。它甚至可以转化内置的数据类型为指针，无须考虑类型安全或者常量的情形。不到万不得已绝对不用（比较不安全）

## C++ struct和class区别

* 在 **C 语言** 中，**结构体** 只能存放一些 **变量** 的集合，并不能有 **函数**，但 **C++** 中的结构体对 C 语言中的结构体做了扩充，可以有函数，因此 C++ 中的结构体跟 C++ 中的类很类似。C++ 中的 struct 可以包含成员函数，也能继承，也可以实现多态。
  
* 但在 C++ 中，使用 class 时，类中的成员默认都是 **private** 属性的，而使用 struct 时，结构体中的成员默认都是 public 属性的。
  
* class 继承默认是 private 继承，而 struct 继承默认是 public 继承。
  
* C++ 中的 class 可以使用模板，而 struct 不能使用模板。
  
## C++ vector

vector.clear 不释放空间

- int size() const:返回向量中元素的个数  
- int capacity() const:返回当前向量所能容纳的最大元素值  
- int max_size() const:返回最大可允许的vector元素个数值
  
## C++智能指针和安全性

智能指针主要解决一个内存泄露的问题，它可以自动地释放内存空间。因为它本身是一个类，当函数结束的时候会调用析构函数，并由析构函数释放内存空间。智能指针分为共享指针(shared_ptr), 独占指针(unique_ptr)和弱指针(weak_ptr)：

（1）shared_ptr ，多个共享指针可以指向相同的对象，采用了引用计数的机制，当最后一个引用销毁时，释放内存空间；

（2）unique_ptr，保证同一时间段内只有一个智能指针能指向该对象（可通过move操作来传递unique_ptr）；

（3）weak_ptr，用来解决shared_ptr相互引用时的死锁问题，如果说两个shared_ptr相互引用,那么这两个指针的引用计数永远不可能下降为0,资源永远不会释放。它是对对象的一种弱引用，不会增加对象的引用计数，和shared_ptr之间可以相互转化，shared_ptr可以直接赋值给它，它可以通过调用lock函数来获得shared_ptr。

- shared_ptr的实现原理是什么？构造函数、拷贝构造函数和赋值运算符怎么写？shared_ptr是不是线程安全的？
  

（1）shared_ptr是通过引用计数机制实现的，引用计数存储着有几个shared_ptr指向相同的对象，当引用计数下降至0时就会自动销毁这个对象；

（2）具体实现：

1）构造函数：将指针指向该对象，引用计数置为1；

2）拷贝构造函数：将指针指向该对象，引用计数++；

3）赋值运算符：=号左边的shared_ptr的引用计数-1，右边的shared_ptr的引用计数+1，如果左边的引用技术降为0，还要销毁shared_ptr指向对象，释放内存空间。

（3）shared_ptr的引用计数本身是安全且无锁的，但是它指向的对象的读写则不是，因此可以说shared_ptr不是线程安全的。[shared_ptr是线程安全的吗？ - 云+社区 - 腾讯云 (tencent.com)](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/developer/article/1654442)

- weak_ptr是为了解决shared_ptr的循环引用问题，那为什么不用raw ptr来解决这个问题？
  

答：一个weak_ptr绑定到shared_ptr之后不会增加引用计数，一旦最后一个指向对象的shared_ptr被销毁，对象就会被释放，即使weak_ptr指向对象，也还是会释放；raw指针，当对象销毁之后会变成悬浮指针。

## Shared_ptr实现原理



### 智能指针如何实现

C++中的智能指针（smart pointers）是一种RAII（Resource Acquisition Is Initialization）技术的实现方式，它们可以自动管理内存资源，并确保在对象离开作用域时正确地释放这些资源。智能指针的主要目的是确保资源获取与对象初始化同时发生，从而能够创建该对象的所有资源并在某行代码中准备就绪。

C++11中引入了三种智能指针：`std::unique_ptr`、`std::shared_ptr`和`std::weak_ptr`。其中`std::unique_ptr`是用于独占式拥有一个对象，`std::shared_ptr`是用于共享拥有一个对象，而`std::weak_ptr`则是用于弱引用一个对象。

当我们需要使用RAII技术来管理动态分配的内存时，我们通常使用`std::unique_ptr`来管理指向单个对象的指针，并使用`std::shared_ptr`来管理指向共享资源的指针。如果我们需要在一个对象中存储多个指向共享资源的指针，则应使用`std::weak_ptr`来避免循环引用问题。

总之，智能指针是一种非常有用的C++编程工具，可以帮助我们更安全、更简单地管理内存资源，并且对于编写高质量的C++代码来说至关重要。更多关于智能指针的资料可以参考[[3](https://zhuanlan.zhihu.com/p/150555165)][[4](https://learn.microsoft.com/zh-cn/cpp/cpp/smart-pointers-modern-cpp?view=msvc-170)]。



## 为什么析构函数一定要被设置为虚函数

[为什么析构函数必须是虚函数？为什么C++默认的析构函数不是虚函数](https://www.cnblogs.com/yuanch2019/p/11625460.html)

首先类的**虚函数**调用是靠**虚函数指针**调用的，而函数成员靠的是对象类型。  
~~~c++
class A
{
public:
    void func1(){};
    virtual void func2(){};
};

int main(){
    A a;
    a.func1();
    a.func2();
}
~~~

对于`func1`是通过`a.this`调用，而`func2`是通过虚函数指针`a._vfptr->func2()`调用。(大概是这个意思，但应该是别的形式)

>“Note：定义在类内部的函数是隐式的inline函数（参见6.5.2节，第214页）。” —— 《C++ Primer》 中文第五版 P230  
>  
>“成员函数通过一个名为**this**的额外隐式参数来访问调用它的那个对象。” —— 《C++ Primer》 中文第五版 P231

**成员函数本质上可以看做全局函数，不过第一个参数固定为this。**

### 菱形继承

https://blog.csdn.net/tounaobun/article/details/8443228

问题：A->B;A->C;B,C->D; B,C继承了A，D继承了A。  
在调用D的成员函数时候就不知道是调用B的，还是C的，所以在继承中使用

~~~c++
class Animal
{
public:
    void func() { cout << "Animal"; };
};

class Lion : virtual public Animal
{
};

class Wolf : virtual public Animal
{
};

class Cat : public Wolf, public Lion
{
};
~~~

加上virtual保证子类只有一个父类的子对象，防止发生二义性。

## C++ 深拷贝浅拷贝

（1）拷贝构造函数的作用就是定义了当我们用同类型的另外一个对象初始化本对象的时候做了什么，在某些情况下，如果我们不自己定义拷贝构造函数，使用默认的拷贝构造函数，就会出错。比如一个类里面有一个指针，如果使用默认的拷贝构造函数，会将指针拷贝过去，即两个指针指向同个对象，那么其中一个类对象析构之后，这个指针也会被delete掉，那么另一个类里面的指针就会变成**野指针（悬浮指针）**；

（2）这也正是深拷贝和浅拷贝的区别，浅拷贝只是简单直接地复制指向某个对象的指针，而不复制对象本身，新旧对象还是共享同一块内存。 但深拷贝会另外创造一个一模一样的对象，新对象跟原对象不共享内存，修改新对象不会改到原对象。

## C++野指针

指针指向了一块非法内存区域(悬空或者说未知区域)。

## C++ 虚函数

### [哪些函数不能是虚函数](https://www.cnblogs.com/NeilZhang/p/5427872.html)

常见的不不能声明为虚函数的有：普通函数（非成员函数）；静态成员函数；内联成员函数；构造函数；友元函数。

## C++ 内存模型

### C++内存布局

#### Heap 堆
由new分配的内存块，其释放编译器不去管，由我们**程序自己控制（一个new对应一个delete）**。如果程序员没有释放掉，在程序结束时OS会自动回收。涉及的问题：“缓冲区溢出”、“内存泄露”

#### Stack 栈

**编译器**在需要时**分配**，在不需要时自动清除。存放**局部变量**和**函数参数**。  
存放在栈中的数据只在当前函数及下一层函数中有效，一旦函数返回了，这些数据也就自动释放了。

#### 全局/静态存储区(.bss和.data段)

全局和静态变量被分配到同一块内存中。在C语言中，未初始化的放在.bss段中称为zero initialization，初始化的放在.data段中，称为const initialization；在C++中二者不进行区分。  
虚函数表就存在这里，因为是全局公用的一张表，通过虚函数指针查找。

#### 常量存储区(.rodata段)

存放常量，不允许修改，比如`const int a = 3`

#### 代码区(.text段)

存放代码，比如我们的函数和类的成员函数。不许修改但是可以执行。

### C++内存区域

根据生命周期不同，C++中可划分出三种不同的内存区域

1.自由存储区、动态区、静态区局部非静态变量的存储区域(栈)  
2.动态区：new，malloc分配的内存  
3.静态区：全局变量，静态变量，字符串常量存在位置

https://www.cnblogs.com/yunlambert/p/9876491.html  
[深入理解计算机系统（内存管理）----内存模型](https://blog.csdn.net/JUST__Tw/article/details/118551674)  
空类(空 class)大小是1，为了标识对象  
[不同类型占用的字节](https://blog.csdn.net/li975242487/article/details/121395693)

## C++运算符重载

https://www.cnblogs.com/liuchenxu123/p/12538623.html

## 派生类的构造函数顺序

https://www.nowcoder.com/questionTerminal/6348a321452a4318a2da5f3757baf620?source=relative

## 迭代器

https://blog.csdn.net/QIANGWEIYUAN/article/details/89184546

## 红黑树

https://blog.csdn.net/u014454538/article/details/120120216

## 小根堆，大根堆

### 完全二叉堆

堆又可称之为完全二叉堆。这是一个逻辑上基于完全二叉树、物理上一般基于线性数据结构（如数组、向量、链表等）的一种数据结构。

#### 完全二叉树

完全二叉树是由满二叉树而引出来的，若设二叉树的`深度为h`，`除第 h 层外`，`其它各层 (1～h-1) 的结点数都达到最大个数(即1~h-1层为一个满二叉树)`，第 h 层所有的结点都连续集中在最左边，这就是完全二叉树。

### 小根堆

根小于节点

### 大根堆

根大于节点



## 对象池

对象池内通常是对象实例的指针，因为对象池的原理是将对象放入池管理的某种内存连续的数据结构中，当不需要对象时，并不销毁对象，而是将对象回收到池中，下次需要的时候再次从池中拿出来。由于对象储存在内存连续的数据结构中，所以能够有效地解决内存碎片的问题。因此，对象池中保存对象的指针比直接保存对象实例更为高效，能够避免频繁分配和销毁内存，从而提高了程序的性能。



## 多线程

[进程间通信方式；线程间通信方式](https://zhuanlan.zhihu.com/p/430069448)

### 进程间通讯的方式？

管道通信，消息队列，共享内存，socket，串口都可以实现。

#### 管道( pipe )：

管道是一种半双工的通信方式，数据只能单向流动，而且只能在具有亲缘关系的进程间使用。进程的亲缘关系通常是指父子进程关系。

#### 有名管道 (namedpipe) ：

有名管道也是半双工的通信方式，但是它允许无亲缘关系进程间的通信。

#### 信号量(semophore ) ：

信号量是一个计数器，可以用来控制多个进程对共享资源的访问。它常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。

#### 消息队列( messagequeue ) ：

消息队列是由消息的链表，存放在内核中并由消息队列标识符标识。消息队列克服了信号传递信息少、管道只能承载无格式字节流以及缓冲区大小受限等缺点。

#### 信号 (sinal ) ：

信号是一种比较复杂的通信方式，用于通知接收进程某个事件已经发生。

#### 共享内存(shared memory ) ：

共享内存就是映射一段能被其他进程所访问的内存，这段共享内存由一个进程创建，但多个进程都可以访问。共享内存是最快的 IPC 方式，它是针对其他进程间通信方式运行效率低而专门设计的。它往往与其他通信机制，如信号两，配合使用，来实现进程间的同步和通信。

#### 套接字(socket ) ：

套接口也是一种进程间通信机制，与其他通信机制不同的是，它可用于不同设备及其间的进程通信。

### 线程间通讯方式。

**锁机制：包括互斥锁、条件变量、读写锁**

- 互斥锁提供了以排他方式防止数据结构被并发修改的方法。  
- 读写锁允许多个线程同时读共享数据，而对写操作是互斥的。  
- 条件变量可以以原子的方式阻塞进程，直到某个特定条件为真为止。对条件的测试是在互斥锁的保护下进行的。条件变量始终与互斥锁一起使用。
  

**信号量机制(Semaphore)：包括无名线程信号量和命名线程信号量**

**信号机制(Signal)：类似进程间的信号处理**

# 图形学部分

## 叉乘

两个向量的叉乘结果是一个垂直于这两个向量的向量，最后这个向量的长度等于以最初两个向量作为边的平行四边形的面积。

## 光照模型

* emissive  
* specular  
* diffuse  
* ambient
  
### 环境光

通常用系统定值，场景物体通用

<img src="/imgs/SpringBoss/C_ambient.png">

### 自发光

材质决定

<img src="/imgs/SpringBoss/C_emissive.png">

### 漫反射

根据 Lambert‘s law

<img src="/imgs/SpringBoss/C_diffuse.png">

### 高光反射

我们有很多变量：normal，eyeDir，lightDir，refDir

#### 已知 normal，lightDir 求 refDir

<img src="/imgs/SpringBoss/NormalLightEyeRef.png">

<img src="/imgs/SpringBoss/RefEqu.png">

##### Phong 模型

<img src="/imgs/SpringBoss/PhongSpecular.png">

##### Blinn-Phong

<img src="/imgs/SpringBoss/BlinnPhong.png">

## 复习下线性代数

### 坐标系

坐标系分为*左手坐标系*和*右手坐标系*

<img src="/imgs/SpringBoss/LeftRight.png" style="zoom:60%;" >

二者在 z 轴方向上有所不同。  
在 Unity 中**模型空间**和**世界空间**为左手系，**观察空间**为右手系。

### 矩阵

矩阵又分为行矩阵和列矩阵。  
***<u>矩阵的左乘/右乘，表示该矩阵在乘号的左侧/右侧。</u>***

#### 行矩阵

<img src="/imgs/SpringBoss/rowMatrix.png" >

表示 1xn 的矩阵，行矩阵**左乘**，变换矩阵乘在右侧-> `1xn dot nxn = 1xn`

##### 齐次式

* 矢量/点
  
  <img src="/imgs/SpringBoss/rowMatrixHC.png">
  
* 变换矩阵
  
  <img src="/imgs/SpringBoss/rowMatrixHCT.png" >
#### 列矩阵

<u>***Unity就是列矩阵！***</u>

> 问题1  
> shader中会看到法线的变换矩阵：  
>float3x3 transform = float3x3(v.tangent.xyz, binormal, v.normal);  
即，**float3x3类型的变量**在创建时，如果传入3个float3类型的变量。整个矩阵是按行优先填充还是按列优先填充呢？  
答：填入的是三个列，即unity shader是**列优先**（column-major）的。  
问题2  
**matrix**.m12是什么意思？  
答：m12是第一行第二列，但是构造矩阵时是**按列的**。

<img src="/imgs/SpringBoss/columnMatrix.png" >

表示 nx1 的矩阵，列矩阵**右乘**，变换矩阵乘在其左侧 `nxn dot nx1 = nx1`

##### 齐次式

* 矢量/点
  
  <img src="/imgs/SpringBoss/colMHC.png">
  
* 变换矩阵
  

<img src="/imgs/SpringBoss/colMHCT.png">

## 矩阵的几何意义变换

变换优先级为：缩放>旋转>位移

### 缩放

<img src="/imgs/SpringBoss/scale.png">

### 旋转

在**左手系**中，绕轴**旋转遵从左手手旋转准则**：大拇指为轴向，四指为旋转方向。  
在**右手系**中，绕轴**旋转遵从右手旋转准则**：大拇指为轴向，四指为旋转方向。

* 绕 x 轴
  

<img src="/imgs/SpringBoss/rotationX.png">

* 绕 y 轴
  

<img src="/imgs/SpringBoss/rotationY.png">

* 绕 z 轴
  

<img src="/imgs/SpringBoss/rotationZ.png">

#### 欧拉角旋转顺序

因为不同轴旋转相乘得到变换矩阵不同。

* 外旋和内旋  
  假定一组旋转顺序 x->y->z 为内旋，则外旋为 z->x->y。  
  一组对应的外旋内旋结果是一样的。
  

**Unity旋转顺序是ZXY**。

### 位移

<img src="/imgs/SpringBoss/translate.png">

### 变换顺序

<img src="/imgs/SpringBoss/transform.png">

## [给定 4x4 矩阵，求缩放，旋转、平移矩阵](https://blog.csdn.net/qq_39300235/article/details/105790743)

就以列矩阵为例解吧，因为大多都是列矩阵运算。  
**M=TRS**

<img src="/imgs/SpringBoss/4x4Matrix.png">

### T矩阵

列矩阵最右侧就是 translate 变换

```c++  
T[0][3]=M[0][3]/M[3][3];  
T[1][3]=M[1][3]/M[3][3];  
T[2][3]=M[1][3]/M[3][3];  
```

### R矩阵

之后我们对 M 矩阵剔除平移变换，得到只有缩放和旋转的矩阵*M*。  
此时 *M*=RS；  
我们只要算出旋转矩阵就行了，因为 R 可以用**旋转矩阵的逆矩阵**计算得到：

<img src="/imgs/SpringBoss/4x4Rn.png">

接下来要分解出 R 矩阵，我们利用 ”Polar decomposition(极分解)“，可以证明矩阵*M*的极分解为 <u>旋转R</u> 和 <u>缩放S</u> 。该方法通过对 *M* 的逆的转置进行连续平均来计算，直到**收敛**，此时 ***M**i **= R***。

<img src="/imgs/SpringBoss/Polar.png">

当矩阵只有旋转变换时，旋转R 的转置和R的逆相等，达到收敛状态。  
因为旋转矩阵为**正交矩阵**。Shoemake和Duff（1992）讨论了该级数收敛的证明，所得矩阵是最接近M的正交矩阵，这是理想的特性。为了计算该序列，我们迭代应用公式，直到连续项之间的差很小或执行了固定的迭代次数为止。 实际上，该系列通常会很快收敛。

```c++  
Float norm;  
int count = 0;  
Matrix4x4 R = M;  
do  
{  
    Matrix4x4 Rnext;  
    Matrix4x4 Rit = Inverse(Transpose(R));    // 转置的逆  
    for(int i = 0; i < 4; ++i)  
    {  
        for(int j = 0; j < 4; ++j)  
        {  
            Rnext.m[i][j] = 0.5f * (R.m[i][j]+Rit.m[i][j]);  
        }  
    }  
    // compute the sub between Mi and Mi+1  
    norm = 0;  
    for(int i = 0; i < 3; ++i)  
    {  
        Float n = abs(R.m[i][0] - Rnext.m[i][0]) +  
            abs(R.m[i][1] - Rnext.m[i][1]) +  
            abs(R.m[i][2] - Rnext.m[i][2]);  
        norm = max(norm,n);  
    }  
    R = Rnext;  
}while(++count < 100 && norm > 0.0001) // 当迭代次数过多，或者连续项差距足够小，退出循环。  
```

### S矩阵

得到 R 矩阵后 S矩阵就很简单

```c++  
S = Matrix4x4::Mul(Inverse(R),M);  
```

### 另外一种办法

我们可以对3x3矩阵的每个列向量求长度，这样就知道每个坐标轴分量上的缩放，最终得到缩放矩阵，然后再根据缩放矩阵求旋转矩阵即可。

## MVP矩阵

### Model

把顶点坐标从局部空间变换到全局空间。

* 行矩阵
  <img src="/imgs/SpringBoss/ModelRow.png">
  
* 列矩阵
<img src="/imgs/SpringBoss/ModelCol.png">

### View-Matrix 推导

在图形渲染中我们要把3D的场景渲染成一张2D的图片，而这张图片是从 Camera 的视角出发，所以为了方便渲染我们做一个变换，把摄影机作为一个新的空间的原点，令摄影机观测方向为-z方向也就是我们常说的 视图变换(Viewing Transform)。  
我们对场景中的物体都进行这样一个变换就可以得到，观察空间下的物体的坐标信息，即以 Camera 为原点，Camera 上方向为 y 轴，Camera 视线为 -z 轴，Camera 右侧为 x轴的坐标空间下物体信息。  
**注意这里观察空间通常为右手系！而非和世界空间/模型空间所采用的左手系！**  
**接下来推导为列矩阵运算！**
这个变换主要分为两步一步是平移，一步是旋转。  
设 Camera 坐标为<img src="/imgs/SpringBoss/posE.png" style="zoom:50%;" >  
则平移矩阵为  
<img src="/imgs/SpringBoss/viewT.png" style="zoom:50%">
现在我们把摄影机放到了原点位置，现在我们只需要把轴向调整即可。  
*注意这里变换推导过程是先平移后旋转，和平时计算规定的 Transform 先 scale 再 rotation最后  translate 的顺序不同。*  
我们假设 Camera up 方向矢量为 *t*，观察方向为 *g*，则其 x 轴 *e* 为 *g*x*t*。  
<img src="/imgs/SpringBoss/cameraETG.png" style="zoom:50%">  
我们要让其轴向与 xyz 轴对齐，因为现在只是把物体进行了移动，坐标信息的基础轴还是世界坐标系而非 camera坐标系的三轴。  
我们只需把 `t->y`,`g-> -z`,`e->x` 其中 `e = gxt`。  
这样虽然可以进行计算，但是十分复杂。  
我们观察到 既然TEG是坐标轴，就意味着他们三者垂直，点积为零，可以构成正交矩阵。  
那么 `t->y`,`g-> -z`,`e->x` 的逆变换 `y->t`,`-z ->g`,`x->e` 是很好计算的。  
而我们知道了逆矩阵，就可以知道正交矩阵的原矩阵。  
正交矩阵性质: 
<img src="/imgs/SpringBoss/orthM.png" style="zoom:80%">

所以
<img src="/imgs/SpringBoss/Rview.png" style="zoom:80%">

最后
<img src="/imgs/SpringBoss/Mview.png" style="zoom:80%">

### Projection Matrix

我们现在已经放好了物体，也以 Camera 的角度出发了，现在我们要从 **View space( 观察空间 )** 到 **Clip Space( 裁剪空间/齐次裁剪空间 )**

#### 我们这里先考虑把深度归到[0,1]的情况

要把 **View 空间**的点投影到**屏幕**上，利用相似三角形原理，
$$
\begin{aligned}
\frac{x_{screen}}{near}&=\frac{x_{view}}{z_{view}}\\
x_{screen}&=\frac{x_{view}\cdot near}{z_{view}}\\
y_{screen}&=\frac{y_{view}\cdot near}{z_{view}}\\
\end{aligned}
$$
但是实际上我们是先把 *X,Y* 坐标变换到[-1,1]后映射到 Screen的，所以还得改成  
将所有顶点映射到[-1,1]是为了方便 GPU 计算
$$
\begin{aligned}
&x_{screen}=\frac{x_{view}\cdot near}{z_{view}}&\in[-wid,wid]\\
&y_{screen}=\frac{y_{view}\cdot near}{z_{view}}&\in[-height,height]\\
&x_{ndc}=\frac{x_{view}\cdot near}{z_{view}\cdot wid}&\in [-1,1]\\
&y_{ndc}=\frac{y_{view}\cdot near}{z_{view}\cdot height}&\in [-1,1]\\
\end{aligned}
$$
所以我们现在知道如何对 x,y 坐标进行变换了，Project Matrix 暂时可以写成这样，
因为 z,w 不受 x,y 影响
$$
\begin{bmatrix}
\frac{near}{z_p\cdot wid}&0&0&0\\
0&\frac{near}{z_p\cdot height}&0&0\\
0&0&A&B\\
0&0&C&D\\
\end{bmatrix}
\begin{bmatrix}
x_p\\
y_p\\
z_p\\
1\\
\end{bmatrix}
$$

而 z 值有些不一样，现在的问题是  

* “我们的矩阵 *X,Y 坐标* 的变换矩阵和 <u>z值相关</u>”，  
  这肯定不是我们希望看到的，这不利于 GPU 计算，
* 而且为了实现深度测试我们还希望可以将深度值归化到[0,1]，  

对于**第一个问题**，既然 *X,Y 坐标* 都要除 View Space 的 *Z* 坐标值，  
那么我们不妨把这个除法操作在后面统一执行，也就是 **齐次除法**，  
所以我们需要在做这个除法操作时能得到 *Z* 坐标值，而恰巧我们可以利用 *W 分量*  
我们可以先把 *Z* 坐标轴存在 W 分量，即
$$
\begin{bmatrix}
\frac{near}{wid}&0&0&0\\
0&\frac{near}{height}&0&0\\
0&0&A&B\\
0&0&1&0\\
\end{bmatrix}
$$

<u>因为View Space 是右手系的所以……z值不是正的</u>

**第二个问题**，而对于 *Z* 值 原本范围是 [-far, -near] -> [?, ?] -> [0,1]
$$
[-far,-near]\rightarrow[?,?]\stackrel{/z}{\rightarrow}[0,1]
$$
所以我们对 -near 和 -far 两个边界条件 列方程，因为是[0,1]在加上齐次除法所以near为0，far为far
$$
\begin{aligned}
	-A\cdot near+B=0\\
	-A\cdot far+B=far\\
	A=\frac{far}{near-far}\\
	B=\frac{near\cdot far}{near-far}
\end{aligned}
$$

得到
$$
\begin{bmatrix}
\frac{near}{wid}&0&0&0\\
0&\frac{near}{height}&0&0\\
0&0&\frac{far}{near-far}&\frac{near\cdot far}{near-far}\\
0&0&1&0\\
\end{bmatrix}
$$
从裁剪空间到NDC需要一次齐次除法

#### think 但可能不对

齐次变换(仿射变换)的 W分量 对于一个点来说，可以表示变换的“距离", 比如用矩阵平移 点(x,y,z) 距离(1,1,1)   
如果把W=2，再W 归一得到的是(x/2+0.5, y/2+0.5, z/2+0.5),  
如果有若干多个点，这样做，相当于一种投影，而投影点集前后，点集之间相对当前 W 分量的点集关系不会改变，比如两个点之前相聚(a,b,c)， 除2后就是 (a/2,b/2,c/2)  
挺有意思但没啥用好像。

#### 从屏幕空间和Depth 反算world Pos

$$
\begin{aligned}
&M_{proj}M_{view}\cdot P_{ws}=P_{cs}\\
&\frac{P_{cs}}{P_{cs}.w}=P_{ndc}\\
&With\;P_{ndc}\;need\;P_{ws}\;but\;no\;P_{cs}.w\\
&There\;is\\
&P_{ws}=M_{view}^{-1}\Big(M_{proj}^{-1}(P_{ndc}*P_{cs}.w)\Big)\\
&P_{ws}.w=\Big(M_{view}^{-1}M_{proj}^{-1}(P_{ndc}*P_{cs}.w)\Big).w=1\\
&so\\
&P_{cs}.w=\frac{1.0}{M_{view}^{-1}M_{proj}^{-1}P_{ndc}.w}\\
&P_{ws}=\frac{M_{view}^{-1}M_{proj}^{-1}P_{ndc}}
{(M_{view}^{-1}M_{proj}^{-1}P_{ndc}).w}
\end{aligned}
$$



## TBN 矩阵

[TBN矩阵不错的Blog](https://zhuanlan.zhihu.com/p/412555049)

## 2D平面三角形，给出算法，生成三角形内随机一点

https://www.cnblogs.com/TenosDoIt/p/4025221.html

## 前向渲染和延迟渲染

https://zhuanlan.zhihu.com/p/28489928

## LUT 表

https://www.jianshu.com/p/fdec2a5e889f

## 描边效果

边缘检测

## 为什么要有光线追踪

1. 传统光栅化做阴影效果不好，操作困难  
2. 物体的模糊反射，比如毛玻璃一般的反射即 Glossy Reflection，光线打到Glossy物体在反射  
3. 间接光照(Indirect illumination)，光线在进入人眼前弹射不止一次

这种光线弹射，对于光栅化来说想要实现比较麻烦，而且也不能保证物理上的正确性。光栅化本质上是一种快速的近似质量较低。  
光线追踪是一种比较准确的办法质量很高，但是最大的问题就是很慢。

## 包围盒BVH

### 如何划分BVH

1.随机选取一个维度

2.选Bounding box中最长的轴进行进一步划分 

3.选取中间的三角形处进行划分（以保证两边三角形数量接近）。

5.基于表面积的启发式评估划分方法（Surface Area Heuristic，SAH），这种方法通过对求交代价和遍历代价进行评估，给出了每一种划分的代价（Cost），寻找代价最小方式进行划分。

6.基于莫顿码（Morton code）的并行化BVH构建。

#### 参考学习！记得学啊！

[PBRT-E4.3-层次包围体(BVH)（一） - 玉米的文章 - 知乎](https://zhuanlan.zhihu.com/p/50720158)  
[PBRT-E4.3-层次包围体(BVH)（二） - 玉米的文章 - 知乎](https://zhuanlan.zhihu.com/p/54620381)  
[PBRT-E4.3-层次包围体(BVH)（三） - 玉米的文章 - 知乎](https://zhuanlan.zhihu.com/p/54694041)  
[【空间加速结构】——层次包围体BVH（Bounding Volume Hierachies） - silence394 - 博客园](https://www.cnblogs.com/silence394/p/17285231.html)  
[Dynamic AABB Trees](https://box2d.org/files/ErinCatto_DynamicBVH_GDC2019.pdf)

## 渲染管线

一般就说说大概的流程 IA->VS->Hull->TS->Domain->GS->OS->RS->PS->OM

### 什么是曲面细分着色器

[大体介绍了一下曲面细分](https://www.cnblogs.com/chenglixue/p/17227713.html)

## 透明效果



# 球谐函数



## 雾效





# 重要性采样



## 阴影

LightMap

ShadowMapping

PCF

PCSS

VSSM

### CSM如何做的？

Cascaded Shadow Maps(CSM)方法根据对象到观察者的距离提供不同分辨率的深度纹理来解决上述问题。它将相机的视锥体分割成若干部分，然后为分割的每一部分生成独立的深度贴图。对于近处的场景使用较高分辨率的阴影贴图，对于远处的场景使用粗糙的阴影贴图，在两张阴影贴图过渡的地方选择其中一张使用。

## 两个多边形如何判断相交

https://blog.csdn.net/StevenKyleLee/article/details/88075814

### 一点是否在多边形内

https://blog.csdn.net/StevenKyleLee/article/details/88044589

一般用射线法这里说一下特殊情况：  
<img src="/imgs/SpringBoss/RayPointSpe.png" style="zoom:100%;" >  
这里的(b)因为中间那个顶点左右两条边都在该顶点的上侧，所以该顶点不识为于射线相交。  
具体的判定过程应该可以根据多边形绕向找相邻点进行判断。

[除射线法外的其他方法](https://blog.csdn.net/WilliamSun0122/article/details/77994526)

## TAA(temporal anti-aliasing)

[TAA in DX12](https://zhuanlan.zhihu.com/p/64993622)  
[快速理解Tone Mapping](https://zhuanlan.zhihu.com/p/147567747)  
[Tone Mapping](http://www.ownself.org/blog/2011/tone-mapping.html)  
[TAA GHOSTING 的相关问题](https://www.cnblogs.com/crazii/p/7244300.html)  
Tone Mapping 是HDR算法的一部分，是用来将渲染出的场景亮度域映射到一个合理的亮度域空间。

> 然后值得一提的是TAA在管线中的位置，虚幻的TAA是放在其它的后处理之前的，这么可以防止其它后处理出现的闪烁，但是因为高光很容易闪，我们又希望能在低动态范围处理，所以这里虚幻选择的是先tonemap，再算超采样，最后逆tonemap输出，去做其他的后处理。

这里是说如果不做ToonMapping把HDR压倒LDR做TAA，因为HDR范围比LDR，然后TAA在混合在一起就会“亮的很花”，所以先Toon Mapping到LDR做一下TAA在 逆Toonmap输出，这样还得到了抗锯齿的输出，后处理做的更好了。这里的HDR和Toonmap我思考了很久，因为我没明白Toonmap，toonmap就是把hdr变成ldr，hdr虽然表示了更多的颜色，但是我们在屏幕空间显示还是0~1，所以要做映射，直接线性映射不行，因为人对暗部感知明显，而且自然界比LDR亮度大得多，所以我们做了很多调整。

### 颜色LDR和HDR

[LUT简述](https://www.jianshu.com/p/fdec2a5e889f)  
[HDR和Tone Mapping](https://zhuanlan.zhihu.com/p/80253409)

### 几何走样和着色走样

* **几何走样：**几何覆盖函数采样不足，即俗称的边缘锯齿，一般发生在**光栅化阶段**。  
* **着色走样：**渲染方程的采样不足，因为*渲染方程也是连续函数*，对某些部分在*空间变化较快（高频部分）采样不足也会造成走样*，反映在视觉上一般是图像闪烁或噪点，这类走样称之为着色走样，一般发生在**各种着色阶段**。
  
## Z-fighting



## 景深

所谓景深就是照片背景的虚化程度，规律是：光圈越大，景深越浅，背景越模糊；光圈越小，景深越深，背景越清晰。  
[景深](http://www.ownself.org/blog/2010/jing-shen-depth-of-field.html)

## 点到三角形距离

https://zhuanlan.zhihu.com/p/148511581  
[点到三角形面距离](https://zhuanlan.zhihu.com/p/148511581)

## PBR

[【基于物理的渲染（PBR）白皮书】（一） 开篇：PBR核心知识体系总结与概览](https://blog.csdn.net/poem_qianmo/article/details/85239398)

## 材质系统

材质系统了解、实现。



# 计组部分

### 大端和小端模式

https://blog.csdn.net/wei_cheng18/article/details/79856207

* **大端**（Big_endian）字数据的**高字节**存储在**低地址**中，字数据的**低字节**存储在**高地址**中。  
* **小端**（Little_endian）字数据的**高字节**存储在**高地址**中，字数据的**低字节**存储在**低地址**中。
  
### Cache 命中率

# 计网

## TCP

### [TCP三次握手](https://blog.csdn.net/jun2016425/article/details/81506353)

* 第一次握手：客户端向服务器发起链接请求  
* 第二次握手：服务器向客户端返回，发送答应客户端请求的确认信息。  
* 第三次握手：客户端向服务器，告诉服务器已经收到第二次握手的确认信息。
  
#### TCP的三次握手过程？为什么会采用三次握手，若采用二次握手可以吗？

答：建立连接的过程是利用客户服务器模式，假设主机A为客户端，主机B为服务器端。

（1）TCP的三次握手过程：主机A向B发送连接请求；主机B对收到的主机A的报文段进行确认；主机A再次对主机B的确认进行确认。

（2）采用三次握手是为了防止失效的连接请求报文段突然又传送到主机B，因而产生错误。失效的连接请求报文段是指：主机A发出的连接请求没有收到主机B的确认，于是经过一段时间后，主机A又重新向主机B发送连接请求，且建立成功，顺序完成数据传输。考虑这样一种特殊情况，主机A第一次发送的连接请求并没有丢失，而是因为网络节点导致延迟达到主机B，主机B以为是主机A又发起的新连接，于是主机B同意连接，并向主机A发回确认，但是此时主机A根本不会理会，主机B就一直在等待主机A发送数据，导致主机B的资源浪费。

（3）采用两次握手不行，原因就是上面说的失效的连接请求的特殊情况，因此采用三次握手刚刚好，两次可能出现失效，四次甚至更多次则没必要，反而复杂了

### 四次挥手

* 第一次挥手：主动关闭方发送一个FIN，用来关闭主动方到被动关闭方的数据传送，**主动方：“我不会再给你数据了。”**  
* 第二次挥手：被动关闭方收到FIN包后，发送一个ACK给对方，确认序号为收到序号+1，**被动方：“我知道了！”**  
* 第三次挥手：被动关闭方发送一个FIN，用来关闭被动关闭方到主动关闭方的数据传送，**被动方：”我的数据也发送完了，不会再给你发数据了。“**  
* 第四次挥手：主动关闭方收到FIN后，发送一个ACK给被动关闭方，确认序号为收到序号+1，**主动方：“行了，那我知道了，一切结束”**
  
# 编译原理

## [逆波兰式](https://blog.csdn.net/weixin_43919932/article/details/103327530)

https://www.cnblogs.com/tangqs/archive/2012/05/18/2507708.html  
 如果是右单目运算符，直接入存储器栈；比如 阶乘！与百分号%

# OS

## 进程和线程

[进程和线程](https://blog.csdn.net/ThinkWon/article/details/102021274)  
[线程调度](https://blog.csdn.net/qq_33182418/article/details/121135914)  
[进程/作业调度](https://blog.csdn.net/qq_41784433/article/details/122194695)

### 多线程比单线程慢

存在上下文切换和死锁问题。  
https://www.cnblogs.com/xrq730/p/5186609.html

## Linux 进程通讯

https://blog.csdn.net/qq_44443986/article/details/115065540

## 死锁和银行家算法

https://blog.csdn.net/wyf2017/article/details/80068608

## 什么是内存碎片，内存碎片是在虚拟内存还是物理内存？

采用分区式存储管理的系统，在储存分配过程中产生的、不能供用户作业使用的主存里的小分区称成“内存碎片”。内存碎片分为内部碎片和外部碎片。内存碎片只存在于虚拟内存上。

# 概率论

## 期望计算和带保底卡池期望计算











# 算法

## LRU手撸

https://leetcode.cn/problems/lru-cache-lcci/

LRU算法还有进化版LFU算法

## 双端队列

一个能用队列和栈实现的增删查改都是O(1)的数据结构，是什么  
这个能用队列和栈实现的增删查改都是O(1)的数据结构是双端队列（deque）[[1](https://zh.wikipedia.org/wiki/队列#双端队列)]。双端队列不仅支持在队列的一端进行入队和出队操作，也支持在队列的另一端进行插入和删除操作。因此，在使用双端队列时，可以根据具体的需求选择在队列的哪一端进行操作，从而实现了增删查改都是O(1)的效率。

### 双端队列定义

能在队列两端进行入队、出队操作。

## 统计子树中城市之间最大距离

https://leetcode.cn/problems/count-subtrees-with-max-distance-between-cities/submissions/412418189/

## A*算法

https://zhuanlan.zhihu.com/p/54510444

## 岛屿问题

https://leetcode.cn/problems/number-of-islands/solutions/13103/dao-yu-shu-liang-by-leetcode/

## Top K 问题

https://zhuanlan.zhihu.com/p/64627590



# C# 和 Unity

[CSDN不错的面试总结-2022年Unity面试题分享 | 全面总结 | 建议收藏](https://blog.csdn.net/qq_21407523/article/details/108814300)  
[知乎上五尘的Unity面试题](https://zhuanlan.zhihu.com/p/554529423)

## 协程

[总结：协程与线程](https://blog.csdn.net/w2009211777/article/details/125514898)

## 请简述GC（垃圾回收）产生的原因，并描述如何避免？

GC回收堆上的内存避免：1.减少new产生对象的次数2.使用公用的对象（静态成员）3.将String换为StringBuilder

## C++指针和C#的引用

[c++引用和c#引用类型的区别](https://zhuanlan.zhihu.com/p/389422617)

## 反射

[[整理]C#反射(Reflection)详解](https://www.cnblogs.com/wangshenhe/p/3256657.html)  
[【Unity|C#】基础篇(12)——反射（Reflection）（核心类：Type、Assembly）](https://www.cnblogs.com/shahdza/p/12261831.html)

* 反射的定义：审查元数据并收集关于它的类型信息的能力。元数据（编译以后的最基本数据单元）就是一大堆的表，当编译程序集或者模块时，编译器会创建一个类定义表，一个字段定义表，和一个方法定义表等。
  
* .NET的应用程序由几个部分：**程序集（Assembly）、模块（Module）、类型（class）组成**。
  
## 工厂模式

https://wittykyrie.github.io/posts/Factor-Pattern/  
[简单工厂模式、工厂方法模式优缺点](https://blog.csdn.net/cxy_zxl/article/details/116695023)  
[C++ 深入浅出工厂模式（初识篇）](https://zhuanlan.zhihu.com/p/83535678)

### 抽象工厂模式

https://blog.csdn.net/m0_46502538/article/details/120343296

## UGUI 如何实现UI物体淡入淡出?

* Text，Image这些组件都有继承Graphic类，这个类提供了CrossFadeAlpha()方法，可以做透明度渐变。  
* 但如果界面东西多了，要获取每个Graphic是挺麻烦了。其实还有一个很简便的方法，用起来跟NGUI差不多。就是CanvasGruop组件，把这个组件放到界面根节点上，对这个组件的alpha做改变就行了。看看官方文档的解释，
  
## ESC 框架

https://blog.codingnow.com/2017/06/overwatch_ecs.html

ECS 的 E ，也就是 Entity ，可以说就是传统引擎中的 Game Object 。但是是 C 的整合其实。

C 和 S 是这个框架的核心。System 系统，也就是我上面提到的模块。每个模块应该专注于干好一件事，而每件事要么是作用于游戏世界里同类的一组对象的每单个个体的，要么是关心这类对象的某种特定的交互行为。比如碰撞系统，就只关心对象的体积和位置，不关心对象的名字，连接状态，音效、敌对关系等。

每个可能单独使用的对象属性归纳为一个个 Component ，比如对象的名字就是一个 Component ，对象的位置状态是另一个 Component 。

## unity的委托是什么? event 关键字有什么用？

委托是一个容器，可以放函数对象，并且可以触发委托面的每个函数调用。委托主要用户回调函数。如果在外部给委托变量加函数进来，那么委托要定义成public, 这样做又有一个问题，public外部的人也可以触发这个委托，如果我希望设计成外部可以加回调，但是只能是模块内部触发委托，那么我可以加一个event来修饰，这样虽然是public,但是外部无法触发委托,只能类的内部触发。

# DX12

DX12和

# Vulkan

[vulkan教程](https://geek-docs.com/vulkan/vulkan-tutorial/vulkan-tutorial-index.html)

## Vulkan和DX12相比OpenGL好在哪里？



# Unity

[unity 八股文 实时更新](https://zhuanlan.zhihu.com/p/585164814)

## UnityUpdate

<img src="/imgs/SpringBoss/unityUpdate.jpg">

## Unity UGUI不规则区域点击问题

https://www.cnblogs.com/msxh/p/9283266.html

## 当一个细小的高速物体撞向另一个较大的物体时，会出现什么情况？如何避免？

碰撞检测失败，会直接穿透

避免方法：  
（1）增大细小物体的碰撞体（不建议这样做）  
（2）使用射线检测，检测他们之间的距离  
（3）FixedUpdate频率修改，可以physics time减小（同样不建议）  
（4）改变物体的速度（废话）  
（5）将检测方式改为连续检测，rigifdbody.collisionDetectionMode =CollisionDetectionMode.Continuous;  
或者是动态连续检测（CollisionDetectionMode.ContinuousDynamic）  
（6）代码限制，加大计算量 提前计算好下一个位置

## [Canvas 有几种模式，如何配置？](https://blog.csdn.net/weixin_42352178/article/details/109034679)

Canvas的三种渲染模式：

* Screen Space-Overlay（屏幕空间-覆盖模式）UI元素的位置坐标是屏幕空间的坐标，Overlay模式下画布会填满整个屏幕空间，并将画布下面的所有的UI元素置于屏幕的最上层。如果屏幕尺寸被改变，画布将自动改变尺寸来匹配屏幕。  
* Screen Space-Camera（屏幕空间-摄影机模式）UI元素的位置坐标是屏幕空间的坐标，画布也是填满整个屏幕空间，如果屏幕尺寸改变，画布也会自动改变尺寸来匹配屏幕。所不同的是，在该模式下，画布会被放置到摄影机前方。在这种渲染模式下，画布看起来绘制在一个与摄影机固定距离的平面上。所有的UI元素都由该摄影机渲染，因此摄影机的设置会影响到UI画面。  
  此时，画布上的UI组件会随视角移动。  
* World Space即世界空间模式，此模式下UI元素的位置坐标是世界空间的坐标。画布作为场景中的一部分被固定显示在场景中，显示效果类似Plane组件。
  
## Unity 渲染队列
| Name        | ID   | Description                                                  |  
| ----------- | ---- | ------------------------------------------------------------ |  
| Background  | 1000 | 会在任何其他队列前渲染，渲染需要绘制在背景上的物体           |  
| Geometry    | 2000 | 默认的渲染队列，大部分物体都使用这个队列，不透明物体一般使用这个队列 |  
| AlphaTest   | 2450 | 需要进行透明测试的物体                                       |  
| Transparent | 3000 | 在 Geometry 和 AlphaTest 后，**从后向前**地渲染，任何使用透明混合的物体都使用此队列 |  
| Overlay     | 4000 | 该队列用于实现叠加效果，任何需要最后渲染的物体都使用此队列。 |



## 游戏动画

### 关节动画

### 单一网格动画

### 骨骼动画

## unity中UGUI如何打包成图集?

[unity中UGUI如何打包成图集? - 鲨鱼辣椒的回答 - 知乎 ](https://www.zhihu.com/question/472146051/answer/2011067756)

* 开启UGUI的图集模式, Editor->Project Settings 下面有sprite packer的模式。  
* 为每个UI图片制定要打入的图集的tag名字。  
* 打包生成图集Window ------>Sprite Packer, 点击Pack即可打包生成图集。
  
## Build-IN RP和SRP

[Unity SRP URP HDRP 的区别](https://blog.csdn.net/weixin_41622043/article/details/107623694)

## SRP

[Unity官方宣传](https://unity.com/cn/srp)

### SRP Batcher

[SPR Batcher 官方简介](https://blog.unity.com/cn/technology/srp-batcher-speed-up-your-rendering)
