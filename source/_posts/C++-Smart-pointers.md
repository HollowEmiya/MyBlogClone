---
title: C++智能指针、RAII、Block(To Be Continued)
date: 2022-04-13 11:14:05
tags:

---
  
# C++ : 智能指针(Smart Pointers)
  
现代c++编程中，标准库中包含了**智能指针**，这些指针用于确保程序不会出现内存和资源泄露，并切具有异常安全。
  
<!--more-->
  
## 对象生存期和资源管理(RAII)
  
*在我们学习智能指针前了解RAII是很有必要的。*
  
众所周知，与托管语言不同，C++没有自动**垃圾回收**。RAII是一个在程序运行是释放**堆内存**和其他资源的内部进程。C++程序将获取的所有资源返还给操作系统，但是如果释放未使用的内存失败，就会造成**内存泄露**的现象，在进程退出前，其他程序将无法使用被泄露的内存。
  
现代C++通过声明**堆栈**(Stack)上的对象来尽可能避免使用**堆内存**(Heap)，如果资源对于**堆栈**(Stack)而言过于庞大，则该资源应归**对象**(Object)所有。并且在对象初始化时，获取其拥有的资源。在对象的析构函数中释放资源，拥有该资源的对象本身应是在堆栈上被声明的。这个*对象拥有资源*的原则就是RAII(Resource acquisition is initialization.)
  
> Resource 资源  
>   
> acquisition 获取  
>   
> initialization 初始化
  
当资源所属的堆栈对象超出作用范围，会自动调用其析构函数，这样一来，C++中的垃圾回收和对象的生存周期便密不可分，而且具有确定性。资源将总是能在程序的已知点释放，我们可以控制该点，只有C++这种确定的析构函数形式，才能平等地处理内存和非内存的资源。
  
### 为什么要提RAII？
  
因为在对象拥有内存资源时，我们要在析构函数中将其对应地释放，如：
  
    class widget  
    {  
    private:  
        int* data;  
    public:  
        widget(const int size) { data = new int[size]; } // acquire  
        ~widget() { delete[] data; } // release  
        void do_something() {}  
    };
    
    void functionUsingWidget() {  
        widget w(1000000);   // lifetime automatically tied to enclosing scope  
                            // constructs w, including the w.data member  
        w.do_something();
    
    } // automatic destruction and deallocation for w and w.data
  
`int *data`是内存的资源，我们在析构函数中要对应的`delete`掉，而自C++11起有一种更好的写法，就是**使用智能指针**。智能指针会处理它自身所拥有的内存的**分配**和**删除**，而且使用智能指针就无需在类里显示析构`widget`函数。
  
    #include <memory>  
    class widget  
    {  
    private:  
        std::unique_ptr<int> data;  
    public:  
        widget(const int size) { data = std::make_unique<int>(size); }  
        void do_something() {}  
    };
    
    void functionUsingWidget() {  
        widget w(1000000);   // lifetime automatically tied to enclosing scope  
                    // constructs w, including the w.data gadget member  
        // ...  
        w.do_something();  
        // ...  
    } // automatic destruction and deallocation for w and w.data
  
> 通过使用智能指针进行内存分配，可以消除内存泄漏的可能性。 此模型适用于其他资源，例如文件句柄或套接字。
  
C++ 的设计可确保对象在超出范围时被销毁。 也就是说，当块(Block)以构造相反的顺序退出时，它们将被销毁。 销毁对象时，将按特定顺序销毁其基项和成员。 在全局范围内在任何块之外声明的对象都可能会导致问题。 如果全局对象的构造函数引发异常，则调试可能很困难。
  
> block是一个对象，这个对象里包含了要执行的代码片段以及一些状态信息。  
>   
> block是一片具有以下特性的内联代码片段集合:  
> 可以像函数一样有类型参数；  
> 可以声明或推算出一个返回类型；  
> 可以访问和block定义在同一个词法范围里的变量（即Status）；  
> 可以修改同一个词法范围里的变量；  
> 同一个词法范围的block之间可以共享变量和变量的修改结果；  
> 当栈被摧毁后，栈里的block依旧可以保持状态信息；
  
#### 什么是句柄
  
> 1.句柄就是一个标识符，只要获得对象的句柄，我们就可以对对象进行任意的操作。  
>   
> 2.句柄不是指针，操作系统用句柄可以找到一块内存，这个句柄可能是标识符，map的key，也可能是指针，看操作系统怎么处理的了。fd算是在某种程度上替代句柄吧；Linux 有相应机制，但没有统一的句柄类型，各种类型的系统资源由各自的类型来标识，由各自的接口操作。  
>   
> 3.在操作系统层面上，文件操作也有类似于FILE的一个概念，在Linux里，这叫做文件描述符fd(File Descriptor)，而在Windows里，叫做句柄(Handle)(以下在没有歧义的时候统称为句柄)。用户通过某个函数打开文件以获得句柄，此后用户操纵文件皆通过该句柄进行。
  
## 智能指针
  
智能指针是std命名空间定义的，其对于RAII编程至关重要。此习惯用法主要目的是确保**资源获取**和**对象初始化**同时发生，从而能够创建该对象的所有资源并在某行代码中准备就绪。
  
实际情况中，RAII的主要原则就是将任何**堆分配资源** *(例如，动态分配的内存或系统对象句柄)* 的所有权授予**堆栈分配的对象**，该对象的析构函数包含用于删除或释放资源的代码以及任何关联的清理代码。
  
大多数情况下，初始化原始指针或资源句柄指向实际资源时，会立即将指针传递给智能指针。在现代C++中，原始指针仅仅用于范围有限的小代码块、循环或者性能至关重要且不会混淆所有权的`Helper`函数中。
  
原始指针和智能指针对比：
  
    void UseRawPointer(){  
        Song* pSong = new Song(L"Rock Rill", L"Rock Rill");  
        ......  
        // dont forget to delete  
        delete pSong;  
    }
    
    
    void UseSmartPointer(){  
        unique_ptr<Song> song2(new Song(L"Rock Rill", L"Rock Rill"));  
        ......  
        wstring s = song2->duration_;  
        ...  
    }// song2 is deleted automatically here
  
如上所述，智能指针是我们在堆栈(Stack)上声明的类模板，并可通过使用指向某个堆(Heap)分配的对象的原始指针进行初始化。在初始化智能指针后，它将拥有那个原始指针。这意味着智能指针将负责删除原始指针指定的内存。智能指针析构函数包括了删除的调用(the call to delete)，并且由于智能指针是在堆栈(Stack)上声明的，当智能指针超出范围时，即便堆栈上空的某个位置引发异常，也会调用其析构函数。
  
使用指针操作(`->`和`*`)访问封装指针，智能指针类将重载这些运算符以返回封装的原始指针。
  
C++智能指针思路类似于**在语言(如，C#、Java)中创建对象的过程**：创建对象后让系统负责在正确的时间将其删除。不同之处在于，单独的垃圾回收器不在后台运行；按照标准C++范围规则对内存进行管理，以便运行时环境更快速、更有效率。
  
**注意：要始终在单独的代码行上创建智能指针，而万万不可在参数列表内创建智能指针，这样就不会由于某些参数列表分配规则而发生轻微资源泄露的现象。**
  
`unique_ptr`封装指向大型对象的指针示例：
  
    class LargeObject{  
        public:  
            void DoSomething(){}  
    };
    
    void ProcessLargeObject(const LargeObject& lo){}  
    void SmartPointerDemo(){  
        // Creat the Object and Pass it to a smart pointer  
        std::unique_ptr<LargeObject> pLarge(new LargeObject());
          
        // Call a method on the object  
        pLarge->DoSomething();
    
        // Pass a reference to a method  
        ProcessLargeObject(*pLarge);  
    }// pLarge is deleted automatically when function block goes out of scope.
  
上述例子展示了如何使用智能指针执行以下关键步骤。
  
    1.将智能指针声明为一个自动(局部)变量。(不在智能指针自身上使用`new`或`malloc`表达式)。即`std::unique_ptr<LargeObject> pLarge(new LargeObject());`而不是传统的原始指针：`LargeObject* pLarge = new LargeObject();`，`new LargeObject()`就是new表达式。
  
也就是说智能指针要直接初始化。
  
    2.在类型参数中，指定封装指针的指向类型。即`std::unique_ptr<LargeObject> pLarge(new LargeObject());`的`<LargeObject>`。
  
    3.将原始指针传递到智能指针构造函数中的新对象。(某些实用工具函数或智能指针构造函数可为我们执行此操作。)上例中的智能指针构造函数即是。
  
    4.使用`->`和`*`访问对象。
  
    5.允许智能指针删除对象。
  
智能指针的设计原则是在内存和性能上尽可能高效。例如，`unique_ptr` 中的*唯一数据成员*是**封装的指针**。 这意味着，`unique_ptr` 与该指针的大小完全相同，不是四个字节就是八个字节。 使用重载 * 和 -> 运算符的智能指针访问封装的指针并不比直接访问原始指针慢得多。
  
**智能指针具有其自己的成员函数**，这些函数通过使用 `.` 表示法进行访问。 例如，某些C++ 标准库智能指针具有释放指针所有权的重置成员函数。 如果我们想要在智能指针超出范围之前**释放其内存**将很有用，如以下示例所示：
  
    void SmartPointerDemo2()  
    {  
        // Create the object and pass it to a smart pointer  
        std::unique_ptr<LargeObject> pLarge(new LargeObject());
    
        //Call a method on the object  
        pLarge->DoSomething();
    
        // Free the memory before we exit function block.  
        pLarge.reset();
    
        // Do some other work...
    
    }
  
**智能指针通常提供直接访问原始指针的方法。** 
  
 C + + 标准库智能指针具有 `get` 用于此目的的成员函数，并且 `CComPtr` 具有公共 `p` 类成员。 通过提供对基础指针的直接访问，你可以使用智能指针管理你自己的代码中的内存，还能将原始指针传递给不支持智能指针的代码。
  
    void SmartPointerDemo4()  
    {  
        // Create the object and pass it to a smart pointer  
        std::unique_ptr<LargeObject> pLarge(new LargeObject());
    
        //Call a method on the object  
        pLarge->DoSomething();
    
        // Pass raw pointer to a legacy API  
        LegacyLargeObjectFunction(pLarge.get());    
    }
  
## 智能指针的类型
  
### C++标准库智能指针
  
> 使用这些智能指针作为将指针封装为纯旧 C++ 对象 (POCO) 的首选项。  
>   
> plain old C++ object
  
* `unique_ptr`
  
  只允许基础指针的一个所有者。除非我们确信需要`shared_ptr`，否则使用该指针作为*POCO的默认选项*。**可以移到新所有者，但不能复制或共享**。替换已经舍弃的`auto_ptr`。和`boost::scoped_ptr`相比，`unique_ptr`小巧且高效。大小仅为一个指针，而且支持右值引用，以便从C++标准库集合中快速插入和检索。
  
* `shared_ptr`
  
  采用引用计数的智能指针，如果我们想要将一个原始指针分配给多个所有者(例如，从容器返回了指针副本又想保留原始指针时)，就可以使用该指针。直到所有`shared_ptr`所有者超出了范围或者放弃了所有权，才会删除原始指针。大小为两个指针：一个用于对象，另一个用于包含引用计数的共享控制块。
  
* `weak_ptr`
  
  结合`shared_ptr`使用的特例智能指针。`weak_ptr`提供对一个或者多个`shared_ptr`实例拥有的对象的访问，但是不参与引用计数。如果我们想要观察某个对象但是不需要其保持活跃状态，可以使用该实例。在某些情况下，需要断开`shared_ptr`实例间的循环引用。
  
*智能指针还和COM有关，等待后续的学习补充.……*  