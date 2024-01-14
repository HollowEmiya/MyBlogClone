---
title: C++ RAII
date: 2022-04-01 12:24:22
tags: 
typora-root-url: ./..
---
  
[What is RAII?](https://medium.com/swlh/what-is-raii-e016d00269f9)  
[Youtube Vedio](https://www.youtube.com/watch?v=q6dVKMgeEkk)
  
<!--more-->
  
# C++ RAII(Resurce Acquisition Is Initialization)
  
## RAII 用来做什么?
  
### 内存
  
当我们在 C++ 中分配内存时，比如开辟一个数组，`int *arr = new int[dynamicSize]` ,那么我后续一定要记得去 `delete[] arr`，否则就会发生**内存泄漏**。也就是管理内存都需要我们手动去进行。
  
### 锁
然后我们看看使用锁，假设我们想显式同步代码的某些部分。我创建一个 `std::mutex`，然后调用 `mutex.lock()`。如果忘记调用 `mutex.unlock()`，或者在解锁前发生异常(导致`mutex.unlock()`)未被调用，当其他线程尝试获取锁时将死锁。
  
### 线程
如果我们想在另一个线程运行代码，然后我创建了一个线程。如果我们记得`jion`就没有问题，但是如果我们忘记了，并且线程超出了自己的作用域，`std::terminate`将被调用，程序会终止。
  
#### 线程相关内容补充
  
在C++中，创建线程后，如果没有适当地处理线程的结束，可能会导致程序意外终止。
  
当你创建一个新的线程并开始执行代码时，主线程和新线程是并行执行的。如果在主线程退出之前没有 join（或者使用 detach 分离线程），那么主线程结束时，程序会调用 `std::terminate` 来终止所有尚未结束的线程。
  
这种情况的发生通常是由于在主线程退出之前，新线程尚未完成，而没有等待新线程结束（或者分离）而导致的。因此，在使用线程时，确保在适当的时机对线程进行 join 或 detach 是非常重要的。
  
##### 关于 join 和 detach
  
join 和 detach 是 c++ 两种管理线程的方法。
  
1. `join` 通过阻塞当前线程等待被调用的线程完成，确保主线程在子线程执行完毕后在继续执行，从而避免子线程在主线程退出去终止，或者超出作用域导致 `std::terminate` 导致整个进程结束。
  
   ~~~c++  
   std::thread t(threadFunction);
   
   t.jion();		// 阻塞该主线程，等待子线程完成  
   ~~~
  
2. `detach` 将当前线程和新线程分离，使其独立运行。一旦被分离后就无法使用 `join` 方法等待线程结束。<u>**此时**线程的生命周期就由自己控制了，主线程和子线程可以并发执行，互相独立。</u>
  
   ~~~c++  
   std::thread(threadFuntion);  
   t.detach();  
   ~~~
  
通过以上三个例子，我们可以看到错误发生是有一定规律的。首先在这些例子中我们获取了某些资源，如Heap上的内存、锁、线程。获取资源当然没有问题，但是当资源超出作用范围问题就发生了(内存泄露，死锁，进程终止)，在第一个问题中我们要时刻记得 delete、第二个需要 unlock，第三个需要 join，这显然是很繁琐的。
  
RAII 就是为了避免犯错和繁琐的手动释放。  
C++ 和 Java，Python等的垃圾回收不同其析构函数是显式的，离开作用域自动销毁。  
其他比如向 vector push一个资源，然后退出引用计数归零，但是不会立即删除，而是过段时间后调用一个 GC 程序将引用计数为零的对象删除。
  
但是如果对高性能计算有要求，如**时序**或**性能**就不能依赖 GC。
  
### RAII: 异常安全( exception-safe )
  
C++ 标准确保在异常发生时，也能调用 已创建对象的析构函数，因此 C++ 不需要 Java 那种 finally 语句确保执行。
  
### RAII 离不开  构造函数 和 析构函数
  
~~~c++
struct Pig
{
    std::string name;
    int age;
    
    Pig():name("Kit"),age(10)
    {
        
    }
    
    Pig(string n, int a):name(n),age(a)		// 拷贝构造函数
    {
        
    }
    
    Pig(string n, int a)		// 拷贝赋值函数
    {
        name = n;
        age = a;
    }
}; 

int main()
{
    Pig pig0;
    Pig pig1("Kiii",2);
}
~~~
  
* 高效：使用初始化列表可以避免重复初始化，如果不使用，首先会先将数据成员空初始化，然后在构造函数内部进行赋值。  
* 避免错误，因为不用初始化列表会先对成员进行无参初始化，而这时若类成员没有无参构造函数就会出错。  
* 类成员为 const，因为 const 对象只能初始化一次，所以只能在初始化列表中进行。
  
> 要记住非初始化列表都是先进行无参初始化然后再在函数内进行一次赋值，即一次初始化，一次赋值。
  
#### 构造函数单个参数
  
~~~cpp
struct Pig
{
	string name;
    int age;
    
    Pig(int age_):name("a Pig aged"+to_string(age_)),age(age_)
    {
        
    }
}

int main()
{
    Pig pig = 80;
}
~~~
  
这里 `= 80` 是隐式地调用了构造函数，如果想为了可读性避免这种隐式构造，可以使用 explicit。  
~~~c++
struct Pig
{
	string name;
    int age;
    
    explicit Pig(int age_):name("a Pig aged"+to_string(age_)),age(age_)
    {
        
    }
}

int main()
{
    // Pig pig = 80;
    Pig pig(80);
}
~~~
  
explicit 使其必须使用`(80)`这种格式显式调用构造函数。  
explicit 的作用就在于避免因为隐式调用而导致代码可读性的降低。
  
#### explicit 多参数
  
~~~c++
struct Pig
{
    string p_name;
    int p_age;
    
    explicit Pig(string name, int age):p_name(name),p_age(age)
    {
        
    }
};

int main()
{
    Pig pig = {"Kit",5};	// Error
    Pig pig2{"Kit2",7};		// Right
    Pig pig3("Kit3",10);	// Right
}
~~~
  
explicit 可以用于多参数，并且在上面例子中，explicit 避免了 `{"kit",5}` 的隐式调用，并且接受了 `Pig pig2{"Kit2",7}` 的显式调用。
  
#### () 和 {} 调用的区别
  
1. `int(3.14f) ` 不会报错，但是 `int{3.14f}` 会报错，因为 {} 是非强制类型转换。  
2. 假如 使用 `Pig pig2{"jit2",4.5}` 这就会出错了……
  
by the way，现在别使用 int() 了，太蠢了，而且不安全，比如 要是转换为指针可能会发生错误。应该使用 `static_cast<int>()` 进行。
  
### 编译器默认生成的构造函数：无参 ( POD 陷阱 )
  
当一个类没有定义任何构造函数，且类成员都有无参的构造函数，编译器会自动生成一个无参构造函数，会调用每个成员的无参构造函数。
  
**注意**这些类型**不会被初始化为0**。
  
1. int, float, double 等基础类型  
2. `void*`, `Object*` 等指针类型  
3. 完全有这些类型组成的类
  
* 这些类型称为 **POD( plain-of-data )**  
  POD 的存在是出于兼容性和性能考虑，这些类型会将其所在内存位置的数据直接作为自己的初始值。
  
所以如果你打算使用编译器自动生成的初始化函数，最好对类成员用 `{}` 进行一个指定初始化，这样在 自动生成的构造函数中就会初始化为你指定的值。  
~~~c++
struct Pig
{
    string p_name;
    int weight{0};		// 自动生成的构造函数会将其初始化为0
};
~~~
  
或者使用`=`也可以，但是要注意类对象其本身的构造函数是否有 explicit 修饰  
~~~c++
struct Demo
{
    explicit Demo(string a, string b)
    {
        
    }
};

struct Pig
{
    string p_name;
    int weight = 0;
    Demo pd{"a","b"};		// 可以通过编译
    // Demo pd = {"a","bb"};	// 这样就不行
};
~~~
  
>`int x{};  
>void* p{};`  
>  
>和  
>  
>`int x{0};  
>void* p{nullptr};`  
>  
>等价，都是**零初始化**
  
### 编译器默认生成的构造函数：C++11 初始化
  
在 C++11 中，如果一个类(和他的基类) 没有定义任何构造函数，这时编译器会自动生成一个**参数个数和成员一样的构造函数**。
  
会将 {} 内容，**按顺序赋值给对象的每一个成员**
  
不过这种只支持 {} 或者 = {}
  
~~~c++
struct Pig
{
    int a;
    int b;
};

struct Deo
{
    Pig pig;
    int a;
};

int main() {
    // Write C++ code here
    std::cout << "Hello world!";
    Pig pig{1,2};
    std::cout << pig.a << std::endl;			// 1
    Deo deo{{3,2},1};
    std::cout << deo.pig.a << std::endl;		// 3
    return 0;
}
~~~
  
当然这时候无参构造函数也是存在的。
  
和默认指定结合  
~~~c++
struct Pig
{
    string p_name;
    int p_weight{0};
};

int main()
{
    Pig pig{"kit"};		// weight 未指定，使用 0
}
~~~
  
### 有自定义构造函数时仍想用默认构造函数：`=default`
  
一旦我们自定义了构造函数，编译器便**不会生成默认的无参构造函数**  
但是如果我们仍然想使用，就可以用 `=default` 关键字。
  
~~~C++
struct Pig
{
    string pName;
    int pWeight;
    
    Pig() = default;
    
    Pig(string name, int wei)
        :pName(name),pWeight(wei)
        {
            
        }
}
~~~
  
*but 初始化列表的那个构造函数好像不能 =default 出来*
  
### 编译器默认生成的构造函数：拷贝构造函数
  
`Pig(cosnt Pig& other)`  
参数为 Pig 类型，调用如下
  
~~~c++
int main()
{
    Pig pig{"Kit",90};
    
    Pig pig2 = pig;
    // Pig pig2(pig);		// 与上面的一样
}
~~~
  
即便我们自定义了构造函数，拷贝构造函数也不会删除。
  
### 舍弃拷贝构造函数：`=delete`
  
如果不想使用拷贝构造函数可以使用 `delete` 修饰  
~~~C++
struct Pig
{
    string pName;
    int pWeight{0};
    
    Pig()=default;
    
    Pig(string name, int wei):pName(name),pWeight(wei)
    {}
    
    Pig(const Pig&) = delete;		// 禁止拷贝构造
    Pig& operatpor=(Pig const &) = delete;	//  禁止拷贝赋值
}
~~~
  
`Pig pig2 = pig;` 
  
这是**拷贝构造**  
~~~C++ 
Pig pig2;	// 无参构造
pig2 = pig;	// 拷贝赋值
~~~
  
这是**拷贝赋值**
  
#### 为什么拷贝赋值函数返回值是引用
  
为了实现链式赋值，返回要是左值引用，而非临时对象的将亡值。
  
### 构造函数全家桶
  
#### 三五法则：一
  
<img src="imgs/C++RAII/35.png">
  
1. 如果定义了析构函数，为了防止 doulefree 要么 delete 拷贝构造和拷贝赋值，要么自定义。  
   也就是深拷贝和浅拷贝。  
   浅拷贝会导致两个对象内的指针指向同一内存，所以我们要手动实现深拷贝。  
   如果定义了析构函数很有可能是因为我们类内对象有些无法被默认的析构函数销毁，  
   所以可能会有深拷贝和浅拷贝问题，如果只需要浅拷贝那就删除拷贝构造和拷贝赋值函数，C++会默认生成的。  
2. 如果不实现拷贝赋值函数，编译器可能会以 析构+拷贝构造 的方式实现拷贝赋值的效果。  
3. 与上面同理。
  
#### 拷贝和移动
  
有时候我们只需要一份 data，不需要复制，我们更希望把 对象 **移动**过去。  
拷贝是 O(n)，移动是 O(1)  
可以使用 std::move  
v2 被移动到 v1 后原来的 v2 会被清空，所以一定要确保 v2 后续不会会再被使用。 
  
#### 移动进阶：swap
  
我们不仅可以使用move  
也可以使用 `std::swap` 交换 v1 和 v2  
可以利用 swap 实现双缓存技术。
  
#### 哪些情况会触发 move
  
* 会触发move  
  * `return v2;`  作为返回值  
  * `v1 = std::vector<int>(200)` 就地构造  
  * `v1 = std::move(v2)` 显式move  
* 拷贝  
  * `return std::as_const(v2)`，显式拷贝   
  * `v1 =v2` , 默认拷贝  
* 下式不会 move 也不会 copy  
  * `std::move(v2)`  
  * `std::as_const(v2)`  
  * 这两个函数只负责转换类型，不会发生实际的 copy 和 move。
  
### 移动构造函数：缺省实现
  
* 移动构造 ≈ 拷贝构造 + 解构他 + 他默认构造   (这里的他是移动的源对象)  
* 移动赋值 ≈ 拷贝赋值 + 他结构 + 他默认构造
  
只要 我们不自己定义，编译器就会这样做  
这也就是三五法则中 第4点, 为什么定义 **拷贝构造** 和 **拷贝赋值** 后最好定义 **移动构造** 和 **移动赋值**
  
## RAII 解决内存管理：`unique_ptr`
  
C++11 引用 `unique_ptr` 容器，其结构函数会调用 delete p。  
~~~c++
struct C
{
    C(){printf("C");}
    ~C(){printf("~C");}
}

int main()
{
    std::unique_ptr<C> p = std::make_unique<C>();
    if(..)
    {
        return 1;	// 自动释放 p
    }
    return 0; 		// 自动释放 p
}
~~~
  
而且将 C++98 的   
~~~C++
delete p;
p = nullptr;
~~~
  
封装为了一个操作：  
`p = nullptr;` 等价于 `p.reset();`
  
#### `uniqur_ptr`: 禁止拷贝
  
为什么禁止拷贝，如果拷贝了这个指针，  
而 `unique_ptr` 的析构函数使用了 delete，可能就会出现 double free 的问题。  
所以，在我们写函数时，如果用 `unique_ptr` 作为参数就会报错。
  
##### 解决方案1：获取原始指针( C* )
  
* 第一种情况，我们不需要 夺走 资源的 **占有权**  
  比如只是一个函数调用，**并不需要结果掌管对象生命周期的大权**。  
  使用 `p.get()` 获取指针，然后进行操作。
  
  ~~~C++  
   void func(C* cp)  
   {
       
   }
  
  func(p.get());  
  ~~~
  
* 第二种，我们需要夺走 资源的 **占有权**。  
  ~~~c++  
  std::vector<std::unique_ptr<C>> objectList;
  
  void func(std::unique_ptr<C> p)  
  {  
      objectList.push_back(std::move(p));  
  }
  
  int main()  
  {  
      std::unique_ptr<C> p = std::make_unique<C>();  
      printf("移交前:%p\n",p.get());		// 不为null  
      func(std::move(p));  
      printf("移交后:%p\n",p.get());		// 为null  
      return 0;  
  }  
  ~~~
  
  因此我们要使用 move 接过掌管对象生命周期的大权。
  
#### 移交控制权后仍希望访问到 p 指向的对象
  
我们移交后，原来的指针就变 nullptr，这时无法访问。  
##### 解决办法：提前获取指针
  
使用get函数提前获取原始指针，  
不用担心周期问题，这个原始指针的所有权归move后的新`unique_ptr` 所有。  
`C* raw_p = p.get();`
  
但是要时刻注意 move 后的 `unique_ptr` 是否被删除！
  
~~~c++
int main()
{
    std::unique_ptr<C> p = std::make_unique<C>();
	C *raw_p = p.get();
	func(std::move(p));
    
    raw_p->do();
    
    objlist.clear();
    
    raw_p->do();	// !!! Error 悬空指针
}
~~~
  
## `shared_ptr`
  
* `unique_ptr` 由于为了解决 double free 而**禁止拷贝**，导致使用起来很困难，容易犯错。  
* `shared_ptr`，牺牲效率换取自由度，通过 **引用计数** 来解决 **重复释放** 的问题。
  
1. 当 `shared_ptr` 被初始化时，将计数器设为1；  
2. 当  `shared_ptr` 被拷贝时， 计数器 +1；  
3.  当一个  `shared_ptr`  被解构时，计数器 -1，直到计数器为 0，则自动销毁其所指向的对象。
  
* 我们可以使用 `p.use_count()` 获取当前的引用计数。
  
###  `shared_ptr` 和 循环引用
  
~~~cpp
struct C
{
    std:;shared_ptr<C> m_child;
    std::shared_ptr<C> m_parent;
};

int main()
{
    auto parent = std::make_shared<C>();
    auto child = std::make_shared<C>();
    
    parent->m_child = child;
    child->m_parent = parent;
    // 这里 parent 和 child 互相引用
    
    parent = nullptr;	// 失败，child 的 m_parent 还在引用
    child = nullptr;	// 失败，parent 的 m_child 还在引用
    
    return 0;
}
~~~
  
即便 main 函数退出，这两块内存都无法释放。
  
## `weak_ptr` 
  
### 循环引用：解决方案1
  
* 将逻辑上“不具有所属权” 的那一个 `shared_ptr`，改为 `weak_ptr` 即可。  
  ~~~c++  
  struct C  
  {  
      std:;shared_ptr<C> m_child;  
      std::weak_ptr<C> m_parent;  
  };
  
  int main()  
  {  
      auto parent = std::make_shared<C>();  
      auto child = std::make_shared<C>();
      
      parent->m_child = child;  
      child->m_parent = parent;  
      // 这里 parent 和 child 互相引用
      
      parent = nullptr;	// 释放，因为 child 指向的是其 **弱引用**  
      child = nullptr;	// 释放，因为指向 child 的 parent 已经释放了
      
      return 0;  
  }  
  ~~~

  
#### 不影响 shared_ptr 计数：弱引用 weak_ptr
  
* expired() 可以判断，weak_ptr 是否失效，如果 shared_ptr 已经释放了，该 weak_ptr 就失效了。  
  `waek_p.expired()`
  
* lock()，如果有需求可以随时使用 lock 函数产生一个新的 shared_ptr。但不 lock 时 weak_ptr 不会影响计数。  
  ~~~c++  
  std::shared_ptr<C> p = std::make_shared<C>();  
  std::weak_ptr<C> weak_p = p;  
  weak_p.lock()->do_something();	// 执行完毕后 lock 出来的 shared_ptr 就销毁了，不影响 shared_ptr 的计数，我理解为 lock 返回一个 shared_ptr 的将亡值，这行结束后自动解构  
  ~~~
  
## 智能指针：作为类的成员变量
  
1. unique_ptr，**当该对象仅属于我**，比如父窗口下的子窗口  
2. 原始指针，**当对象不属于我，但是他释放**<u>**前**</u>**我必然被释放。**  
   有一定风险，比如子窗口中指向父窗口的指针。  
3. shared_ptr：**当有多个对象共享时，或虽然该对象仅属于我，但是有 weak_ptr 的需要**  
4. weak_ptr：**当该对象不属于我，且他释放**<u>**后**</u>**我仍可能不被释放。**比如：指向窗口上一次被点击的元素。  
5. 初学者可多用 shard_ptr 和 weak_ptr 的组合，更加安全。
  
### 循环引用解决方案2：设置为原始指针
  
~~~c++
struct C
{
	std::shared_ptr<C> m_child;
	C* m_parent;
}

int main()
{
	auto parent = std::make_shared<C>();
	auto child = std::make_shared<C>();
	
	parent->m_child = child;
	child->m_parent = parent.get();		// 这里 parent.get 得到的原始指针是归属于 shared_ptr parent 的
	
	parent = nullptr;
	child = nullptr;
	
	return 0;
}
~~~
  
还有更适合父窗口-子窗口的解决方案。  
刚才提到原始指针的应用场景：当一个对象不属于我，但是他释放<u>前</u>我必须释放。  
这里可以发现父窗口的释放必须导致子窗口的释放。
  
## 智能指针使用
  
### 成员都是安全类型：五大函数，一个也不用声明
  
* 如果类的所有成员都是 **安全** 类型，那么五大函数都无需声明，你的类型就是自动**安全的**。  
* 最好的判断方式是：如果你不需要 **自定义的解构函数** 这个类就无需担心。  
* 如果我们需要自定义 解构函数，往往意味着你的类成员中，包含不安全类型。  
* 一般有两种情况：  
  1. **类管理着资源**  
  2. **类是数据结构**
  
#### 管理着资源(仅需要浅拷贝)：删除拷贝函数，统一用 shared_ptr 管理
  
* 因为资源，往往是不能被复制的。比如 一个 openGL 的 shader
  
* 如果允许拷贝(浅拷贝)，就相当于把 标记资源的 int 复制两遍，之后就会出现 double free 的问题。
  
#### 数据结构(需要深拷贝)：如果可以，定义拷贝和移动
  
* 我们设计的**数据结构**通常是支持深拷贝的，这需要我们自己去定义实现，如果实在无法实现那就删除。
  
### 函数如何避免拷贝
  
使用常引用：`Pig const & pig`
  
#### 如何避免不经意的隐式拷贝
  
我们可以将 拷贝构造函数声明为 explicit   
这样隐式拷贝就会出错，让我们发现隐式拷贝的发生。
  
## 后续阅读
  
1. P-IMPL模式  
2. 虚函数与纯虚函数  
3. 拷贝如何作为虚函数  
4. `std:unique_ptr::release()`  
5. ` std:.enable_shared_from_this`  
6. `dynamic_cast`  
7. `std::dynamic_pointer_cast`  
8. 运算符重载  
9. 右值引用&&  
10. `std::shared_ptr<void>`和 `std:any`  