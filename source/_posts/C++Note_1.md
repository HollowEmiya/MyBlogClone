---
title: C++ Note 1
math: true
tags: [C++]
index_img: /imgs/Hexo主题变更/Shiki&Tsukihime.png
banner_img: /imgs/Hexo主题变更/Shiki&Tsukihime.png
date: 2025-08-16 02:03:13
typora-root-url: ../
---
# C++ Note 1

重新学C++了家人们，自从2023年毕业之后就碰了三个月cpp，后面去做技术美术写了一年的Shader和C# URP，Render Pipeline，但是 C++ 是我的心病，就得学 C++，卧槽，拦不住的。

## 什么是面向对象？

面向对象是相对面对过程而言，通过类和对象的设计，封装、继承、多态的特性进行开发。  
即以对象为核心，通过对象封装**数据**和**行为(函数)**。  
其中核心特性：封装、继承、多态。  

### 封装：   
将**数据**和**函数**封存在类内，**隐藏内部细节**，只暴露出必要的部分，  
尽量避免外部的直接修改。  
通过public 方法尽量做到安全访问。  
所有类的设计都应该是一个封装的过程。

### 继承:  
子类能够复用父类方法，实现代码复用。  
继承的情况：  

* 单一继承：子类只有一个父类

* 多个继承：子类多个父类，如果父类有相同的成员，需要在子类中使用类名限定。  
  ~~~c++
  class A
  {
      public:
      int age;
      A()
      {
          age = 1;
      }
  };
  
  class B
  {
      public:
      int age;
      B()
      {
          age = 2;
      }
  };
  
  class C : public A, public B
  {
      public :
      int age;
      C()
      {
          age = 3;
      }
  };
  
  C c = C();
  // c.age 3
  // c.B::age 2
  // c.A::age 1
  ~~~

* 菱形继承：1-n-1 的继承方式，通过使用虚函数进行虚继承来解决  
  ~~~C++
  // ERROR
  class GrandFather
  {
      public:
          int age;
          GrandFather(){ age = 100; }
  };
  
  class Father1 : public GrandFather{};
  class Father2 : public GrandFather{};
  class Son : public Father1, public Father2{}
  
  int main()
  {
      Son p;
      p.age = 12;
      cout << p.age << endl;
      return 0;
  }
  ~~~

  这个例子会报错，因为编译器不知道age是哪个作用域下的age，不知道是Father1还是Father2的，我们可以指定作用域。  
  ~~~C++
  int main()
  {
      Son p;
      p.Father1::age = 12;
      p.Father2::age = 23;
      cout << p.Father1::age << endl;
      cout << p.Father2::age << endl;
      return 0;
  }
  ~~~

  但是这种解决办法缺少实际意义，我们往往需要的是son只有一个age属性即可。
  使用`virtual`  

  ~~~C++
  class GrandFather
  {
      public:
          int age;
          GrandFather(){ age = 100; }
  };
  
  class Father1 : virtual public GrandFather{};
  class Father2 : virtual public GrandFather{};
  class Son : public Father1, public Father2{}
  
  int main()
  {
      Son p;
      p.Father1::age = 12;
      p.Father2::age = 22;
      cout << p.age << endl;                // 22
      cout << p.Father1::age << endl;        // 22
      cout << p.Father2::age << endl;        // 22
      return 0;
  }
  ~~~

  结果会是三个`22`，因为son中只有一个age了。
  [l菱形继承](https://zhuanlan.zhihu.com/p/395173418)  
  通过在子类中vbptr指向**虚基类表**，通过读取**虚基类表存储的偏移量，计算目标地址**。  
  ![](/imgs/C++/虚继承.jpg)  
  虚基类表存储了内存偏移，使得有菱形继承的子类能够将爷爷的成员存储的相同地址。

### 多态

多态的核心思想就是为不同数据类型的实体提供统一的接口，  
使得统一操作在面对不同类型的对象时采取不同的行为。  
比如两个继承同一父类的子类对象，对父类的虚函数的可能有不同的实现方案。  
C++ 中多态分为静态多态和动态多态。

#### 静态多态

通过**函数重载、运算符重载、模板**实现，可以在编译阶段确定函数调用。

#### 动态多态

通过**虚函数、继承、基类指针、引用**实现，在运行过程中动态绑定函数调用

## 虚函数

在基类中能够使用 `virtual` 修饰成员函数，允许子类重写该函数改变函数行为。

在深入虚函数之前需要对 C++ 的 [内存布局](#内存布局) 有一定的了解。

## 内存布局

*代码段、数据段、BSS 段(Block Started by Symbol)、Heap、Stack*

### 代码段

- 存储程序编译后的机器指令（函数体二进制代码）
- 特性：只读，防止指令被意外修改

### 数据段

- 存储**已初始化的全局变量、静态变量、常量字符串**

### BSS 段(Block Started by Symbol)

* **未初始化的全局变量、静态变量（C++ 中默认初始化为 0）**  

* 不占用空间，程序加载到内存后，由操作系统自动初始化为0（清零）

### Heap

- 动态内存分配区域，通过 `new/malloc` 管理

### Stack

- 自动管理临时数据的区域，由编译器控制
- 特性
  - 地址**从高到低增长**
  - 存储**局部变量、函数参数、返回地址、临时对象**
  - 每个函数调用创建独立的**栈帧（Stack Frame）**
  - 栈溢出（Stack Overflow）风险（如递归过深）

### 关于对象的内存布局

一不包含静态函数，只有普通成员函数的类：  
~~~c++
class Base
{
public:
	void fun_a()
    {
	}
    
    void fun_b()
    {
    }
    
    int a;
};
~~~

~~~mermaid
graph LR
A[方形] -->B["fun_a()
fun_b()"]
C[堆] --> D[a:int]
~~~

