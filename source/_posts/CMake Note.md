---
title: 我的CMake学习
date: 2023-03-10 13:15:23
tags:
typora-root-url: ./..
---
  
对 CMake 粗浅的学习，之前看一个up的讲的一般，后来看了小彭老师，的确不错，因为官方文档看起来有点吃力，所以就……  
学习过程中把知识点做了一点笔记方便后续复习
  
<!--more-->
  
# CMake
  
本文档为学习笔记，该部分内容来源[b站Up刘贝斯的CMake教学](https://www.bilibili.com/video/BV1vR4y1u77h/?p=2&spm_id_from=pageDriver&vd_source=1fa1b82383f6efb8a2632316da9afad0)。
  
## What is CMake?
  
CMake 是一种高级编译配置工具。
  
当多人使用一种或不同种语言、编译器开发一个项目时，最终需要输出一个**可执行文件**或者**共享库**(dll, so等等)，此时——CMake就可以帮助到我们，不必用G++或GCC逐个编译我们所写过的代码。  
所有操作都是通过编译CMakeLists.txt完成的。  
使用CMake来处理大型的C/C++/Java等项目。
  
## CMake 安装
  
* Linux大多都有安装。  
* Windows, 下载网站(https://cmake.org/download/)
  
## CMake: Hello World
  
1.首先写一个C/C++的hello world
  
~~~c++
#include <iostream>

int main()
{
    std::cout << "Hello World!" << std::endl;
}
~~~
  
2.编写`CmakeLists.txt`
  
~~~cmake
PROJECT (HELLOW)

SET(SRC_LIST main.cpp)

MESSAGE(STATUS "This is BINARY dir " ${HELLO_BINAR_DIR})

MESSAGE(STATUS "This is SOURCE dir " ${HELLO_SOURCE_DIR})

ADD_EXECUTABLE{hello $(SRC_LIST)}
~~~
  
3.使用CMake，生成`makefile`文件
  
~~~cmake
cmake .  
~~~
  
### CMake: Hello World语法介绍
  
#### `PROJECT`关键字 —— [简书CMake命令之project](https://www.jianshu.com/p/cdd6e56c2422)
  
可以用来**指定工程的名字和支持的语言**，默认支持所有语言。  
PROJECT (HELLO) 指定了工程的名字——HELLO，并且支持所有语言。
  
# [**CMake教程**](https://www.bilibili.com/video/BV16P4y1g7MH)
  
## 第一章 添加源文件
  
* 第一种，添加名为 *main* 的 *executable* ，源文件为 *main.cpp* 。
  
~~~cmake
add_executable(main main.cpp)
~~~
  
* 第二种，先创建 **目标( executable )**，稍后再添加源文件。
  
~~~cmake
add_executable(main)
target_sources(main PUBLIC main.cpp)
~~~
  
### 若有多个源文件呢？
  
逐个添加即可
  
~~~cmake
add_executable(main)
target_sources(main PUBLIC main.cpp other.cpp)
~~~
  
或者使用**变量**来存储
  
~~~cmake
add_executable(main)
set(sources main.cpp other.cpp)
target_sources(main PUBLIC ${sources})
~~~
  
建议把头文件也加上，这样在VS中可以出现在 "Header Flies" 一栏
  
~~~cmake
add_executable(main)
set(sources main.cpp other.cpp other.h)
target_sources(main PUBLIC ${sources})
~~~
  
我们还可以使用 **GLOB** 自动查找当前目录下指定拓展名的文件，实现批量添加源文件
  
~~~cmake
add_executable(main)
file(GLOB sources *.cpp *.h)
target_sources(main PUBLIC ${sources})
~~~
  
但是使用 **GLOB** 需要注意，如果我们增添新的源文件时，CMake可能不会更新，所以要启用 **CONFIGURE_DEPENDS** 选项，当添加新文件时，**在 Build 时进行检测**，自动更新变量
  
~~~cmake
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})
~~~
  
### 源码在子文件夹中？
  
#### 例子：
  
* mylib  
  * other.cpp  
  * other.h  
* CMakeLists.txt  
* main.cpp
  
将**路径名、后缀名全部**写出来
  
~~~cmake
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h mylib/*.cpp mylib/*.h)
target_sources(main PUBLIC ${sources})
~~~
  
然鹅，dark不必，我们可以使用 **aux_source_directory** , 自动搜集需要的文件后缀名  
本例中`aux_source_directory(. sources)`和`aux_source_directory(mylib sources)`  表示**当前目录**和 **mylib 目录** 全部加入项目中。
  
~~~cmake
add_executable(main)
aux_source_directory(. sources)
aux_source_directory(mylib sources)
target_sources(main PUBLIC ${sources})
~~~
  
更进一步：**GLOB_RECURES** 能够自动包含所有子文件夹下的文件  
但是注意，**GLOB_RECURES** 会把 **build 目录**下的临时 .cpp 文件( 这些临时文件是 CMake 为了测试编译器 )也加进来。  
解决办法：一种，可以把源码统一放到 src 目录下。二种，要求使用者不要把 build 放到和源码同一个目录里。这两种之间前者好一点。
  
~~~cmake
add_executable(main)
file(GLOB_RECURES sources CONFIGURE_DEPENDS *.cpp *.h)
target_source(main PUBLIC ${sources})
~~~
  
## 第二章 项目配置变量——BUILD_TYPE
  
### CMAKE_BUILD_TYPE 构建的类型，调试模式 or 发布模式 
  
* CMAKE_BUILD_TYPE 是 CMake 中一个特殊的变量， 用于控制构建类型，他的值可以是：  
  * Debug 调试模式，完全不进行优化，生成调试信息，方便调试程序  
  * Release 发布模式，优化程度最高，性能最佳，但是编译比 Debug 慢  
  * MinsizeRel 最小体积发布，生成的文件比 Release 更小，不完全优化，减少二进制体积。  
  * RelWithDebInfo 带调试信息发布，生成的文件比 Release 更大，因为带有调试的符号信息。  
* 默认情况下，CMAKE_BUILD_TYPE 为空字符，这时相当于 Debug。
  
~~~cmake
cmake_minimun_required(VERSION 3.15)
project(hellowcmake LANGUAGES CXX)

set(CMAKE_BUILD_TYPE Release)

add_executable(main main.cpp)
~~~
  
各种构建模式在编译器选项上的区别
  
* 在 Release 模式下，追求的是程序最佳的性能表现，在此情况下编译器会对程序做最大的代码优化以达到最快的运行速度。另一方面，由于代码优化后不与源代码一致，此模式下一般会丢失大量的调试信息。
  
在编译器上各种构建类型的体现：
  
* Debug : '-O0 -g'  
* Release : '-O3 -DNDEBUG'  
* MinSizeRel : '-Os -DNDEBUG'  
* RelWithDebInfo : '-O2 -g -DNDEBUG'
  
此外，定义 NDEBUG 宏会使 assert 被去除掉。
  
因为默认情况下是 Debug 导致生成程序的效率很低。  
小技巧：设定一个变量的默认值
  
如何让 CMAKE_BUILD_TYPE 在用户没有指定的情况时为 Release，指定的时候保持用户的指定的值不变？  
即 CMake 默认情况下 CMAKE_BUILD_TYPE 是一个空字符串。  
因此可以通过 `if( NOT CMAKE_BUILD_TYPE )`判断是否为空，空则自动设置为 Release 模式。  
大多数 CMakeLists.txt 开头都会有这样三行，目的是让默认的构建类型为发布模式 (高度优化) 而不是默认的调试模式 (不会优化) 。
  
~~~cmake
if( NOT CMAKE_BUILD_TYPE )
	set(CMAKE_BUILD_TYPE Release)
endif()
~~~
  
### Project
  
project : 初始化项目信息，并把当前 CMakeLists.txt 所在位置作为根目录
  
这里初始化名为 hellocmake 的项目；为什么一定需要项目名？  
因为对于 MSVC，他会在 build 里生成 hellocmake.sln 作为 IDE 眼中的项目。  
CMAKE_CURRENT_SOURCE_DIR 表示**当前源码目录**的位置，例如 ~/hellocmake  
CMAKE_CURRENT_BINARY_DIR 表示**当前输出目录**的位置，例如 ~/hellocmake/build
  
~~~cmake
cmake_minumum_required(VERSION 3.15)
project(hellocmake)

message("PROJECT_NAME: ${PROJECT_NAME}")
message("PROJECT_SOURCE_DIR: ${PROECJR_SOURCE_DIR}")
message("PROJECT_BINARY_DIR: ${PROJECT_BINARY_DIR}")
message("CMAKE_CURRENT_SOURCE_DIR: ${CMAKE_CURRENT_SOURCE_DIR}")
message("CMAKE_CURRENT_BINARY_DIR: ${CMAKE_CURRENT_BINARY_DIR}")
add_executable(main main.cpp)
~~~
  
#### 和子模块的关系：PROJECT_X_DIR 和 CMAKE_CURRENT_x_DIR
  
PROJECT_SOURCE_DIR 表示最近一次调用 project 的 CMakeLists.txt 所在的源码目录。  
CMAKE_CURRENT_SOURCE_DIR 表示当前 CMakeLists.txt 所在的源码目录。  
CMAKE_SOURCE_DIR 表示最为外层 CMakeLists.txt 的源码根目录。  
利用 PROJECT_SOURCE_DIR 可以实现从子模块直接获取项目最外层的路径。  
**不建议使用 CMAKE_SOURCE_DIR , 那样会让你的项目无法被人作为子模块使用。**
  
>mylib got PORJECT_SOURCE_DIR: /home/bate/Codes/course/11/template  
>  
>mylib got CMAKE_CURRENT_SOURCE_DIR: /home/bate/Codes/course/11/template/mylib
  
#### 其他相关变量
  
* PROJECT_SOURCE_DIR : 当前项目源码路径( 存放 main.cpp 的地方)  
* PROJECT_BINARY_DIR : 当前项目输出路径 ( 存放 main.exe 的地方 )  
* CMAKE_SOURCE_DIR : 根项目源码路径 ( 存放 main.cpp 的地方 )  
* CMAKE_BINARY_DIR : 根项目输出路径 ( 存放 main.cpp 的地方 )  
* PROJECT_IS_TOP_LEVEL : BOOL 类型，表示当前项目是否是 ( 最顶层的 ) 根项目  
* PROJECT_NAME : 当前项目名  
* CMAKE_PROJECT_NAME : 根项目的项目名  
* 详见 : [CMake之Project](https://cmake.org/cmake/help/latest/command/project.html)
  
#### 子模块也可使用 project 命令，将当前目录作为一个独立的子项目
  
这样 PROJECT_SOURCE_DIR 就会是子模块的源码目录而不是外层了。  
这时候 CMake 会认为这个子模块是一个独立的项目，会额外做一些初始化。  
他的构建目录 PROJECR_BINARY_DIR 也会变成 build/<源码相对路径>  
这样在 MSVC 上也会看见 build/mylib.vcxproj 的生成
  
>PORJECT_NAME : hellowcmake  
>  
>PROJECT_SOURCE_DIR : /home/bate/Codes/course/11/template  
>  
>PROJECT_BINARY_DIR : /home/bate/Codes/course/11/template/build  
>  
>CMAKE_CURRENT_SOURCE_DIR : /home/bate/Codes/course/11/template  
>  
>CMAKE_CURRENT_BINARY_DIR : /home/bate/Codes/course/11/template/build  
>  
>mylib got PROJECT_NAME : mylib  
>  
>mylib got CMAKE_SOURCE_DIR : /home/bate/Codes/course/11/template  
>  
>mylib got CMAKE_BINARY_DIR : /home/bate/Codes/course/11/template/build  
>  
>mylib got PROJECT_SOURCE_DIR : /home/bate/Codes/course/11/template/mylib  
>  
>mylib got PROJECT_BINARY_DIR : /home/bate/Codes/course/11/build/mylib  
>  
>mylib got CMAKE_CURRENT_SOURCE_DIR : /home/bate/Codes/course/11/template/mylib  
>  
>mylib got CMAKE_CURRENT_BINARY_DIR : /home/bate/Codes/course/11/template/build/mylib
  
#### project 的初始化 : LANGUAGES 字段
  
* project( 项目名 LANGUAGES 使用的语言列表...) 指定了该项目使用了那种编程语言  
  目前支持的语言：  
  * C : C语言  
  * CXX : C++  
  * ASM : 汇编  
  * Fortran : 老年人的编程语言(雾)，IBM  
  * CUDA : 英伟达的黑科技 CUDA ( 需要 CMake 3.8 版本 )  
  * OBJC : 苹果的 Objective-C ( 需要 CMake 3.16 版本 )  
  * OBJCXX : 苹果的 Objective-C++ ( 需要 CMake 3.16 版本 )  
  * ISPC : 一种英特尔的自动 SIMD 编程语言 ( 需要 CMake 3.18 版本 )  
* 如果不指定 LANGUAGES, 默认为 C 和 CXX。
  
##### 常见问题：LANGUAGES 中没有启用 C 语言，但却用到了 C 语言
  
~~~cmake
cmake_minimum_required(VERSION 3.15)
project(hellocmake LANGUAGES CXX)

add_executable(main main.c)
~~~
  
~~~c
#include <stdio.h>

int main(void)
{
    printf("Hello world from Cmake!\n");
    return 0;
}
~~~
  
这样不行滴哥们，会报错，因为你在 CMakeLists.txt 的设置中没有启用 C 语言。  
解决办法：
  
~~~cmake
cmake_minimum_required(VERSiON 3.15)
project(hellocmake LANGUAGES C CXX)

add_executable(main main.c)
~~~
  
这次启用了 C 和 C++ 就不会报错力。
  
##### 也可以先设置 LANGUAGES NONE, 之后调用 enable_language(CXX)
  
这样可以把 enable_language 放到 if 语句中，从而只有某些选项开启才启用某语言类似的操作。
  
~~~cmake
cmake_minimum_required(VERSION 3.15)
project(hellocmake LANGUAGES NONE)
enable_language(CXX)

add_executable(main main.cpp)
~~~
  
#### 设置 C++ 标准：CMAKE_CXX_STANDARD 变量
  
* CMAKE_CXX_STANDARD 是一个整数，表示要用的 C++ 标准。  
  比如需要 C++17 即设为 17。  
* CMAKE_CXX_STANDARD_REQUIRED 是 BOOL 型，可以为 ON / OFF , 默认为 OFF。  
  表示是否一定要支持指定的 C++ 标准，如果为 OFF 则CMake 检测到编译器不支持 C++17 时不报错，而是将设置调整为 C++14 让开发人员使用；为 ON 时，不支持会报错，具有更好的安全性。  
* CMAKE_CXX_EXTENSIONS 是 BOOL 变量，默认为 ON。为 ON 表示启用 GCC 特有的一些拓展功能；OFF 则关闭 GCC 的拓展功能，只使用标准的 C++。  
  要兼容其他编译器( 如 MSVC )的项目都会将其设为 OFF，以防使用了 GCC 特有的特性。  
* 注意，最好在 project 命令前设置 CMAKE_CXX_STANDARD 一系列变量，这样一来 CMAKE 可以在 project 函数内对编译器进行一些检测，查看是否能支持对应版本C++的特性。
  
~~~cmake
cmake_minimum_required(VERSION 3.15)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

project(hellocmake LANGUAGES CXX)
~~~
  
##### 常见误区 : 小彭老师，我手动加 -std=c++17 行不得行
  
* 不要直接修改 CMAKE_CXX_FLAGS 来添加 -std=c++17  
  使用 CMake 封装好的 CMAKE_CXX_STANDARD  
  前者为什么不好，GCC 用户一旦手动指定 -std=c++17, 就是使用了 GCC 的特性，而 MSVC 的用户就无法使用了。   
  而且 CMake 已经自动根据 CMAKE_CXX_STANDARD 的默认值 11 添加了 -std=c++11，之后你再手动添加 -std=c++17 选项就发生了冲突。  
  所以一定要使用 CMake 已经封装好的 CMAKE_CXX_STANDARD !
  
### projec 的初始化：VERSION 字段
  
* project(项目名 VERSION x.y.z) 可以将当前项目的版本号设定为 x.y.z  
  之后可以使用 PROJECT_VERSION 获取当前项目的版本号  
* PROJECT_VERSION_MAJOR 获取 x ( 主版本号 )  
  PROJECT_VERSION_MINOR 获取 y ( 次版本号 )  
  PROJECT_VERSION_PATCH 获取 z ( 补丁版本号 )
  
#### 项目名的另一个作用 : 会设置另外 <项目名>_SOURCE_DIR 等变量
  
~~~cmake
cmake_minimum_required(VERSION 3.15)
project(hellocmake VERSION 2.7.1)

message("PROJECT_NAME: ${PROJECT_NAME}")
message("PROJECT_VERSION: ${PROJECT_VERSION}")
message("PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")
message("PROJECT_BINARY_DIR: ${PROJECT_BINARY_DIR}")
message("hellocmake_VERSION: ${hellocmake_VERSION}")
message("hellocmake_SOURCE_DIR: ${hellocmake_SOURCE_DIR}")
message("hellocmake_BINARY_DIR: ${hellocmake_BINARY_DIR}")
~~~
  
>PROJECT_NAME: hellocmake  
>PROJECT_VERSION: 2.7.1  
>PROJECT_SOURCE_DIR: /home/bate/Codes/course/11/templa  
>PROJECT_BINARY_DIR: /tmp/build/home/bate/Codes/course/11/templa  
>hellocmake_VERSION: 2.7.1  
>hellocmake_SOURCE_DIR: /home/bate/Codes/course/11/templa  
>hellocmake_BINARY_DIR: /tmp/build/home/bate/Codes/course/11/templa
  
这个功能可以让我们在当前项目去查询别的项目的版本号比如在 hellocmake 查询 helloworld 的版本号或者其他信息。
  
#### 小技巧 : CMake 的 ${} 表达式可以嵌套
  
因为 ${PROJECT_NAME} 的值是 hellocmake  
所以 ${${PROJECT_NAME}_VERSION} 相当于 ${hellocmake_VERSION} 即 2.7.1  
[CMake 其他关键字](https://blog.csdn.net/fuyajun01/article/details/8891749)
  
#### 一个标准的 CMakeLists.txt 模板
  
~~~cmake
cmake_minimum_required(VERSION 3.15)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(zeno LANGUAGES C CXX)

if(PROJECT_BINARY_DIR STREQUAL PROJECT_SOURCE_DIR)
	message(WARING "The binary directory of CMake cannot be the same as source directory!")
endif()

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release)
endif()

if(WIN32)
	add_definitions(-DNOMINMAX -D_USE_MATH_DEFINES)
endif()

if(NOT MSVC)
	find_program(CCACHE_PROGRAM ccache)
	if(CCACHE_PROGRAM)
		message(STATUS "Found CCache: ${CCACHE_PROGRAM}")
		set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ${CCACHE_PROGRAM})
		set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ${CCACHE_PROGRAM})
	endif()
endif()
~~~
  
## 第三章：链接库文件
  
### main.cpp 调用 mylib.cpp 里的 say_hello 函数
  
* CMakeLists.txt  
  ~~~cmake  
  add_executable(main main.cpp mylib.cpp)  
  ~~~
  
* main.cpp
  
  ~~~c++  
  #include "mylib.h"
  
  int main()  
  {  
  	say_hello();  
  }  
  ~~~
  
* mylib.cpp  
  ~~~cpp  
  #include "mylib.h"  
  #include <cstdio>
  
  void say_hello()  
  {  
      printf("Hello, mylib!\n");  
  }  
  ~~~
  
#### 改进：mylib 作为静态库
  
> [《静态库和动态库》](https://www.jianshu.com/p/090e1c0310ab) , [《CMake | 编译静态库、动态库和对象库》](https://blog.csdn.net/weixin_39766005/article/details/122368414)  
> 静态库会在*链接时*完整的复制到每一个可执行文件，被多次使用时就会造成多分冗余。  
> 动态库在*链接时*不复制，程序运行时由系统动态加载到内存中，供程序调用，系统仅需加载一次，多个程序公用，节省内存。
  
* CMakeLists.txt
  
  ~~~cmake  
  add_library(mylib STATIC mylib.cpp)
  
  add_executable(main main.cpp)
  
  target_link_libraries(main PUBLIC mylib)  
  ~~~
  
#### 改进：mylib 作为一个动态库
  
* CMakeLists.txt ( 动态库在Windows上有坑 )  
  ~~~cmake  
  add_library(mylib SHARED mylib.cpp)
  
  add_executable(main main.cpp)
  
  target_link_libraries(main PUBLIC mylib)  
  ~~~

  
#### 改进：mylib 作为一个对象库
  
对象库类似于静态库，但不生成 .a 文件，只由 CMake 记住该库生成了那些对象文件  
~~~cmake
add_libraty(mylib OBJECT mylib.cpp)

add_executable(main main.cpp)

target_link_libraries(main PUBLIC mylib)
~~~
  
对象库是 CMake 自创的，绕开了编译器和操作系统的各种繁琐规则，保证了跨平台统一性。  
在自己的项目中，推荐全部使用对象库( OBJECT ) 替代静态库 ( STATIC ) 避免跨平台的麻烦。  
对象库仅仅作为组织代码的方式，而实际生成的可执行文件只有一个，减轻了部署的困难。
  
#### 静态库的麻烦：GCC 编译器会自作聪明，将自动剔除没有引用符号的对象
  
* CMakeLists.txt  
  ~~~cmake  
  add_library(mylib STATIC mylib.cpp)  
  ~~~
  
* mylib.cpp  
  ~~~cpp  
  #include <cstdio>  
  // 静态初始化  
  static int unused = printf("My initialized\n");	// 会在主函数前被执行  
  ~~~
  
* main.cpp  
  ~~~c++  
  #include <cstdio>
  
  int main()  
  {  
      printf("Main function\n");  
  }  
  ~~~
  
* Output  
  >main function
  
这里就是 GCC 看到没有引用人 mylib ，就会删掉 mylib.o 但是恰恰遇到了静待初始化，GCC 就做错了。
  
#### 对象库就可以绕开编译器的不统一：保证不会自动剔除没有用到的对象文件
  
* CMakeLists.txt  
  ~~~cmake  
  add_library(mylib OBJECT mylib.cpp)  
  ~~~
  
* mylib.cpp
  
  ~~~c++  
  #include <cstdio>
  
  static int unused = printf("Mylib Function\n");  
  ~~~
  
* main.cpp  
  ~~~c++  
  #include <cstdio>
  
  int main()  
  {  
      printf("Main Function\n");  
  }  
  ~~~
  
* Output  
  >Mylib Function  
  >Main Function
  
#### add_library 无参数时，是静态库还是动态库
  
会根据 BUILD_SHARED_LIBS 这个变量的值决定是动态库还是静态库。  
ON 则相当于 SHARED, OFF 则相当于 STATIC  
如果未指定 BUILD_SHARED_LIBS 变量，则默认为 STATIC。  
因此，如果发现一个项目内的 add_library 都是无参的，意味着我们可以使用：  
`cmake -B build -DBUILD_SHARD_LIBS:BOOL=ON`  
来让他全部生成为动态库，这里涉及到*命令行传递变量的规则。*
  
~~~cmake
set(BUILD_SHARED_LIBS ON)

add_library(mylib mylib.cpp)
~~~
  
#### 小技巧：设定一个变量的默认值
  
要让 BUILD_SHARED_LIBS 默认为 ON，可以用之前类似的思路  
如果该变量没有定义，则设为 ON，否则保持用户指定的值不变  
这样用户没有指定 BUILD_SHARED_LIBS 时，会默认变成ON。  
只有用户指定 BUILD_SHARED_LIBS 为 OFF 即 `-DBUILD_SHARED_LIBS:BOOL:OFF`  
才会生成静态库，否则默认生成动态库。
  
~~~cmake
if(NOT DEFINED BUILD_SHARED_LIBS)
	set(BUILD_SHARED_LIBS ON)
endif()
~~~
  
#### 常见坑点：动态库无法链接静态库
  
~~~cmake
add_library(otherlib STATIC otherlib.cpp)

add_library(mylib SHARED mylib.cpp)
target_link_libraries(mylib PUBLIC otherlib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC mylib)
~~~
  
`target_link_libraries(mylib PUBLIC otherlib)`试图将静态库`otherlib`链接到`mylib`中，发生错误。*静态库`otherlib`误以为用户将其连接到一个可执行文件上*，但用户却连接到动态库上。动态库在内存中的地址会变化的，在编译时会指定一个`fPIC`选项，但是静态库没有`fPIC`选项，静态库的地址并不想变化，而动态库本身却想改变地址，二者会发生冲突。
  
##### 解决办法：
  
1、将`otherlib`变为对象库
  
~~~cmake
add_library(otherlib OBJECT otherlib.cpp)

add_library(mylib SHARED mylib.cpp)
target_link_libraries(mylib PUBLIC otherlib)

add_executable(main main.cpp)
target_link_libraties(main PUBLIC mylib)
~~~
  
2、让静态库编译也生成位置无关的代码( PIC )，这样才能装在动态库中
  
~~~cmake
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

add_library(otherlib STATIC otherlib.cpp)

add_library(mylib SHARED mylib.cpp)
target_link_libraries(mylib PUBLIC otherlib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC mylib)
~~~
  
但是这样处理会导致本来不需要为静态 PIC 的静态库也变成 PIC 了，**所以我们可以只针对一个库，只对他启用位置无关的代码( PIC )**  
~~~cmake
add_library(otherlib STATIC otherib.cpp)
set_property(TARGET otherlib PROPERTY POSITION_INDEPENDENT_CODE ON)

add_library(mylib SHARED mylib.cpp)
target_link_libraries(mylib PUBLIC otherlib)

add_executable(main main.cpp)
target_link_libraried(main PUBLIC mylib)
~~~
  
**注意，add_library() 是要指定头文件的，这里偷懒没加，指定头文件后其就会出现在IDE中。**


  
## 第四章 : 对象的属性
  
前面提到的 `POSITION_INDEPENDENT_CODE` 就是一个属性。  
### 除了 `POSITION_INDEPENDENT_CODE` 还有哪些属性呢？
  
~~~cmake
add_executable(main main.cpp)

set_property(TARGET main PROPERTY CXX_STANDARD 17)	# 采用 C++17 标准编译( 默认为11 )
set_property(TARGET main PROPERTY CXX_STANDARD_REQUIRED ON)	# 如果编译器不支持 C++17，则直接报错( 默认为 OFF )
set_property(TARGET main PROPERTY WIN32_EXECUTABLE ON)	# 在 Windows 系统中运行时不启动控制台窗口，只有 GUI 界面 (默认 OFF)
set_property(TARGET main PROPERTY LINK_WHAT_YOU_USE ON)	# 告诉编译器不要自动剔除没有引用符号的链接库(默认 OFF)
set_property(TARGET main PROPERTY LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)	# 设置动态链接库的输出路径(默认 ${CMAKE_BINARY_DIR})
set_property(TARGET main PROPERTY ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)	# 设置静态链接库的输出路径(默认 ${CMAKE_BINARY_DIR})
set_property(TARGET main PROPERTY RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)	# 设置可执行文件的输出路径(默认 ${CMAKE_BINARY_DIR})
~~~
  
这样一个一个 `set_property` 好麻烦啊！要是有更简单的写法就好了，于是……
  
### 另一个方法 : `set_target_properties` 批量设置多个属性
  
~~~cmake
add_executable(main main.cpp)

set_target_properties(main PROPERTIES
	CXX_STANDARD 17
	CXX_STANDARD_REQUIRED ON
	WIN32_EXECUTABLE ON
	LINK_WHAT_YOU_USE ON
	LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib
	ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib
	RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin
)
~~~
  
### 另一种方法：通过全局的变量，让之后创建的所有对象都享有同样的属性
  
相当于改变了各属性的默认初始值，要注意*此时* `set(CMAKE_xxx)` *必须在* `add_executable` *之前才有效*。  
~~~cmake
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_WIN32_EXECUTABLE ON)
set(CMAKE_LINK_WHAT_YOU_USE ON)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)

add_executbale(main main.cpp) # 这里需要放在后面
~~~
  
### 百度常见的错误！！！
  
对于 `CXX_STANDARD` 这种 CMake 本身提供了变量进行配置设置的，不要自己去设置 -std=c++17 选项，会和 CMake 自己设置好的产生冲突，导致出错！  
请始终使用 `CXX_STANDARD` 或者全局变量 `CMAKE_CXX_STANDARD` 来设置 -std=c++17 这个 flag，CMake 会在配置阶段进行编译器检测是否支持 C++17。  
CUDA 的 -arch=sm_75 也是同样的道理，请使用 `CUDA_ARCHITECTURES` 属性。  
再者说 -std=c++17 只是 GCC 编译器的选项，也不能进行跨平台适用于 MSVC 编译器啊！！！
  
~~~cmake
add_executable(main main.cpp)

set_property(TARGET main PROPERTY CXX_STANDARD 17)	# 正确
target_compile_options(main PUBLIC "-std=c++17")	# 错误！！！！
set_property(TARGET main PROPERTY CUDA_ARCHITECTURES 75)	# 正确
target_compile_options(main PUBLIC "-arch=sm_75")	# 错误！！！
~~~
  
### 假如在 Windows 使用动态链接库，需要额外操作
  
`m/mylib.cpp`
  
~~~c++
#include <cstdio>
// 需要手动加入这几句，在实现处加入dllexport
#ifdef _MSC_VER
__declspec(dllexport)
#endif
void say_hello(){
    printf("Hello, world!\n");
}
~~~
  
`m/mylib.h`
  
~~~c++
#pragma once
// 在声明处，加入import
#ifdef _MSC_VER
__declspec(dllimport)
#endif
void say_hello();
~~~
  
根目录下的 `CMakeLists.txt`  
~~~cmake
cmake_minimum_required(VERSION 3.15)

add_subdirectory(mylib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC mylib)
~~~
  
子目录`m`下的`CMakeLists.txt`, `m/CMakeLists.txt`  
~~~cmake
add_library(mylib SHARED mylib.cpp mylib.h)
~~~
  
#### 常见问题：链接了自己的 dll，但是运行时会找不到
  
* 这是因为 dll 和 exe 不在用一个目录，而愚蠢的 Windows 只会在**当前 exe 所在目录查找**，**然后查找 *PATH环境变量***，找不到就报错。而 dll 在其他目录，因此 Windows 找不到。  
  * **解决办法1**：把 dll 所在位置加到 *PATH环境变量* 里，一劳永逸。  
  * **结局办法2**：把这个 dll，以及这个 dll 依赖的其他所有 dll，全部拷贝到和 exe 文件同一目录下。
  
#### 手动拷贝 dll 好麻烦，CMake 能不能救一下！把 dll 自动生成在 exe 同一目录下
  
* 说到底还是因为 CMake 把定义在顶层模块里的 main 放在 `build/main.exe`  
  而 mylib 因为是定义在 mylib 这个子模块里的，因此被放到了 `build/mylib/mylib.dll`
  
##### 解决1：设置 mylib 对象的 xx_OUTPUT_DEIRECTORY 系列属性
  
* 所以，可以设置 mylib 的这些属性，让 mylib.dll 文件输出到 PROJECT_BINARY_DIR，也就是项目根目录( main 所在的位置 )，这样 main.exe 在运行时就能找到 mylib.dll
  
* 为了侍奉 Windows，要设置全部的 6个属性！很烦！  
  `m/CMakeLists.txt`
  
  ~~~cmake  
  add_library(mylib SHARED mylib.cpp mylib.h)
  
  set_property(TARGET mylib PROPERTY RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})  
  set_property(TARGET mylib PROPERTY ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})  
  set_property(TARGET mylib PROPERTY LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})  
  set_property(TARGET mylib PROPERTY RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR})  
  set_property(TARGET mylib PROPERTY ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR})  
  set_property(TARGET mylib PROPERTY LIBRARY_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR})  
  set_property(TARGET mylib PROPERTY RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR})  
  set_property(TARGET mylib PROPERTY ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR})  
  set_property(TARGET mylib PROPERTY LIBRARY_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR})  
  ~~~
  
  这样就会输出到项目根目录 build 目录下
  
##### 而在 Linux 系统下就显得简便了
  
* Linux 系统支持 RPATH，CMake 会让生成出来可执行文件的 RPATH 指向他链接了的 .so 文件所在目录，运行时会优先从 RPATH 里找链接库，所以即使不在同目录也能找到。  
  所以**第三种解决办法**，卸载 Windows 安装 Linux。  
* 需要手动修改或者查看一个 ELF 文件的 RPATH，可以用 chrpath 或者 pathchelf 命令。


  
## 第五章：连接第三方库
  
### 例子：需要使用 tbb 库
  
* CMakeLists.txt  
  ~~~cmake  
  add_executable(main main.cpp)  
  target_link_libraries(main PUBLIC tbb)	# Linux 上直接链接 tbb 是可以的，但是 Windows 可能不行。  
  ~~~
  
* main.cpp  
  ~~~c++  
  #include <tbb/parallel_for.h>
  
  int main(){  
  	tbb::parallel_for(0,4,[&](int i){  
          printf("Hello, %d!\n", i);  
      });  
  }  
  ~~~
  
* OutPut  
  >Hello, 0!  
  >Hello, 1!  
  >Hello, 2!  
  >Hello, 3!
  
#### 直接链接 tbb 的缺点
  
Linux 可以直接链接，是因为其有默认的库目录 `usr/lib` ，但是 Windows 没有一个固定的库安装位置。Linux 因为 `usr/lib/`, Linux 可以找到 `usr/lib/libtbb.so`  
如果这样直接指定 tbb，CMake 会让连接器在系统的库目录里查找 tbb，他会找到 `usr/lib/libtbb.so` 这个系统自带的，但是对于没有一个固定库安装位置的 Windows 系统并不适用。  
此外，他还要求 tbb 的头文件就在 `usr/include` 这个系统默认的头文件目录，  
这样才能 `#include <tbb/parallel_for.h` 不报错，如果 tbb 的头文件在其他地方  
就需要再加一个 `target_include_directories` 设置额外的头文件查找目录。
  
* CMakeLists.txt  
  ~~~cmake  
  add_executable(main main.cpp)  
  target_link_libraries(main PUBLIC tbb)  
  ~~~
  
#### Windows 可以直接写出全部的绝对路径，十分的硬核
  
也可以直接写出全部的路径，这样就能让没有默认系统路径的 Windows 系统找到安装在不知何处的 tbb，不过这样就不能跨平台了，如果其他人安装在不同位置就会发生错误。  
**顺便一提，CMake 的路径分隔符始终是`/`。即使在 Windows 上，也要把所有的 `\` 改成 `/`**，这是为了跨平台考量。请放心，CMake 会自动在调用 MSVC时转换成 `\` ，可以放心的用 `${x}/bin` 来实现和 Python 的 [`os.path.join(x, 'bin')`](https://blog.csdn.net/swan777/article/details/89040802) 一样的效果。
  
* CMakeLists.txt  
  ~~~cmake  
  add_executable(main main.cpp)  
  target_link_libraries(main PUBLIC C:/User/archibate/installed/tbb/tbb.dll)  
  ~~~
  
  > 大多数操作系统都是 Unix-like，只有 Windows 搞特殊。  
  > `cd /d C:\\Program\ Files\\(x86\)\\Micsoft\ Visual\ Studio\\2019\\`  
  > 在路径中动不动就放一堆转移符、空格、特殊符号  
  > 高情商：Windows 是最适合练习 C 语言转移符使用水平的平台！
  
#### 终于！`find_package`
  
更通用的方式：find_package  
更好的办法就是使用 CMake 的 `find_package` 命令。  
`find_package(TBB REQUIRED)` 会查找 `/usr/lib/cmake/TBB/TBBConfig.cmake` 这个配置文件，根据里面的配置信息创建 `TBB::tbb` 这个伪对象( 实际它指向真正的 tbb 库文件路径 `usr/lib/libtbb.so` )，之后通过 `target_link_libraries` 链接 `TBB::tbb` 就可以正常工作。
  
* CMakeLists.txt  
  ~~~cmake  
  add_executbale(main main.cpp)
  
  find_package(TBB REQUIRED)  
  target_link_libraries(main PUBLIC TBB::tbb)	# TBB 包下的 tbb 库  
  ~~~
  
##### `TBB::tbb` 的秘密：自带了一些 PUBLIC 属性
  
`TBB::tbb` 是一个伪对象( imported )，他除了会指向 `/usr/lib/libtbb.so`，TBBConfig.cmake 还会给 TBB::tbb 添加一些 PUBLIC 属性，用于让链接了他的对象带上一些 flag 之类的。  
比如，TBB 安装在 `/opt/tbb` 目录下，头文件在 `/opt/tbb/include` 里，那么这时 TBBConfig.cmake 里就会有 : `target_include_directories(TBB::tbb PUBLIC /opt/tbb/include)`  
这样 main 在链接了 TBB::tbb 时也会被“传染”上 `/opt/tbb/include` 这个目录，无需手动添加。  
再比如，TBB::tbb 链接了另一个库 Blosc::blosc，那么这个库也会自动连接到 main 上，无需调用者手动添加。
  
> 比如 spdlog 的 spdlog-config.cmake 就会定义 SPDLOG_NOT_HEADER_ONLY 这个宏为 PUBLIC 。从而实现直接 #include <spdlog/spdlog.h> 时候时纯头文件，而 find_package(spdlog REQUIRED) 时却变成预编译链接库的版本。( 其实不是 PUBLIC 而是 INTERFACE，因为伪对象没有实体 )
  
##### 和 `find_package(TBB CONFIG REQUIRED)` 有什么区别
  
其实更好的是通过 `find_package(TBB CONFIG REQUIRED)`，添加一个 CONFIG 选项。  
这样他会优先查找 TBBConfig.cmake ( 系统自动的 ) 而不是 FindTBB.cmake ( 项目作者常把他塞在 cmake/ 目录里并添加到 CMAKE_MODULE_PATH )。这样能保证寻找包的这个 .cmake 脚本是和系统自带的 tbb 版本是适配的，而不是项目作者当年下载的那个版本的 .cmake 脚本。
  
* CMakeLists.txt  
  ~~~cmake  
  add_executable(main main.cpp)
  
  find_package(TBB CONFIG REQUIRED)  
  target_link_libraries(main PUBLIC TBB::tbb)  
  ~~~
  
当然如果坚持要用 find_package(TBB REQUIRED) 也是可以的。  
没有 CONFIG 选项：先找 FindTBB.cmake，再找 TBBConfig.cmake，找不到就报错。  
有 CONFIG 选项：只会找 TBBConfig.cmake，找不到则报错。  
此外有一些老年项目( 比如 OpenVDB ) 只提供 Find 而没有 Config 文件，这时候只能用 find_package(OpenVDB REQUIRED) 而不能带 CONFIG 选项。
  
#### /usr/lib/cmake/TBB/TBBConfig.cmake 长什么样？
  
不论 TBBConfig.cmake 还是 FindTBB.cmake，这个文件通常由库的作者提供，在 Linux 的包管理器安装 tbb 后也会自动安装这个文件。少部分对 CMake 不友好的第三方库，需要自己写 FindXXX.cmake 才能使用。
  
* TBBConfig.cmake  
  ~~~cmake  
  # Create imported target TBB::tbb  
  add_library(TBB::tbb SHARED IMPORTED)
  
  set_target_properties(TBB::tbb PROPERTIES  
  	INTERFACE_COMPILE_DEFINITIONS "\$<\$<CONFIG:DEBUG>:TBB_USE_DEBUG>"  
  	INTERFACE_INCLUDE_DIRECTORIES "${_IMPORT_PREFIX}/include"  
  )
  
  # Create imported target TBB::tbbmalloc  
  add_library(TBB::tbbmalloc SHARED IMPORTED)
  
  set_target_proerties(TBB::tbbmalloc PROPERTIES  
  	INTERFACE_COMPILE_DEFINITIONS "\$<\$<CONFIG:DEBUG>:TBB_USE_DEBUG>"  
  	INTERFACE_INCLUDE_DIRECTORIES "${_IMPORT_PREFIX}/include"  
  )
  
  # Create imported target TBB::tbbmalloc_proxy  
  add_library(TBB::tbbmalloc_proxy SHARED IMPORTED)
  
  set_target_properties(TBB::tbbmalloc_proxy PROPERTIES  
  	INTERFACE_COMPILE_DEFINITIONS "\$<\$<CONFIG:DEBUG>:TBB_USE_DEBUG>"  
  	INTERFACE_INCLUDE_DIRECTORIES "${_IMPORT_PREFIX}/include"  
  )  
  ~~~
  
  
#### find_package(Qt5 REQUIRED) 出错
  
~~~cmake
find_package(Qt5 REQUIRED)
target_link_libraries(main PUBLIC Qt5::Widgets Qt5::Gui)
~~~
  
* **ERROR**  
  <img src="/imgs/CMake Note/Qt5Error.png" alt="Qt5报错">  
  这里是说( 看最后一句 ) Qt5 包至少需要一个组件。  
  Qt5 有很多组件，但是直接 `find_package(Qt5 REQUIRED)` 他不知道用户需要哪些组件。
  
##### 原因：Qt5 具有多个组件，你必须指定你需要哪些组件
  
find_package 生成的伪对象 (imported target) 都按照 “包名::组件名” 的格式命名。  
可以在 find_package 中通过 **COMPONENTS** 选项，后面跟随一个列表表示需要用的组件。
  
~~~cmake
find_package(Qt5 COMPONENTS Widgets Gui REQUIRED)
target_link_libraries(main PUBLIC Qt5::Widgets Qt5::Gui)
~~~
  
~~~cmake
find_package(TBB COMPONENTS tbb tbbmalloc tbbmalloc_proxy REQUIRED)
target_link_libraries(main PUBLIC TBB::tbb TBB::tbbmalloc TBB::tbbmalloc_proxy)
~~~
  
##### 常见错误：Windows 找不到 Qt5
  
因为 Windows 系统安装路径混乱没有固定的 /usr/lib 之类的默认路径能供CMake搜索所以报错了。  
<img src="/imgs/CMake Note/Qt5CannotFound.png" alt="Qt5找不到">
  
* 假设 Qt5 安装在 C:\Qt\Qt5.14.2，去找这个目录  
  C:\Qt\Qt5.14.2\msvc2019_64\lib\cmake\
  
* 会有一个 Qt5Config.cmake，现在有四种办法可以让CMake找到他
  
  * **第一种**：设置 CMAKE_MODULE_PATH 变量，添加一下包含 Qt5Config.cmake 这个文件的目录路径 C:\Qt\Qt5.14.2\msvc2019_64\lib\cmake，当然这里也要把 `\` 换成 `/`，因为 CMake 是倾向 Unix 的构建，*<del>这是派别和历史问题了</del>*。这种方法相当于在 CMake 搜索目录里加上这个路径，其他包在搜索遍历时也会遍历过这个路径，后面会有更好的办法。
  
    ~~~cmake  
    set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} C:/Qt/Qt5.14.2/msvc2019_64/lib/cmake)
    
    find_package(Qt5 REQUIRED COMPONENTS Widgets Gui REQUIRED)  
    target_link_libraries(main PUBLIC Qt5::Widgets Qt5::Gui)  
    ~~~
  
  * **第二种(更好的办法)：设置<包名>_DIR 变量指向 <包名>Config.cmake 所在位置**  
    设置 Qt5_DIR 这个变量为 C:\Qt\Qt5.14.2\msvc2019_64\lib\cmake  
    这样只有 Qt5 这个包会去这个目录里搜索 Qt5Config.cmake 更有针对性。
  
    ~~~cmake  
    set(Qt5_Dir C:/Qt/Qt5.14.2/msvc2019_64/lib/cmake)
    
    find_package(Qt5 REQUIRED COMPONENTS Widgets Gui REQUIRED)  
    target_link_libraries(main PUBLIC Qt5::Widgets Qt5::Gui)  
    ~~~

  
  * 第三种(推荐)，直接在命令行通过 `-DQt5_DIR="xxx"` 指定，这样不用修改 CMakeLists.txt  
    `cmake -B build -DQt5_DIR="C:/Qt/Qt5.14.2/msvc2019_64/lib/cmake"`  
  * 第四种，还可以设置环境变量 Qt5_DIR 也是可以的，就是对 Windows 用户比较困难  
    `export Qt5_DIR="/opt/Qt5.14.2/lib/cmake"`
  
### 不指定 REQUIRED 找不到时不报错，只会设置 TBB_FOUND 为 FALSE
  
 ~~~cmake  
 find_package(TBB)  
 if(TBB_FOUND)  
 	message(STATUS "TBB found at:${TBB_DIR}")  
 	target_link_libraries(main PUBLIC TBB::tbb)  
 	target_compile_definitions(main PUBLIC WITH_TBB)  
 else()  
 	message(WARNING "TBB not found! using serial for")  
 endif()  
 ~~~
  
前面很多例子都在 find_package() 加上 REQUIRED 选项。  
如果我们不加 REQUIRED，在找不到对应 package 时不会报错。  
这样的设计目的在于，当我们添加一些可选的依赖，如果没有也不会影响程序基本运行，我们找不到可选项，就可以向用户抛出一个警告。  
找到了会把 TBB_FOUND 设为 TRUE，TBB_DIR 也会设置为 TBBConfig.cmake 所在目录。  
找不到会把 TBB_FOUND 设为 FASLE，TBB_DIR 也会为空。  
这里我们在找到 TBB 的 if 里定义一个 WITH_TBB 宏，稍后在 .cpp 里就可以根据这个判断。  
如果找不到 TBB 可以 fallback 到保守的实现方式。  
`-- TBB found at: /usr/lib64/cmake/TBB`
  
#### 在 C++ 中判断 WITH_TBB 宏，找不到 TBB 则退化到串行 for 循环
  
~~~c++
#include <cstdio>
#ifdef WITH_TBB
#include <tbb/parallel_for.h>
#endif

int main()
{
#ifdef WITH_TBB
	tbb::parallel_for(0,4,[&](int i){
#else
        for(int i = 0; i < 4; i++){
#endif
			printf("hello, %d!\n",i);
#ifdef WITH_TBB
    });
#else
    }
#endif
}
~~~
  
### 也可以使用 TARGET 判断是否存在 TBB::tbb 这个伪对象，实现 TBB_FOUND 的效果
  
~~~cmake
find_package(TBB)
if(TARGET TBB::tbb)
	message(STATUS "TBB found at:${TBB_DIR}")
	target_link_libraries(main PUBLIC TBB::tbb)
	target_compile_definitions(main PUBLIC WITH_TBB)
else()
	message(WARNING "TBB not found! using serial for")
endif()
~~~
  
同时也可以在 if 进行复合语句判断  
`NOT TARGET TBB::tbb AND TARGET Eigen3::eigen`  
表示找得到 TBB 但是找不到 Eigen3。
  
## 第六章：输出与变量
  
### 在运行 cmake -B build 时，打印字符串用于调试程序 
  
~~~cmake
message("Hello World")
~~~
  
>Hello world
  
message 会把字符串打在命令行。
  
#### message(STATUS "...")
  
~~~cmake
message(STATUS "Hello World")
~~~
  
>-- Hello world
  
不带 `STATUS` 选项，cmake 认为：“哦，你的需求很紧急，你只想调试程序。”，被认为是调试信息。  
带上 `STATUS` 表示是状态信息，告诉用户做了这件事。
  
#### message(WARING "...") 表示警告信息
  
~~~cmake
message(STATUS "Hello world")
message(WARNING "This is a warning sign")
~~~
  
>CMake Warning at CMakeLists.txt:2 (message):  
>  This is a warning sign!
  
#### message(AUTHOR_WARNING "...") 表示仅仅是给项目作者看的警告
  
~~~cmake
message(STATUS "Hello world")
message(AUTHOR_WARNING "Hollow Knight is the best!")
~~~
  
>-- Hello world  
>CMake Warning (dev) at CMakeLists.txt:2 (message):  
>  Hollow Knight is the best!  
>This warning is for project developers. Use -Wno-dev suppress it.
  
**AUTHOR_WARNING 可以通过 -Wno-dev 关闭**
  
`cmake -B build -Wno-dev`
  
#### message(FATAL_ERROR "...") 表示是错误信息，会终止 CMake 的运行
  
~~~cmake
message(STATUS "Hello world")
message(FATAL_ERROR "This is an error message!")
message(STATUS "After Error")
~~~
  
>-- Hello world  
>CMake Error at CMakeLists.txt:2 (message):  
>  This is an error message!
  
因为程序被中断所以 Hello world 被打印，Error 有执行，但是 After Error 没有执行，因为在 Error message 处被中断运行了。
  
#### message(SEND_ERROR "...") 表示错误信息，但之后的语句仍继续执行
  
~~~cmake
message(STATUS "Hello world")
message(SEND_ERROR "This is an error message!")
message(STATUS "After Error")
~~~
  
>-- Hello world  
>CMake Error at CMakeLists.txt:2 (message) :  
>  This is an error message  
>  
>After Error
  
可以看到 After Error 正常运行。
  
### message 可以用于打印变量
  
~~~cmake
set(myvar "Hello world")
message("myvar is: ${myvar}")
~~~
  
>myvar is: Hello world  
>-- Configuring done  
>-- Generating done
  
#### 如果 set 没加引号会怎么样？
  
**会变成分号分隔的列表。**  
这时候 set(myvar Hello world) 等价于 set(myvar "Hello;world")
  
~~~cmake
set(myvar Hello world)
message("myvar is: ${myvar}")
~~~
  
>myvar is: Hello;world  
>-- Configuring done  
>-- Generating done
  
#### 如果 message 没加引号会怎么样？
  
**会把列表里的字符串当作他的关键字**  
结论：除非实在实在需要列表，不然建议始终在你不确定的地方加上引号，例如：  
set(source "main.cpp" "mylib.cpp" "C:/Program Files/a.cpp")  
message("${source}")  
这里的 `C:/Program Files/a.cpp` 有空格所以最好使用引号，这也是为什么CMake用 `;` 做分割，因为 ` ` 可能出现在路径中，而 `;` 不会出现在文件路径中。
  
~~~cmake
set(myvar FATAL_ERROR hello)
message(${myvar})
# 这样${myvar} 会变成 FATAL_ERROR hello
~~~
  
>CMake Error at CMakeLists.txt:2 (message) :  
>  hello
  
## 第七章：变量与缓存(最大坑点)
  
### 重复执行 cmake -B build 会有什么区别
  
假如多次执行后几次执行会比第一次执行输出少很多。  
这是因为 CMake 第一遍需要检测编译器和 C++ 特性等比较耗时，检测完后会把结果存储的**缓存**中，这样第二遍运行cmake -B build 时就可以直接使用缓存的值，就不需要在检测一遍了。
  
### 如何清理缓存？删 build 大法
  
虽然 CMake 缓存为了加快编译的出发点是好的，但是有问题。比如有时候外部的情况有所更新比如原来的编译器被卸载或者更改，这时候 CMake 缓存内却还存储的是旧的值，就会导致一些问题。  
最简单的解决办法就是删除 build 文件夹，然后重新运行 cmake -B build。  
缓存是很多 CMake 出错的根源，因此如果出现诡异的错误，可以先试着删除 build 重新构建看看。  
删除了缓存就会重新跑一边检测，写新缓存。  
所以有了经典的 CMake 笑话：
  
>99%的 cmake 错误都可以用删 build 解决  
>删 build 大法好  
>rm -rf build
  
### 清除缓存，其实只需删除 build/CMakeCache.txt 就可以了
  
删除 build 虽然简单粗暴彻底，但是这会导致编译产生的中间结果 (.o文件) 也都被删除了，重新编译需要花费很长时间。  
如果只想清除缓存，不想从头重新编译，可以只删除 build/CMakeCache.txt 这个文件。  
它存储了缓存的变量，删除它就可以让 CMake 强制重新检测一遍所有的库和编译器。  
但是有些错误是中间文件的问题还是要删除 build  
### find_package 就用到了缓存机制
  
变量缓存的意义在于能把 find_package 找到的库文件位置等信息，存储起来。  
这下下次 find_package 时就会利用上缓存的变量，直接返回。  
避免重复执行 cmake -B 时速度变慢的问题。
  
但是会有这样一个问题，假如第一次 build，没有 TBB 库  
然后我们安装 TBB 后，再次进行build  
这时候 CMake 读了缓存发现：“哦，我之前找过 TBB，没找到，那不用找了没有 TBB 库”  
这时候 CMake 就不会去找了……  
这时候就要 rm -rf build/CMakeCache.txt，删除缓存文件让 CMake 强行重新配置。
  
### 设置缓存变量
  
语法是：`set(变量名 "变量值" CACHE 变量类型 "注释")`
  
~~~cmake
set(myvar "hello" CACHE STRING "this is the docstring.")
message("myvar is: ${myvar}")
~~~
  
**缓存的 myvar 会出现在 build/CMakeCache.txt 里。**
  
### 常见问题：我修改了 CMakeLists.txt 里 set 的值，却没有更新。
  
为了更新缓存变量，经常有人偷懒直接修改 CMakeLists.txt 里的值，这是没用的。  
因为 set(...CACHE..) 在缓存变量已经存在的时候，不会去更新缓存的值。  
CMakeLists.txt 里 set 的值被认为是“默认值”，因此不会在第二次 set 的时候更新。  
比如上面我们设置过 myvar 的内容是 hello，然后 build，现在改成 world，但是 CMakeCaChe.txt 内的 myvar 还会是 hello
  
~~~cmake
set(myvar "world" CACHE STRING "This is the docstring!")
message("myvar is: ${myvar}")
~~~
  
### 缓存变量如何更新？标准解法：通过命令行 -D参数
  
当然可以使用 ”删 build 大法“，但是有更好的。  
更新缓存变量正确的方式，通过命令行参数：`cmake -B build -Dmyvar=world`
  
#### 命令行 -D 太硬核了，有没有图形化的缓存编译器？
  
* 在 Linux 中，可以运行 ccmake -B build 启动基于终端的可视化缓存编辑菜单。  
* 在 Windows 中，可以 cmake-gui -B build 来启动图形界面编辑各个缓存选项。  
* 当然，可以直接用编辑器打开 build/CMakeCaChe.txt 修改后保存。  
  CMakeCaChe.txt 用文本文件存储的目的就是可供用户手动编辑，或者被第三方软件打开并解析的。
  
### 也可以通过指定 FORCE 来强制 set 更新缓存
  
set 可以在后面加一个 FORCE 选项，表示无论缓存存在与否，都强制更新缓存。  
不过这样就会导致没办法用 -Dmyvar=othervalue 来更新缓存变量。
  
~~~cmake
set(myvar "world" CACHE STRING "This is the docstring" FORCE)
message("mtvar is: ${myvar}")
~~~
  
### 缓存变量除了 STRING 还有哪些类型？
  
* STRING 字符串，比如："hello","world"  
* FILEPATH 文件路径，例如："C:/vcpkg/scripts/buildsystems/vcpkg.cmake"  
* PATH 目录路径，例如："C:/Qt/Qt5.14.2/msvc2019_64/cmake/"  
* BOOL 布尔值，只有两个取值：ON 或 OFF  
  注意：TRUE 和 ON 等价，FALSE 和 OFF 等价；YES 和 ON 等价，NO 和 OFF 等价。
  
#### 添加一个 BOOL 类型变量，用于控制是否启用某些特性
  
~~~cmake
add_executable(main main.cpp)

set(WITH_TBB ON CACHR BOOL "set to ON to enable TBB, OFF to disable TBB.")

if(WITH_TBB)
	target_compile_definitions(main PUBLIC WITH_TBB)
	find_package(TBB REQUIRED)
	target_link_libraries(main PUBLIC TBB::tbb)
endif()
~~~
  
这里就是用 CACHE变量 控制是否启用 TBB。
  
#### CMake 对 BOOL 类型缓存的 set 提供了简写：option
  
`option(变量名 "描述" 变量值)`  
等价于  
`set(变量名 CACHE BOOL 变量值 "描述")`
  
~~~cmake
add_executable(main main.cpp)

option(WITH_TBB "set to ON to enable TBB, OFF to disable TBB.")
if(WITH_TBB)
	target_compile_definitions(main PUBLIC WITH_TBB)
	find_package(TBB REQUIRED)
	target_link_libraries(main PUBLIC TBB::tbb)
endif()
~~~
  
option 的本质还是 set CACHE  
所以还会有和 set CACHE 一样的问题：option 设置成了 OFF，但是还是 ON。
  
#### 常见问题：在 CMakeLists.txt 里修改了 option 为 OFF，但是运行出来还是 ON
  
因为 option 本质就是 set CACHE，  
虽然修改了，但是 CMakeCaChe.txt 内还是 ON
  
#### 解决办法还是 -D参数 来修改
  
-D变量名:BOOL=ON/OFF  
`cmake -B build -DWITH_TBB:BOOL=OFF`
  
#### 或者改用 set 然后 Force
  
#### 绕靠缓存：使用普通变量，但仅当没有定义时设定为默认值
  
一般而言，CMake 自带的变量( 如 CMAKE_BUILD_TYPE )都这样设置。  
**这样项目的使用者还是可以用 -D来指示参数，**只不过不会在 ccmake 里被显示。(ccmake是查询缓存的)
  
~~~cmake
if(NOT DEFINED WITH_TBB)
	set(WITH_TBB ON)
endif()
message("WITH_TBB: ${WITH_TBB}")
if(WITH_TBB)
	target_compile_definition(main PUBLIC WITH_TBB)
	find_package(TBB REQUIRED)
	target_link_libraries(main PUBLIC TBB::tbb)
endif()
~~~
  
## 第八章：跨平台和编译器
  
### 在CMake 中给 .cpp 定义一个宏
  
~~~cpp
#include <cstdio>

int main()
{
#ifdef MY_MACRO
    printf("MY_MACRO defined! value: %d\n", MY_MACRO);
#else
    printf("MY_MACRO not defined!\n");
#endif
}
~~~
  
~~~cmake
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})
target_compile_definitions(main PUBLIC MY_MACRO=233)	# 相当于 gcc -DMY_MACRO=233
~~~
  
> MY_MACRO defined! value: 233
  
### 根据不同的操作系统，把宏定义为不同的值
  
~~~c++
#include <cstdio>

int main()
{
#ifdef MY_NAME
    printf("Hello, %s!\n", MY_NAME);
#else
    printf("I don`t know your name!\n");
#endif
}
~~~
  
~~~cmake
if(CMAKE_SYSTEM_NAME MATCHES "Windows")
	target_compile_definitions(main PUBLIC MY_NAME="Bill Gates")
elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")
	target_compile_definitions(main PUBLIC MY_NAME="Linus Torvalds")
elseif(CMAKE_SYSTEM_NAME MATCHES "Darwin")
	target_compile_definitions(main PUBLIC MY_NAME="Steve Jobs")
elseif()
~~~
  
### CMake 提供了一些简写变量：WIN32, APPLE, UNIX, ANDROID, IOS等等
  
虽然是 WIN32 但是对 32位和 64位Windows一样适用  
APPLE 对所有苹果产品 MacOS/IOS 都为真  
UNIX 对所有 Unix 类系统( FreeBSD, Linux, Android, MacOS, IOS ) 都为真
  
~~~cmake
if(WIN32)	# WIN32 这些是 bool 类型
	target_compile_definitions(main PUBLIC MY_NAME="Bill Gates")
elseif(UNIX AND NOT APPLE)
	target_compile_definitions(main PUBLIC MY_NAME="Linus Torvalds")
elseif(APPLE)
	target_compile_definitions(main PUBLIC MY_NAME="Steve Jobs")
elseif()
~~~
  
### 使用生成器表达式，简化指令
  
语法：`$<$<类型:值>:为真时表达式>`  
比如`$<$<PLATFORM_ID:Windows>:MY_NAME="Bill Gates">`  
在 Windows 平台上还会变成 MY_NAME="Bill Gates"  
其他平台则为空字符。
  
~~~cmake
target_compile_definitions(main PUBLIC
	$<$<PLATFROM_ID:Windows>:MY_NAME="Bill Gates">
	$<$<PLATFROM_ID:Linux>:MY_NAME="Linus Torvalds">
	$<$<PLATFROM_ID:Darwin>:MY_NAME="Steve Jobs">
)
~~~
  
### 生成器表达式：如需多个平台可以用逗号分隔
  
~~~cmake
target_compile_definitions(main PUBLIC
	$<$<PLATFORM_ID:Windows>:MY_NAME="DOS-like">
	$<$<PLATFROM_ID:Linux,Darwin,FreeBSD>:MY_NAME="Unix-like">
)
~~~
  
相关参考：[CMake : PLATFROM_ID](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#genex:PLATFROM_ID)
  
### 判断当前是哪一款 C++ 编译器
  
~~~cmake
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
	target_compile_definitions(main PUBLIC MY_NAME="gcc")
elseif(CMAKE_CXX_COMPILER_ID MATCHES "NVIDIA")
	target_compile_definitions(main PUBLIC MY_NAME="nvcc")
elseif(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
	target_compile_difinitions(mian PUBLIC MY_NAME="clang")
elseif(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
	target_compile_definitions(main PUBLIC MY_NAME="msvc")
endif()
~~~
  
### 也同样可以使用生成器表达式
  
~~~cmake
target_compile_definitions(main PUBLIC
	$<$<CXX_COMPILER_ID:GNU,Clang>:MY_NAME="Open-source">
	$<$<CXX_COMPILER_ID:MSVC,NVIDIA>:MY_NAME="Commercial">
)
~~~
  
### 生成器表达式可以做复杂的逻辑判断
  
~~~cmake
target_compile_definitions(main PUBLIC
	$<$<AND:$<CXX_COMPILER_ID:GNU,Clang>,$<PLATFROM_ID:Linux,FreeBSD>>:MY_NAME="Open-source">
)
~~~
  
### CMake 还提供了一些简写变量：MSVC, CMAKE_COMPILER_IS_GNUCC
  
~~~cmake
if(MSVC)
	target_compile_definitions(main PUBLIC MY_NAME="MSVC")
elseif(CMAKE_COMPILER_IS_GNUCC)
	target_compile_definitions(main PUBLIC MY_NAME="GCC")
else()
	target_compile_definitions(main PUBLIC MY_NAME="Other compiler")
endif()
~~~
  
### CMAKE_CXX_COMPILER_ID 直接作为字符串变量
  
~~~cmake
target_compile_definitions(main PUBLIC MY_NAME="The ${CMAKE_CXX_COMPILER_ID} Compiler")
~~~
  
### 从命令行参数指定编译器
  
`cmake -B build -DCMAKE_CXX_COMPILER="/usr/bin/clang++"`  
当然这个得第一次就定义，如果第二次使用这个命令需要 删build 清除缓存。
  
### 也可以通过环境变量 CXX 指定
  
`CXX='which clang' cmake -B build`
  
### CMAKE_GENERATOR 也可以了解一下
  
~~~cmake
message("Generator:${CMAKE_GENERATOR}")
message("C++ compiler: ${CMAKE_CXX_COMPILER}")
message("C compiler: ${CMAKE_C_COMPILER}")
~~~
  
## 第九章：分支和判断
  
### BOOL 类型的值
  
* 通常来讲只有 ON/OFF 两个取值  
  但是由于历史问题，TRUE/FALSE 和 YES/NO 也可以表示 BOOL 类型  
* 但是推荐只使用 ON/OFF 避免混淆
  
### if 的特点：不需要加 ${}, 会自动尝试作为变量名求值
  
由于历史问题，if 的括号中有着特殊的语法，如果是一个字符串，比如 MYVAR，则他会先看是否有 ${MYVAR} 这个变量，如果有则被替换为变量的值来进行接下来的比较，否则保持原来的字符串不变。  
~~~cmake
set(MYVAR Hello)
if(MYVAR MATCHES Hello)
	message("MYVAR is Hello")
else()
	message("MYVAR is not hello")
endif()
~~~
  
### 如果加上 ${} 也没区别
  
`if(${MYVAR} MATCHES "Hello")` 会展开成 `if(Hello MACHES "Hello")`  
因为没有 Hello 变量所以被视为字符串正常进行匹配。
  
### 万一定义了 Hello 变量那就寄了
  
~~~cmake
set(MYVAR Hello)
set(Hello world)
if(${MYVAR} MATCHES "Hello")
	message("MYVAR is Hello!")
else()
	message("MYVAR is not Hello!")
endif()	
~~~
  
`if(${MYVAR} MATCHES "Hello")` 变成 `if(Hello MATCHES "Hello")`  
if 认为用户要使用 Hello 变量，然后就出错了。  
这里不要自作聪明加 ${} 就好了。
  
### 解决：用引号包裹，防止被当作变量名
  
~~~cmake
set(MYVAR Hello)
set(Hello world)
if("${MYVAR}" MATCHES "Hello")
	message("MYVAR is Hello!")
else()
	message("MYVAR is not Hello!")
endif()	
~~~
  
但是你不觉得麻烦吗？直接变量名就好了。  
**另外：CMake 仅仅是指令( set,message 这些 )不分大小写，但变量名什么的是分大小写的！**
  
## 第十章：变量和作用域
  
### 变量的传递规则：父传子
  
* 父模块内容会传递给子模块
  
* CMakeLists.txt
  
  ~~~cmake  
  cmake_minimum_required(VERSION 3.15)
  
  set(MYVAR ON)  
  add_executable(main main.cpp)  
  ~~~
  
* m/CMakeLists.txt  
  ~~~cmake  
  message("MYVAR is ${MYVAR}")  
  ~~~
  
* Output  
  >MYVAR is ON
  
### 变量传递规则：子不传父
  
* 如果父模块本来就定义同名变量，则离开子模块后仍保持父模块原来设置的值。
  
* CMakeLists.txt  
  ~~~cmake  
  cmake_minimum_required(VERSION 3.15)
  
  set(MYVAR OFF)  
  add_subdirectory(mylib)  
  message("MYVAR:${MYVAR}")  
  ~~~
  
* m/CMakeLists.txt
  
  ~~~cmake  
  set(MYVAR ON)  
  ~~~
  
* Output
  
  >MYVAR:OFF
  
### 若子模块想向父模块传递变量该怎么办？
  
* 可以使用 set 的 PARENT_SCOPE 选项把一个变量传递到上一层作用域
  
* CMakeLists.txt
  
  ~~~cmake  
  cmake_minimum_required(VERSION 3.15)
  
  set(MYVAR OFF)  
  add_subdirectory(mylib)  
  message("MYVAR:${MYVAR}")  
  ~~~
  
* m/CMakeLists.txt
  
  ~~~cmake  
  set(MYVAR ON PARENT_SCOPE)  
  ~~~
  
* Output
  
  >MYVAR:ON
  
* 如果父模块没有定义 MYVAR，也可以使用缓存变量向外传递( 不建议, 这样很不安全 )，但是因为缓存变量是全局的，这样不仅父模块可见，父模块的父模块也可见。
  
  * CMakeLists.txt
  
    ~~~cmake  
    cmake_minimum_required(VERSION 3.15)
    
    add_subdirectory(mylib)  
    message("MYVAR:${MYVAR}")  
    ~~~
  
  * m/CMakeLists.txt
  
    ~~~cmake  
    set(MYVAR ON CACHE BOOL "" FORCE)  
    ~~~
  
  * Output
  
    >MYVAR:ON
  
### 除了父模块还有哪些是带有独立作用域的
  
* include 的 XXX.cmake **没有**独立作用域  
* add_subdirectory 的 CMakeLists.txt 有独立作用域  
* macro **没有**独立作用域，插入执行，变量会暴露出来  
* function **有**独立作用域，变量不会暴露出来  
* 因此 PARENT_SCORE 也可以用于 function 的返回值
  
### 环境变量的访问方式：$ENV{xx}
  
* 用 ${xx} 访问的是局部变量，局部变量服从刚刚说的父子模块传递规则。
  
* 而还有一种特殊的方式可以访问系统的环境变量( enviroment variable ) : $ENV{xx}
  
* 比如 $ENV{PATH} 获取的就是 PATH 这个环境变量的值  
  ~~~cmake  
  cmake_minimum_required(VERSION 3.15)  
  project(helloCMake)
  
  message("PATH:$ENV{PATH}")  
  ~~~
  
### 缓存变量的访问方式：$CACHE{xx}
  
* 还可以用 $CACHE{xx} 访问缓存变量  
  缓存变量和环境变量都是全局的，没有作用域一说
  
  ~~~cmake  
  cmake_minimum_required(VERSION 3.15)  
  project(helloCMake)
  
  message("CMAKE_BUILD_TYPE:$CACHE{CMAKE_BUILD_TYPE}")  
  ~~~
  
### ${xx} 找不到局部变量时，会自动去找缓存变量
  
* 当 ${xx} 在局部变量找不到时，回去查询名为 xx 缓存变量
  
* 所以这里虽然没有定义 CMAKE_BUILD_TYPE，但是 ${} 在缓存变量中找到了  
  ~~~cmake  
  cmake_minimum_required(VERSION 3.15)  
  projecr(helloCMake)
  
  message("CMAKE_BUILD_TYPE:${CMAKE_BUILD_TYPE}")
  
  add_executable(main main.cpp)  
  ~~~
  
### if(DEFINED XX) 判断变量是否存在
  
if(DEFINED MYVAR) 可以判断是否定义了 MYVAR 变量，判断的是**局部变量**和**缓存变量**  
~~~cmake
cmake_minimum_required(VERSION 3.15)
projecr(helloCMake)

if(DEFINED MYVAR)
	message("MYVAR:${MYVAR}")
else()
	message("MYVAR not defined")
endif()
~~~
  
需要注意的是即便变量是空字符串也是被认为存在的，因为 DEFINED 判断的是*是否被定义*。
  
### if(xx) 就可以判断是否存在且不为空
  
可以直接用 if(xx) 来判断空字符串，因为空字符串等于 OFF  
~~~cmake
cmake_minimum_required(VERSION 3.15)
projecr(helloCMake)

set(MYVAR "")
if(MYVAR)
	message("MYVAR is:${MYVAR}")
else()
	message("MYVAR is empty or not defined")
endif()
~~~
  
### if(DEFINED ENV{xx}) 判断环境变量是否存在
  
* 因为 $ENV{xx} 代表环境变量，因此在 set 和 if 中也可以用 ENV{xx} 来表示环境变量  
  因为 set 的第一参数和 if 的参数都是不加 $ 的，  
  所以要设置 ${x} 就变成了 set(x ...);  
  设置 $ENV{x} 就变成了 set( ENV{x} ...)  
  同理还可以用 if(DEFINED CACHE{x} ) 判断是否存在 缓存变量x  
  但是 set( CACHE{x} ...) 不行，别搞错了。
  
  ~~~cmake  
  cmake_minimum_required(VERSION 3.15)  
  project(helloCMake)
  
  set(ENV{MYVAR} "hello")  
  if(DEFINED ENV{MYVAR})  
  	message("MYVAR:$ENV{MYVAR}")  
  else()  
  	message("MYVAR is not defined!")  
  endif()  
  ~~~
  
### 第十一章：小建议
  
### CCache：编译加速缓存
  
* 用法：把 gcc -c main.cpp -o main 换成 ccache gcc -c main.cpp -o main 即可  
  在 CMake 中，可这样来启用 ccache ( 就是给每个编译和链接命令前面加上 ccache )
  
  ~~~cmake  
  cmake_minimum_required(VERSION 3.15)  
  project(helloCMake)
  
  find_program(CCACHE_PROGRAM ccache)  
  if(CCACHE_PROGRAM)  
  	message(STATUS "Found CCache:${CCACHE_PROGRAM}")  
  	set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ${CCACHE_PROGRAM})  
  	set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ${CCACHE_PROGRAM})  
  endif()  
  ~~~
  
* CCache 官网：https://ccache.dev/ ( 不过好像不支持 MSVC )
  
### 添加一个 run 伪目标，用于启动主程序( 可执行文件 )
  
* 创建一个 run 伪目标，其执行 main 的可执行文件  
  这里用了生成器表达式 `$<TARGET_FILE:main>` 会自动让 run 依赖 main  
  如果自动依赖失败，可以手动加上 add_dependencies(run main) 也是可以的。
  
* 这样就可以在命令行运行 cmake --build build --target run 来启动 main.exe 运行了。而不必根据不同的平台，手动写出 build/main 或者 build\main.exe  
  ~~~cmake  
  add_executable(main main.cpp)
  
  add_custom_target(run COMMAND $<TARGET_FILR:main>)  
  ~~~
  
### 再加一个 configure 伪目标，用于可视化地修改缓存变量
  
* 这样就可以 cmake  --build build --target configure 来启动 ccmake 修改缓存了  
  Linux 上相当于 ccmake -B build，Windows 则是 cmake-gui -B build
  
  ~~~cmake  
  add_custom_target(run COMMAND $<TARGET_FILE:main>)  
  if(CMAKE_EDIT_COMMAND)  
  	add_custom_target(configure COMMAND ${CMAKE_EDIT_COMMAND} -B ${CMAKE_BINARY_DIR})  
  endif()  
  ~~~

  
  
## [文件目录组织规范](https://www.bilibili.com/video/BV1V84y117YU)
  
基于CMake的项目组织。
  
### 推荐的目录组织
  
* project_name/include/project_name/module_name.h  
* project_name/src/module_name.cpp
  
将头文件放在include/project_name目录下是防止**不同子项目**或**项目**与**系统头文件**相冲突。
  
**在CMakeLists.txt中**使用
  
`target_include_directories(project_name PUBLIC include)`
  
指定项目名project_name, PUBLIC导入include文件
  
**源文件中**
  
* #include<project_name/module_name>  
* project_name::func();
  
**头文件中(project_name/include/project_name/module_name.h)**
  
~~~c++
#pragma once
namespace project_name{
    void func();
}
~~~
  
**实现文件(projecr_name/src/module_name.cpp)**
  
~~~c++
#include <project_name/module.h>
namespace project_name{
    void func()
    {
        int a;
	}
}
~~~
  
#### 例子
  
* biology  
  * CMakeLists.txt  
  * include  
    * biology  
      * Animal.h  
  * src  
    * Animal.cpp  
* CMakeList.txt  
* pybmain  
  * CMakeLists.txt  
  * include  
    * pybmain  
      * myutils.h  
  * src  
    * main.cpp
  
有点抽象……
  
<img src="D:\Book\C++\MyC++Note\CMake.png" alt="CMake" style="zoom:67%;" />
  
### 划分子项目
  
一个大型的项目不可能是仅仅一个项目，往往是要分成多个子项目。
  
通常分为库文件，可执行文件两个部分，**库文件**主要负责逻辑运算、数据处理诸如此类的**代码逻辑**；**可执行文件**主要是和**用户的交互逻辑**。
  
### 根项目的 CMakeLists.txt 配置
  
* 在根项目的 CMakeLists.txt 中，设置了该项目默认的构建模式，设置了统一的 C++ 版本等各种选项。然后通过 `project` 命令初始化了根项目。  
* 随后通过 `add_subdirectory` 把子项目添加进来( 顺序无关紧要 )，这会调用子项目的 CMakeLists.txt 。  
  比如调用 biology/CMakeLists.txt 和 pybmain/CMakeLists.txt
  
~~~cmake
cmake_minimum_required(VERSION 3.18)

if (NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release)
endif()
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

project(CppCMakeDemo LANGUAGES CXX)

add_subdirectory(pybmain)
add_subdirectory(biology)
~~~
  
# 我遇到的实际问题
  
## 将资源拷贝到Build目录，供可执行文件读取
  
~~~cmake
add_custom_command(
	TARGET main POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy_directory
			${CMAKE_CURRENT_SOURCE_DIR}/obj
			${CMAKE_CURRENT_BINARY_DIR}/obj
)
~~~
  
[CMake官方：add_custom_command](https://cmake.org/cmake/help/latest/command/add_custom_command.html?highlight=add_custom_command)  
[Post copy files to currently building target directory.](https://discourse.cmake.org/t/post-copy-files-to-currently-building-target-directory/6027)在这个问题中：“For example, if I’m building demo1 I want the output dir to be demo1s binary dir, if I’m building demo2, demo2’s binary dir etc.”有点意思但是我没想到解决思路用`${PROJECT_SOURCE_DIR}`和`${PROJECT_BINARY_DIR}`不行吗？  