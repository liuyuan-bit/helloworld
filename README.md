# gcc/g++ 编译c/cpp的区别（十分啰嗦，十分详细）

小科普

+ GCC：GNU Compiler Collection(GUN 编译器集合)，它可以编译C、C++、JAV、Fortran、Pascal、Object-C、Ada等语言。

+ gcc是GCC中的GUN C Compiler（C 编译器）

+ g++是GCC中的GUN C++ Compiler（C++编译器）

另外注意两点

+ 实际上 g++ == gcc -xc++ -lstdc++ -shared-libgcc，第一项是编译选项，表示按照c++编译，后面两项是链接选项，表示g++要相比gcc多链接其他库函数

+ 大多数系统，GCC 安装时会安装一名为 c++ 的程序。如果有安装，它和 g++ 等同

gcc编译的四个阶段：预处理、编译、汇编、链接。前三个阶段对gcc和g++**几乎**都是一样的，最后的链接差异比较大。下面逐一进行解释。

## gcc/g++编译c文件

首先说预处理和汇编，从g++ == gcc -xc++ -lstdc++ -shared-libgcc，能看出来，g++只针对编译和链接做了调整，但对预处理和汇编而言，g++与gcc是**完全等价**的。其次，如果编译的是.cpp文件，gcc会自动按照.cpp的标准也就是c++的标准进行编译；如果编译的是.c文件，在没有涉及一些不规范语法的情况下，两者也是完全等价的，前面的-xc++可能因为使用c++的编译标准而不支持c语言一些语法，要求更严一些，但后面链接的其他库对没有使用c++库的代码是没有意义的。我们写一个demo1.c来作示范。

示例如下：

```c
#include <stdio.h>
int main()
{
    printNum();
    getchar();
    return;
}

int printNum()
{
    printf("%d", 2);
}
```

这是一段不规范的c代码，return的返回值类型与函数的返回值类型不一致，函数未声明就被调用，被调用的函数也没有返回值。

下面我们先用gcc来编译

```
gcc -xc demo1.c // 指定语言类型为c语言 （此处我忘记指定输出文件名了，Windows下默认为a.exe, Linux下默认为a.out)
```

输出了一堆警告（其中*note是*类型不统一）

```c++
D:\study\master\code\cpp\demo>gcc -xc demo1.c   
demo1.c: In function 'main':
demo1.c:13:5: warning: implicit declaration of function 'printNum'; did you mean 'printf_s'? [-Wimplicit-function-declaration] 
     printNum();
     ^~~~~~~~
     printf_s
demo1.c:15:5: warning: 'return' with no value, in function returning non-void
     return;
     ^~~~~~
demo1.c:11:5: note: declared here
 int main()
     ^~~~
```

然后我们允许生成的a.exe可执行文件，刚刚忘记 -o 指定文件名了

```
D:\study\master\code\cpp\demo>a.exe
2
```

我们可以看到，正常输出了，当然代码是不规范的，不推荐大家这么写代码。

然后使用指定为c++进行编译

```c
D:\study\master\code\cpp\demo>gcc -xc++ demo1.c   （此处我又忘记指定输出文件名了......）
```

然后就报错了，可见c++对语法要求更为严格。

```c++
demo1.c: In function 'int main()':
demo1.c:13:5: error: 'printNum' was not declared in this scope
     printNum();
     ^~~~~~~~
demo1.c:13:5: note: suggested alternative: 'printf_s'
     printNum();
     ^~~~~~~~
     printf_s
demo1.c:15:5: error: return-statement with no value, in function returning 'int' [-fpermissive]
     return;
     ^~~~~~
demo1.c: In function 'int printNum()':
demo1.c:21:1: warning: no return statement in function returning non-void [-Wreturn-type]
 }
 ^
```

为了保险起见，我们根据gcc编译链接的四个过程，逐步进行

```c++
D:\study\master\code\cpp\demo>gcc -o demo1.i -xc++ -E demo1.c  // 此处是对源代码进行预处理，生成纯c代码，文件名指定为demo1.i，无差错
    
D:\study\master\code\cpp\demo>gcc -o demo1.s -xc++ -S demo1.i  // 此处是对预处理过后的纯c代码进行编译，并加入xc++选项，即按照c++的标准进行编译（只编译，并不进行汇编和链接，即只生成汇编代码 ，文件名指定为demo1.s，不产生二进制目标代码及可执行文件）
    
// 报错    
demo1.c: In function 'int main()':
demo1.c:13:5: error: 'printNum' was not declared in this scope
     printNum();
     ^~~~~~~~
demo1.c:13:5: note: suggested alternative: 'printf_s'
     printNum();
     ^~~~~~~~
     printf_s
demo1.c:15:5: error: return-statement with no value, in function returning 'int' [-fpermissive]
     return;
     ^~~~~~
demo1.c: In function 'int printNum()':
demo1.c:21:1: warning: no return statement in function returning non-void [-Wreturn-type]
 }
 ^
```

然后，我们将代码修改为规范的c语言代码

```c
#include <stdio.h>
int printNum()
{
    printf("%d", 2314);
    return 0;
}

int main()
{
    printNum();
    // getchar();
    return 0;
}
```

然后编译运行

```c
D:\study\master\code\cpp\demo>gcc -xc++ demo1.c  // 因为不涉及c++的库，因此gcc -xc++ 在此等价于g++

D:\study\master\code\cpp\demo>a.exe  // 不带后缀.exe，直接调用a也可
2314   // 正确输出
    
// 不放心的话，我们再用g++试一次
D:\study\master\code\cpp\demo>g++ -o cpp1 demo1.c 

D:\study\master\code\cpp\demo>cpp1
2314
```

由上述两个例子可以看出，对于c文件来说，gcc和g++的区别不大（前提是你要将代码写规范）

## gcc/g++编译cpp文件

从文章的第一部分，我们已经知道，gcc编译链接的四个阶段对于c文件几乎没有区别。那么对于cpp文件来说，前三个阶段就是**完全没有区别**，因为cpp文件本身就要按照c++的标准编译。那么问题就来了，第四个阶段链接的区别究竟在哪呢？

这次我们重新写一个demo2.cpp

```c++
#include <iostream>
using namespace std;
int main()
{
    cout << 1234;
    return 0;
}
```

这里使用了c++的标准库<iostream>，我们尝试用gcc进行编译

```
D:\study\master\code\cpp\demo>gcc -o demo2    demo2.cpp            
C:\Users\L\AppData\Local\Temp\ccBaPqOg.o:demo2.cpp:(.text+0x1e): undefined reference to `std::cout'
C:\Users\L\AppData\Local\Temp\ccBaPqOg.o:demo2.cpp:(.text+0x23): undefined reference to `std::ostream::operator<<(int)'
C:\Users\L\AppData\Local\Temp\ccBaPqOg.o:demo2.cpp:(.text+0x43): undefined reference to `std::ios_base::Init::~Init()'
C:\Users\L\AppData\Local\Temp\ccBaPqOg.o:demo2.cpp:(.text+0x64): undefined reference to `std::ios_base::Init::Init()'
collect2.exe: error: ld returned 1 exit status
```

报错了，找不到对应的函数，这也就是网上都说的无法连接到c++对应的库。

那我们将c++的标准库链接进来试试看

```c++
D:\study\master\code\cpp\demo>gcc -o demo2    demo2.cpp  -lstdc++

D:\study\master\code\cpp\demo>demo2 
1234
```

这时候可以了，但是还无法说明究竟是哪一步出了问题，我们再试试看

```c++
D:\study\master\code\cpp\demo>gcc -o demo2.o -c  demo2.cpp       

D:\study\master\code\cpp\demo>ls            
demo1.c  demo1.h  demo2.cpp  demo2.exe  demo2.o  stringAllCombination.cpp
```

看来链接之前都是可以的，但是果真是这样的吗？如果我们一步一步进行呢，从预处理开始呢？

```c++
D:\study\master\code\cpp\demo>gcc -o demo2.i -E  demo2.cpp  // 预处理可以

D:\study\master\code\cpp\demo>gcc -o demo2.s -S  demo2.i    // 编译这一步就不行了
In file included from C:/Program Files (x86)/mingw-w64/i686-8.1.0-posix-dwarf-rt_v6-rev0/mingw32/lib/gcc/i686-w64-mingw32/8.1.0/include/c++/iostream:38,
                 from demo2.cpp:112:
C:/Program Files (x86)/mingw-w64/i686-8.1.0-posix-dwarf-rt_v6-rev0/mingw32/lib/gcc/i686-w64-mingw32/8.1.0/include/c++/i686-w64-mingw32/bits/c++config.h:236:1: error: unknown type name 'namespace'
 namespace std
 ^~~~~~~~~
C:/Program Files (x86)/mingw-w64/i686-8.1.0-posix-dwarf-rt_v6-rev0/mingw32/lib/gcc/i686-w64-mingw32/8.1.0/include/c++/i686-w64-mingw32/bits/c++config.h:237:1: error: expected '=', ',', ';', 'asm' or '__attribute__' before '{' token
```

我们想一想这是为什么？

之所以直接进行到编译这一步可以，是因为gcc可以根据后缀名判断按照c还是cpp的标准进行编译，当我们完成了预处理之后，将纯cpp代码的后缀名改为了.i（其实就是.c），于是按照c文件进行处理，结果报错。因此，说明gcc确实可以根据文件后缀判断如何进行编译。

```c++
D:\study\master\code\cpp\demo>gcc -o demo2i.cpp -E  demo2.cpp   // 我们将预处理的文件后缀改为.cpp

D:\study\master\code\cpp\demo>gcc -o demo2i.s -S  demo2i.cpp    // 没有报错

D:\study\master\code\cpp\demo>ls
demo1.c  demo1.h  demo2.cpp  demo2.exe  demo2.i  demo2.o  demo2.s  demo2i.cpp  demo2i.s  stringAllCombination.cpp
```

但是，到了链接的时候，gcc也没有办法，因为它确实无法链接到C＋＋程序使用的库，如果非要用gcc的话，需要在后面加上一个 -lstdc++，将c++的标准库链接进来。

## 总结

好了，啰里啰唆讲了这么多，也不知道大家看懂了没有。现在来总结一下：

+ 对于c文件，gcc与g++在**代码规范**的情况下，是**完全等价**的。
+ 对于cpp文件，在预处理、编译、汇编这三部分，gcc和g++也是等价的，前提是这三个步骤一起做，或者你在中间继续能够体现cpp的文件名。
+ 在涉及c++的标准库时，gcc无法链接到这些库，必须加上-lstdc++ 选项，相比之下，不如直接使用g++来得方便快捷。
+ g++就是gcc以默认c++的方式进行编译，然后链接的时候加上了一些c++的库。

啰嗦一些细节

+ -lstdc++ 要放在后面，因为编译器是从右向左调用和处理变量的（根据编译器版本的不同，有些版本可能支持放在前面）
+ -l 后面加的是动态库libstdc++.-l加的时候，把"lib"三个字符省略，例如链接libtest.so你就需要加 -ltest ,一般这个库在usr/lib下可以找到
+ gcc可以编译c++文件，也可以编译c文件，但默认是编译c文件的，加-lstdc++表示编译c++文件，即链接c++库，加-lc表示链接c库，默认情况下就是链接c库，所以如果编译c文件可以不加-lc。
+ --shared-libgcc 链接动态libgcc库

