---
title: C++11之 lambda 和 function
date: 2023-05-01 14:20:22
tags:
---
  
C++11 的lambda 表达式和 function 是新引入的功能，他们的好处在于可以**简化代码、使之更灵活**而且在并发编程中一个 lambda 表达式或 function 对象作为线程的执行体会更安全，避免为一个线程单独写一个函数。  
<!--more-->
  
# Lambda 和 Function
  
## Lambda 和 Function 的好处
  
1. Lambda表达式能够简化代码，使代码更加简洁。Lambda表达式相当于一个匿名函数，可以在需要的地方直接定义使用，不必再为了实现某个功能而额外定义一个函数。  
2. Lambda表达式和Function都能够实现函数对象的传递，即将函数作为参数传递给其他函数。这种方式可以让代码更加灵活，更加符合面向对象编程的思想。  
3. Function是一个通用的函数封装器，可以将任何可调用的对象（比如函数指针、成员函数指针、Lambda表达式等）转换为一个统一的函数类型。Function可以让代码更加通用，更不易出错。  
4. Lambda表达式和Function都能够简化并发编程的代码。在并发编程中，有时需要定义一个Lambda表达式或Function对象作为一个线程的执行体，这样可以避免为线程另外写一个函数。
  
总结：Lambda表达式和Function的好处包括代码简洁、函数传递灵活、通用性强、并发编程简单等。
  
## 实例代码
  
~~~c++
function<int(int)> dfs=[&](int cur) -> int {
    int res=0;
    for(int nei : g[cur])
    {
        res = max(res,dfs(nei));
    }
    return information[cur]+res;
};
~~~
  
## function
  
function 是 std::function 类型，可以用来存储任意可调用对象，包括函数指针、Lambda 表达式等等。
  
其使用方法为  
`function<int(char)>` , int 为声明的返回值类型，而 char 为声明的参数类型，如果有多个参数即多个声明，如 `function<void(int,char)>`。
  
function 和函数指针有些类似，但是代码更灵活可以和 lambda 表达式相结合使用，而没有指针带来的问题，但是因为使用了模板所以会有一定的实例化开销。  
我们也可以用 auto 去指定 lambda 表达式的匿名函数。
  
~~~c++
auto funcPtr = &functionName;
auto lambda = [](int x, int y){ return x+y;};
int (*function) (int,int) = lambda;	// 将 Lambda 表达式赋值给函数指针
~~~
  
int (*funcPtr)(int, int) 是一个函数指针，它可以指向一个返回类型为 int，参数列表为 (int, int) 的函数。而 lambda 表达式则相当于一个匿名函数，用于表示一个可调用对象，可以被赋值给一个函数指针。
  
## Lambda
  
可以用在 sort 函数中自定义排序
  
~~~c++
sort(nums.begin(),nums.end(),[&](const int& a, const int& b)
     {
         if(a < b)
         {
             return true;
         }
         return false;
     });
~~~
  
其中“[&]"表示的是捕获方式，Lambda 表达式有三种捕获方式：
  
1. ”[&]“，用引用去捕获所有外部变量，表示该 Lambda 表达式使用一个引用以捕获所有局部变量。这意味着Lambda函数内的局部变量实际上是对外部作用域中的同名变量的别名。在Lambda函数中，对于捕获的变量的修改将直接影响外部作用域中的原始变量。  
2. "[=]"：以复制方式捕获所有外部变量。Lambda 表达式使用复制构造函数以复制传递给它们的所有数据，并在 Lambda 函数体中使用该复制的变量副本。当然，对于复制的对象进行修改，不会影响到原始参数。  
3. "[cur]"：以复制方式仅捕获外部变量 cur。这种方式只捕获指定的变量，每个指定的变量都使用复制构造函数以复制传递给它们的数据。
  
### 捕获变量例
  
这里的捕获变量不是说 lambda 表达式定义的函数参数，而是对函数外部变量的使用，比如  
~~~c++
int a = 1;
auto func = [=]() mutable
{
    a += 1;
    return a;
};a
std::cout << func() << std::endl;	// 2
std::cout << a << std::endl;	// 1
~~~
  
这里就是对 int a，lambda表达式用那种方式使用，而非()内的参数。  
还有一点
  
~~~c++
int a = 1;
auto func = [=](int& a) mutable
{
    a += 1;
    return a;
};
int b = 0;
std::cout << func(b) << std::endl;	// 1
std::cout << b << std::endl;	// 1
~~~
  
这里的(int& a)对a有更高优先级，就是先匹配函数参数，外部定义的 int a，lambda函数是无法使用的。
  
1. [&], 所有参数全为引用
  
   ~~~c++  
   int a=1;  
   auto func=[&]()  
   {  
       a+=1;  
       return a;  
   }  
   std::cout << func()<<std::endl;	// 2  
   ~~~
  
2. [=], 所有变量都复制捕获
  
   ~~~c++  
   int a = 1;  
   auto func=[=]() mutable  
   {  
       a+=1;  
       return a;  
   };  
   std::cout << func() <<std::endl;	// 2  
   std::cout << a <<std::endl;	// 1  
   ~~~
  
3. [cur], 指定某个参数的捕获方式
  
   ~~~c++  
   int num1 = 1, num2 = 2;  
   auto func = [num1, &num2]() mutable{  
   	num1 += 1;  
   	num2 += 1;  
   	return num1 + num2;  
   };  
   std::cout << func() << std::endl; // 2+3=5  
   std::cout << num1 << num2 << std::endl; // 1,3  
   ~~~
  
4.[], 如果在 Lambda 表达式中使用了 `[]`，而没有写任何符号，表示不捕获任何外部变量。这种情况下，**Lambda 表达式只能访问自己内部定义的变量和函数，不能访问任何外部的变量和函数。**  
对于之前的a，b参数覆盖问题就应该声明成[]。
  
~~~c++
int a = 1;
auto func = [](int& a)
{
    a += 1;
    return a;
};
int b = 0;
std::cout << func(b) << std::endl;	// 1
std::cout << b << std::endl;	// 1
~~~
  
### mutable
  
`mutable` 是 C++ 中的一个关键字，用于修饰类的成员变量或者 Lambda 表达式中捕获的变量，表示这些变量是可变的，即使在 const 或者引用等限制的情况下也可以被修改。
  
在 Lambda 表达式中，如果想要修改按值捕获的外部变量，需要将其声明为 `mutable`，否则会编译错误。例如：  
~~~c++
int a = 10;
auto func = [a]() mutable {
    a += 1;
    return a;
};
std::cout << func() << std::endl; // 输出 11
std::cout << a << std::endl; // 输出 10
~~~
  
### 指定返回值类型
  
lambda 表达式可以指定返回值类型，只需要  
~~~c++
function<int(int)> dfs=[&](int cur) -> int {
    return cur;
};
~~~
  
这里的`->`就指定了返回值 int 类型。  