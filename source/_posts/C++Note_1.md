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

## 什么是面向对象？

面向对象是相对面对过程而言，通过类和对象的设计，封装、继承、多态的特性进行开发。  
即以对象为核心，通过对象封装**数据**和**行为(函数)**。  
其中核心特性：封装、继承、多态。  

* 封装：   
  将**数据**和**函数**封存在类内，**隐藏内部细节**，只暴露出必要的部分，  
  尽量避免外部的直接修改。  
  通过public 方法尽量做到安全访问。  
  所有类的设计都应该是一个封装的过程。

* 继承:  
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
    }
    
    class B
    {
        public:
        int age;
        B()
        {
            age = 2;
        }
    }
    
    class C : public A, public B
    {
        public :
        int age;
        C()
        {
            age = 3;
        }
    }
    
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
    }
    
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
    }
    
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
    通过在子类中vbptr指向虚基类表，通过读取虚基类表存储的偏移量，计算目标地址。  
    ![](/imgs/C++/虚继承.jpg)