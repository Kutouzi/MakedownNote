## c++笔记-->细节

### c和c++的区别

c是面向过程的，c++除了这个还可以面向对象。以及c++新增了new和delete管理动态内存

### 指针和引用的区别

引用本质是一个被限制的指针，它不能被重新指向，不能被初始化为nullptr，作为参数传地址可以少写*号解引用

### 结构体和联合体的区别

结构体的长度是整合的数据类型内存对齐后的长度和，联合体的长度是最长的数据类型的长度

### 内存对齐的原因

cpu读取内存是一次读取几个块，为了增加缓存命中率进行内存对齐可以有效提升性能

### 结构体和类的区别

类中允许有私有和保护的访问级别，而结构体中是公有的，除此之外都一样

### define和const的区别

define不是常量，不会分配内存。

const在编译时确定，有三种情况：

1. 当存被const修饰的变量在类或结构体内时，对象以new开辟空间会存放在堆中，否则存放在栈中
2. 当存被const修饰的变量同时是全局变量（被extern修饰）或同时是静态变量（被static修饰），它会保存在只读数据段的常量区中，被写入可执行文件并保存在磁盘内，下次程序运行时操作系统会映射到只读数据段中
3. 当存被const修饰的变量在函数内或函数外，编译器会让它会直接成为一个立即数，不会分配任何内存

以上三种情况根据编译器不同可能不同，测试结果是vs编译器得出的

### 什么情况必须使用宏

1. 当一个函数内有预处理器指令时
2. 需要写跨平台代码时
3. 需要记录调试信息时
4. 需要封装、替换大量代码，并且不方便使用函数封装时

除此之外请使用模板

### 重载、重写和重定义的区别

重载是一个函数名根据参数不同调用不同的同名函数；重写是父子类继承时把父类的函数覆盖掉，这种父类一般是接口；重定义是子类定义了与父类同名的函数但仅有函数名一样

### new和delete的实现过程

new有两阶段，分配内存，调用构造函数。delete则是反过来

### new和malloc的区别

new比malloc对对象的支持更好，new不需要给出内存长度，new可以运算符重载

### delete和delete[]的区别

delete[]是用来释放动态分配的数组的，比如new int[12]会在堆上开辟数组，要用delete[]来释放

### static和const的区别

static将数据保留在数据段中静态区，表示在程序运行中不可以释放。const将数据保存在只读数据段中常量区，表示在程序运行中不可以更改。如果const static一起用，那就表示在程序运行中不可释放也不可改变，会保存在**只读数据段**中。

### 程序的内存的结构

具体结构地址顺序由节表来定义，一般是如下

低地址-------------->高地址

| 代码段（.text） | 初始化数据段（.data）<->只读数据段（.rdata）<->未初始化数据（.bss） | 导入函数和变量（.idata或.import） | 堆（heap）<-> 栈（stack） |
| --------------- | ------------------------------------------------------------ | --------------------------------- | ------------------------- |

1. **可执行代码**：这是程序运行的主体部分，通常位于`.text`节中。
2. **初始化数据**：包括初始化的全局和静态变量，可能位于`.data`节中。
3. **只读数据**：例如常量字符串和导入表，这些通常存储在`.rdata`或类似命名的节中。
4. **未初始化数据**：未初始化的全局和静态变量可能存储在`.bss`节中。
5. **导入函数和变量的信息**：这部分数据描述了程序所依赖的外部函数和变量，通常位于`.idata`或`.import`节中。

代码段、数据段、导入函数和变量的信息在磁盘中会存在，堆和栈在程序运行后才产生。

1. **代码段**：包含程序的执行指令，也就是机器码。这部分在硬盘上的PE文件中是静态的，并且在程序加载到内存时会被复制到相应的代码段。
2. **初始化数据段**：包含程序在编译时已知并初始化的全局和（由static声明）静态变量。这部分数据也在硬盘上的PE文件中是静态的，并在程序加载时复制到内存中的初始化数据段。
3. **只读数据段**：通常包含（由const声明）常量数据和只读的全局变量。这些信息在PE文件中也是静态的，并在程序加载时复制到内存中的只读数据段。
4. **未初始化数据段**：用于存储未初始化的全局和静态变量。在PE文件中，这个段通常只占用空间而不包含实际的数据，因为未初始化的变量在内存中默认为0。当程序加载时，操作系统会为这部分变量在内存中分配空间并初始化为0。
5. **导入函数和变量段**：这个段包含了程序所依赖的外部函数和变量的信息，用于动态链接库（DLL）的导入。这部分数据在PE文件中也是静态的，但在程序运行时，操作系统会使用这部分信息来解析和加载所需的DLL。
6. **堆栈段**：堆栈用于存储局部变量、函数调用的返回地址等信息。这个段在程序运行时由操作系统动态创建和管理，不在硬盘上的PE文件中存在。

堆和栈共用一个区域，堆向地址增加的方向生长，栈向地址减少的方向生长。因此内存不足时容易发生堆栈冲突。

### static、const和extern多修饰下的数据分布

**初始化数据段**和**未初始化数据段**合并称为**全局静态区**，不包括只读数据段

1. 仅被`const`修饰：
   - 在C语言中，`const`全局变量存储在**只读数据段**，并在编译期最初保存在符号表中，第一次使用时为其分配内存，程序结束时释放。而`const`局部变量存储在栈中，代码块结束时释放。
   - 在C++中，一个`const`并不总是需要创建内存空间，这取决于其用途。如果是作为一个值替换（即就是将一个变量名替换为一个值），则**不分配内存空间**。但是，如果对这个`const`全局变量取地址或者使用`extern`，则会为其分配内存，并存储在**只读数据段**中。
2. 仅被`static`修饰：
   - 在C或C++中，`static`修饰的变量（无论是全局还是局部的）都存储在内存的**全局静态区**。这样的变量只在本模块（即定义它的源文件）的所有函数中可见，其他模块无法直接访问。
3. 仅被`extern`修饰：
   - `extern`用于声明一个变量或函数，它告诉编译器这个变量或函数是在其他地方（通常是另一个源文件中）定义的。因此，`extern`本身并不分配存储空间，它只是一个引用或声明。真正的存储空间是**在该变量或函数实际定义的地方分配**的。
4. 同时被`const`和`static`修饰：
   - 如果一个变量同时被`const`和`static`修饰，那么它大多是被存储在**全局静态区**，也可能被储存在**只读数据段**，根据编译器实现而定。不过其内容都是不可变的。在C或C++中，这样的变量具有文件作用域（即只在定义它的文件中可见）和内部链接（即它不会与其他文件中的同名变量冲突）。
5. 同时被多个修饰符修饰（如`const`、`static`、`extern`）：
   - 当一个变量同时被`const`、`static`和`extern`修饰时，情况会变得复杂，并且实际上可能不是有效的组合。`extern`通常用于声明一个在其他地方定义的变量，而`static`则用于在源文件内部定义和初始化一个变量。同时使用这两个修饰符可能会导致编译错误或未定义的行为。
   - 一般来说，如果`extern`和`static`一起使用，`static`的作用会被忽略，因为`extern`表明变量是在其他地方定义的。而`const`表明变量的内容是不可变的。
   - 在实际编程中，很少会同时看到这三个修饰符用于同一个变量。更常见的做法是使用其中的两个，或者仅使用其中一个。

### 堆和栈的区别

堆保存了new的对象，生命周期直到程序运行结束

栈保存了局部变量、函数、返回值等数据，生命周期在花括号以内

栈的速度要比堆快，因为只需要计算并移动栈顶指针就完成了内存分配；堆需要额外计算堆使用情况，是否存在连续内存块等。

### 为什么尽量不用全局变量

因为在函数和类的外部定义变量是线程不安全的

### <>和""引入头文件的区别

尖括号是从标准库路径查找，双引号是从当前工作路径查找

### 什么导致内存泄漏

使用new分配内存而不进行delete释放导致

### 如何防止内存泄漏

使用智能指针，或者用ide的插件来进行检测

### 定义和声明的区别

分配内存是定义，不分配内存是声明（用extern实现）

### c++文件编译的阶段

1. 预处理：根据文件中预处理指令修改文件
2. 编译：将高级语言编译成目标处理器汇编代码
3. 汇编：把汇编代码翻译成目标机器指令
4. 链接：链接到其他动态链接库后生成可执行程序

### 什么是自由存储区

c++中的new说是在自由存储区上分配内存，操作系统层面并不存在自由存储区这种区域。

根据编译器的优化不同，new可能堆上分配内存，也可能在栈上分配内存（比如一个较短的字符串，当它使用new分配空间时，编译器会进行优化，将它分配到栈上而不是堆）这就是所谓的自由存储。

### c++的四种强制类型转换

1. 静态转换(static_cast)：可以用于非多态类型之间的转换，如整数和浮点数之间的转换。但是不会自己检测转换是否合法。
2. 动态转换(dynamic_cast)：主要用于多态类型的转换，将一个父类指针或引用转换为子类指针或引用。与静态转换不同的是，动态转换在运行时会检查转换是否合法，如果不合法则返回空指针或引用。
3. 重新解释转换(reinterpret_cast)：用于不同类型之间的位模式的转换，例如将整数转换为指针或将指针转换为整数。这种转换非常危险，容易引起未定义行为，因此应该尽量避免使用。
4. 常量转换(const_cast)：用于将常量或只读对象的类型转换为非常量或可写对象的类型。常量转换也是非常危险的，因为它可能会导致未定义行为，所以应该谨慎使用。

### extern “C”的作用

表示调用c的代码

### typedef和define的区别

typedef类型的函数安全，而define文本替换的函数可能存在隐患类型错误。typedef可读性更强。

### 引用作为返回值的好处

避免对象拷贝、避免空指针、支持对象的链式修改

### 常见的未定义行为

？

### 如何把当函数对象用

重载此对象的()运算符

### 什么是野指针

指针指向未知的内存

### 悬垂引用的发生

悬垂引用（dangling reference）通常发生在试图保留对已经销毁的对象的引用时。

```c++
#include <iostream>  
const int& getLocalConstReference() {  
    const int localConst = 42; // 局部变量  
    return localConst; // 返回局部变量的引用，这是错误的！  
}  
int main() {  
    const int& refToLocalConst = getLocalConstReference(); // 获取局部变量的引用  
    std::cout << "The value is: " << refToLocalConst << std::endl; // 试图使用悬垂引用  
    return 0;  
}
```

下面这个不是悬垂引用，而是return进行了值复制

```c++
#include <iostream>  
int printConstValue() {  
    const int localConst = 42; // 单被const修饰的局部变量  
    std::cout << "The value of localConst is: " << localConst << std::endl;  
    return localConst; // 返回localConst的值，而不是引用或地址  
}  
int main() {  
    int valueFromFunction = printConstValue(); // 调用函数并接收返回值  
    std::cout << "The value returned from the function is: " << valueFromFunction << std::endl; // 打印从函数中返回的值  
    return 0;
}
```

### 悬垂引用的后果

- 数据错误：引用的内存可能被释放
- 程序崩溃：如果当操作系统对上面的行为进行阻止，会引发崩溃
- 未定义行为：程序不可预测，每次运行结果不一样
- 难以追查：悬垂并不会立刻导致问题，出现问题后追踪会藏得很隐蔽，难以发现

### 栈溢出的原因和解决方法

尽量不使用深度过深的递归

减少局部变量的体积，将其转成堆内存

### 什么左值和右值

见指针篇

### 左右值引用的区别

见指针篇

### 头文件怎么防止重复包含

使用#ifndef，配合#define和#endif，这个方式对较老编译器支持好

使用#program once

### 指针数组和数组指针区别

？

### c++是否类型安全

不是，因为指针可以强转

### main前还执行什么

1. 操作系统加载可执行文件到内存中
2. 为程序分配代码段、数据段、堆和栈
3. 操作系统为全局变量和静态变量进行初始化，并映射可执行文件中的常量到只读数据段
4. 操作系统为程序设置堆栈指针，给出栈顶指针
5. 操作系统打开输入流、输出流和错误流
6. 调用构造函数（如果有全局或静态的对象的话）

### sizeof究竟是什么

它是由编译器提供的一个特性，当有sizeof()被调用，编译器在编译时直接给出此对象长度的立即数

它不是函数也不是关键字

它不能用在动态数组，外部数组，静态变量上

### 常用类型的内存长度

char是1字节、int是4字节、指针根据操作系统来（64位是8字节）、float是4字节、double是8字节、string是4字节、空类是1字节。

有short的字节数砍一半，有long的字节数翻一倍，有unsigned不影响长度

数组总字节数=数组长*元素类型的字节数（字符数组要+1空字符）

### 内联函数的使用

适用于较短的函数，为了减少函数调用时保存数据和还原现场的性能开销

它的原理是在编译阶段，直接把这函数内的代码嵌入调用此函数的函数中，会增加文件体积

### operator new和new operator的区别

new operator是常见的new表达式，它分为operator new和placement new两步

new operator不允许重载，但operator new可以

operator new阶段会为对象分配内存

placement new阶段会调用对象的构造函数

### c++的存储期

- 自动存储期：未被static、extern或thread_local修饰的对象，生命周期为花括号内
- 静态存储期：被static或extern修饰的对象，生命周期为程序运行期间内
- 线程存储期：被thread_local修饰的对象，生命周期为线程存在期间内
- 动态存储期：由new表达式产生的对象，生命周期可自定义

### c++的函数闭包是什么

是一个lambda函数可以直接访问到调用此函数的函数内的全部变量

### c++中的垃圾回收

使用智能指针来实现堆内存的自动回收

### STL中容器的开销问题

clear除了线性存储的容器外，时间复杂度都不是常数

遍历时尽量使用auto&避免写错类型而导致的对象复制

at()和下标访问都会进行一次遍历，避免重复遍历

sort对象时必须要写移动构造，否则将进行复制

### 枚举类和enum的区别

枚举类限制了枚举成员，只在类内可见，结构上更符合面向对象

### lambda使用的注意点

需要注意被捕获区域的指针，可能会被外部改变而导致lambda内捕获到空指针

### 移动语义和完美转发

见指针篇

### 进程、线程和协程区别

当一个程序启动时，操作系统会启动一个进程，为它分配堆栈空间和初始化常量。

以上完成后，进入主函数运行的是主线程，在主线程上开辟的线程都是子线程（这个开辟可以是申请用其他cpu，也可以是轮巡当前cpu的时间）。如果主线程结束，子线程没结束，进程会变为僵尸进程，此时操作系统不会回收分配给这个程序的空间

协程本质是线程，可以是单线程也可以是多线程。由于它可以在代码中主动让出自己线程的cpu占有权被称为协程。

### 协程切换和线程切换区别

协程切换是主动让出cpu，线程切换是抢占cpu。协程切换可以不给变量加锁而保证最终一致性，线程切换不能保证最终一致性。由于cpu的指令重排序，它们两都不能保证过程一致。

### 线程不安全产生的原因

操作系统随机调度、多线程修改同一个变量或非线程安全的容器、原子操作被破坏、指令重排序（this逃逸）、内存可见性问题

（这些和java出现的问题基本相同，见java笔记中的多线程下的高并发篇）

### 各种锁

见java笔记中的多线程下的高并发篇

### 死锁的解决方式

？

### 进程之间通信方法

管道（PIPE）、信号量（Semaphore）、信号（Signal）、消息队列（Message Queue）、共享内存（Shared Memory）、套接字（Socket）

### 线程之间通信方法

互斥锁（Mutex）、读写锁（Reader-writer Lock）、自旋锁（Spin Lock）、条件变量（Condition）、信号量（Semaphore）

### SLT中常用组件

容器、算法、迭代器、仿函数、适配器、空间配置器

### stack中top和pop分离的原因

为了容器的安全性。如果不分离，在栈为空的情况下，pop返回失败，top返回意料外的值，这也是一种未定义行为

### STL中关联式容器实现原理

map、set、multiset、multimap都是由红黑树实现

### STL容器线程不安全的问题

尽管容器一般情况是线程安全的，但如果触发了容器的再散列和扩容，读操作都会变得不安全

### STL容器迭代器失效的情况

插入、删除、再散列、扩容，都会导致迭代器失效

### STL容器是侵入性还是复制性的

是复制性的，会发生数据拷贝

### TCP粘包和拆包问题

？

### 单双链表的优缺点

单链表头确定，双链表扩展性好

### 红黑树比平衡树的优势

见数据结构笔记的树篇

### 如何判断链表是不是环形

用hash表判断访问地址是否重复；用快慢指针法