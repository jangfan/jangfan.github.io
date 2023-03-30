---
title: 现代C++学习笔记（一）
date: 2023-03-05 17:45:22
tags: c++
categories: 学习笔记 

---

## C++的基本特性

### 程序的执行过程

程序被执行后就被称为一个进程，一个进程可以被划分为很多区域。

其中比较重要的是以下四个区域。

**1代码区与常量区：**进程按照代码区的代码执行，真正的常量也存储在这里，比如“abc”字符串，“1”，“88”等数字。这些是真正的常量。再看一下const关键字。const只不过是让编译器将变量视为常量罢了，和真正的常量有本质上的区别

**2栈区：**函数的执行所需的空间，注意，当函数执行完毕，函数对应的栈内存全部销毁。

**3堆区：**进程用来分配内存的地方，只有手动释放才能销毁内存。

**4静态变量区：**

(1)静态变量：常常遇到的一些局部作用范围，生命周期却很长的变量。

(2)全局变量：在c++中不建议使用，会破坏封装性。

![](https://cdn.jsdelivr.net/gh/jangfan/picb@main/image-20230306163148843-1678164508218-1.png)

------

**堆和栈的关系**

堆区有灵活的生命周期。如果需要创建的对象有几十M，每次调用函数都需要创建一个这么大的对象，再复制到对应的容器中，那就太过耗费内存了。而且栈内存非常的小，通常不超过8M。而使用堆内存，每调用一次函数就可以在堆内存中创建一个对象，容器中只要存储指针就可以了，极大的提高了程序效率。栈区是函数执行的区域，堆区是函数内灵活分配内存的地方，二者缺一不可。堆的唯一寻址方式就是指针，如果没有栈，根本无法使用堆。

------

### (*) new 关键字及内存泄漏

**1.new关键字是c++用来动态分配内存的主要方式**.

```c++
#include<isotream>

int main()
{
	int* pi = new int(100);
	std::cout << *pi << std::endl;
    delete pi;
	return 0;
}
```



new可以直接分配单个变量的内存，也可以分配数组。

```c++
#include<iostream>

int main()
{ 
	int* pi = new int[100]();//小括号初始化为零，没有小括号分配未定义的内存，而且不可以赋初值
	std::cout << pi[20] << std::endl;
	delete[] pi;//不加中括号会导致动态内存泄露
	return 0;
}
```



**在分配单个对象的内存时，**

当对象是普通变量时，可以分配对应的内存

当对象是类对象时，会调用构造函数，如果没有对应的构造函数，就会报错。

```c++
#include<iostream>

int main()
{
	std::string* pString = new std::string("hello world");//如果是字符串数组的话不能赋初值
	std::cout << *pString << std::endl;
	delete pString;
	return 0;
}
```



在分配数组对象内存时：

对于普通变量：可以使用“（）”将所有对象全部初始化为0。

对于类对象，有没有“（）”都一样，均使用默认构造函数，如果没有默认构造函数就会报错。

```c++
#include<iostream>

class Test
{
public:
	Test(int i_) :i(i_){}
private:
	int i;

};
int main()
{ 
	Test* pTest = new Test[100];//这是错误的,类Test不存在默认构造函数
	delete[] pTest;
    return 0;
}
```

**2内存泄漏**

​	内存泄露会导致堆内存的逐渐被占用，最终内存用完程序崩溃。常见的情况就是项目测试没问题，上线几天就炸了。然后就会非常麻烦，排查困难，损失很大。

**内存泄露是最严重的错误之一，程序不怕报错，就怕一开始运行的好好的，突然就出现了莫名其妙的错误。**

这句话也引出了后面的两个部分。（期待学习hhh）

**Part4的智能指针**可以非常好的避免内存泄露的问题。

**Part9的异常处理**部分可以恰当的处理程序出现的异常，让程序有错误就立马处理，或直接终止进程，或忽略，不要让异常莫名其妙。这是程序设计的重要理念。

### 命名空间

C++经常需要多个团队合作来完成大型项目。多个团队就常常出现起名重复的问题，C++就提供了命名空间来解决这个问题。

**例子**

```c++
ATest.h
#pragma once
void test();
BTest.h
#pragma once
void test();
BTest.cpp
#include "BTest.h"
#include<iostream>
void test()
{
	std::cout << "B::()" << std::endl;
}
ATest.cpp
#include "ATest.h"
#include<iostream>
void test()
{
	std::cout << "A::()" << std::endl;
}
main.cpp
#include<iostream>
#include"ATest.h"
#include"BTest.h"
int main() 
{
	test();//报错，不知道调用哪个test函数
	return 0;
}

```

解决(使用命名空间)

```c++
ATest.h
#pragma once
namespace A
{
void test();
}


BTest.h
#pragma once
namespace B
{   
void test();
}


BTest.cpp
#include "BTest.h"
#include<iostream>
namespace B
{
void test()
{
	std::cout << "B::test()" << std::endl;
}
}


ATest.cpp
#include "ATest.h"
#include<iostream>
namespace A
{
void test()
{
	std::cout << "A::test()" << std::endl;
}
}


main.cpp
#include<iostream>
#include"ATest.h"
#include"BTest.h"
int main() 
{
	B::test();
    A::test();
	return 0;
}
```

**顺便提两点**

命名空间的实现原理，C++最后都要转化为C来执行程序。在namespace A中定义的Test类，其实全名是A::Test。C++所有特有的库（指c没有的库）,都使用了std的命名空间。比如最常用的iostream。

**using关键字设计的目的之一就是为了简化命名空间的。using关键字在命名空间方面主要有两种用法。**

1. **using 命名空间::变量名**。这样以后使用此变量时只要使用变量名就可以了。

   ```c++
   main.cpp
   #include<iostream>
   #include"ATest.h"
   #include"BTest.h"
   using A::test;
   using B::test;//同时使用会报错
   int main() 
   {
   	test();
   	return 0;
   }
   ```

   

2. **using namspce 命名空间**。这样，每一个变量都会在该命名空间中寻找。

   ```c++
   main.cpp
   #include<iostream>
   #include"ATest.h"
   #include"BTest.h"
   using namespace A;
   using namespace B;//同时使用会报错
   int main() 
   {
   	test();
   	return 0;
   }
   ```

   **所以，头文件中一定不能使用using关键字。会导致命名空间的污染**

   ```c++
   错误代码
   ATest.h
   #pragma once
   namespace A
   {
   void test();
   }
   using namespace A;
   ```

------

### (*)C++的标准输入输出简介

输入输出简单来说就是数据在输入设备，内存，硬盘，输出设备之间移动的过程。

c语言设定了很多不相关的函数还实现这些过程。

比如printf就是让数据从内存到显示屏（显示屏就是输出设备）。scanf就是让数据从键盘（键盘是输入设备）到内存。此外还有从内存到磁盘的文件操作函数。

 c语言的函数虽然简单方便，但彼此之间没有关联。C++有了继承功能，可以让子类与父类之间有关联性，极大的提高各种输入输出功能之间的耦合性。

于是C++用继承功能重写了输入输出功能，这就是io库，io库引入了“流”的概念，数据从一个地方到另一个地方，原本地方的数据就没了，叫做流很贴切。

 io库是一个很大的部分，但现阶段我们只要会使用输入输出流，cout和cin就可以了。

cout可以让数据从内存流到输出设备，cin可以让数据从输入设备流到内存。

------

### const关键字的介绍

const是让编译器将变量视为常量，用const修饰的变量和真正的常量有本质的区别。

1. 真正的常量存储在**常量区**或**代码区**，比如“abcdefg”这个字符串就存储在常量区，而“3”，“100”这些数字就存储在代码区中，这些都是真正的常量，**无法用任何方式修改。**

2. const修饰的变量仍然存储在**堆区**或**栈区**中，**从内存分布的角度讲，和普通变量没有区别。**const修饰的变量并非不可更改的，C++本身就提供了mutable关键字（这个关键字在Part3就会讲的）用来修改const修饰的变量，从汇编的角度讲，const修饰的变量也是可以修改的。

------

### (**)auto关键词的使用

auto是C++11新加入的关键字，就是为了简化一些写法。

为了学习auto的类型推断，我使用一个boost库来确定变量的具体类型。boost库很大，可以选择编译自己想要的模块，我就直接全部编译了。boost是很复杂的，不是几句话能说清楚，要深入理解可以去官网学习。

演示

```c++
#include<iostream>
#include<boost/type_index.hpp>

using boost::typeindex::type_id_with_cvr;

int main()
{
	auto i = 100;
	std:: cout << type_id_with_cvr<decltype(i)>().pretty_name() << std::endl;
	return 0;
}
```



**来说一下auto，有好几个点需要注意：**

**1.**auto只能推断出类型，引用不是类型，所以auto无法推断出引用，要使用引用只能自己加引用符号。

```c++
#include<iostream>
#include<boost/type_index.hpp>

using boost::typeindex::type_id_with_cvr;

int main()
{
	int i = 100;
	auto& i2 = i;
	std:: cout << type_id_with_cvr<decltype(i2)>().pretty_name() << std::endl;//输出类型int &
	return 0;
}
```

**2.**auto关键字在推断引用的类型时：会直接将引用替换为引用指向的对象。其实引用一直是这样的，引用不是对象，任何使用引用的地方都可以直接替换成引用指向的对象。

```c++
#include<iostream>
#include<boost/type_index.hpp>

using boost::typeindex::type_id_with_cvr;

int main()
{
	int i = 100;
	const int& refi = i;
	auto& i2 = i;
	std:: cout << type_id_with_cvr<decltype(i2)>().pretty_name() << std::endl;//int &
	return 0;
}
```

**3.**auto关键字在推断类型时，如果没有引用符号，会忽略值类型的const修饰，而保留修饰指向对象的const，典型的就是指针。**可能有些不好理解，看看代码就好说了。3和4的主要作用对象就是指针.**

例子1

```c++
#include<iostream>
#include<boost/type_index.hpp>

using boost::typeindex::type_id_with_cvr;

int main()
{
	int i = 100;
	const int* const pi = &i;//前者const修饰的是指针pi修饰的值，后者const修饰的是pi，后者const会被忽略
	auto pi2 = pi;
	std:: cout << type_id_with_cvr<decltype(pi2)>().pretty_name() << std::endl;//int const *=const int *
	return 0;
}
```

例子2

```c++
#include<iostream>
#include<boost/type_index.hpp>

using boost::typeindex::type_id_with_cvr;

int main()
{
	const int i = 100;
	auto i2 = i;
	std:: cout << type_id_with_cvr<decltype(i2)>().pretty_name() << std::endl;//int
	return 0;
}
```



**4.**auto关键字在推断类型时，如果有了引用符号，那么值类型的const和修饰指向对象的const都会保留。

```c++
#include<iostream>
#include<boost/type_index.hpp>

using boost::typeindex::type_id_with_cvr;

int main()
{
	const int i = 100;
	const int* const pi = &i;

	auto& pi2 = pi;
	std:: cout << type_id_with_cvr<decltype(pi2)>().pretty_name() << std::endl;//int const * const &
	return 0;
}
```



```c++
#include<iostream>
#include<boost/type_index.hpp>

using boost::typeindex::type_id_with_cvr;

int main()
{
	const int i = 100;
	auto& i2 = i;
	std:: cout << type_id_with_cvr<decltype(i2)>().pretty_name() << std::endl;//int const &
	return 0;
}
```

**其实3，4为什么会出现这种情况，因为在传递值时，修改这个值并不会对原有的值造成影响。而传递引用时，修改这个值会直接对原有的值造成影响。**

**5.**当然，我们可以在前面加上const，这样永远都有const的含义。

```c++
#include<iostream>
#include<boost/type_index.hpp>

using boost::typeindex::type_id_with_cvr;

int main()
{
	const int i = 100;
	const auto i2 = i;
	std:: cout << type_id_with_cvr<decltype(i2)>().pretty_name() << std::endl;//int const 
	return 0;
}
```

1. auto不会影响编译速度，甚至会加快编译速度。因为编译器在处理XX a = b时，当XX是传统类型时，编译期需要检查b的类型是否可以转化为XX。当XX为auto时，编译期可以按照b的类型直接给定变量a的类型，所以效率相差不大，甚至反而还有提升。
2. （*）最重要的一点，就是auto不要滥用，对于一些自己不明确的地方不要乱用auto，否则很可能出现事与愿违的结果，使用类型应该安全为先。
3. （*）auto主要用在与模板相关的代码中，一些简单的变量使用模板常常导致可读性下降，经验不足还会导致安全性问题。

------

### (*)静态变量，指针和引用

变量的存储位置有三种，分别是静态变量区，栈区，堆区。

**1.**静态变量区在编译时就已经确定地址，存储全局变量与静态变量。

演示

```c++
#include<iostream>

unsigned g_i = 0;//全局变量，在程序编译时已经初始化
unsigned test()
{
	static unsigned callCount = 0;//这行代码在编译时已经初始化，直接执行下一行
	return ++callCount;//统计函数调用次数
}
int main()
{
	test();
	test();
	unsigned testFuncCallCount = test();
	++g_i;//程序运行时执行
	std::cout << testFuncCallCount << std::endl;
	return 0;
}
```

**2.**指针都是存储在栈上和堆上，不管在栈上还是堆上，都一定有一个地址。

本质上说，指针和普通变量没有区别。

在32位系统中，int变量和指针都是32位。指针必须和“&”，“*”这两个符号一起使用才有意义。

&a代表的a这个变量的地址，a代表的a对应地址存储的值，*a代表对应地址存储的值作为地址对应的值。

所以指针才可以灵活的操作内存，但这也带来了严重的副作用，比如指针加加减减就可以操作内存，所以引用被发明了，引用就是作用阉割的指针（可以视为“类型*const”，所以引用必须上来就赋初值，不能设置为空），编译器不将其视作对象，操作引用相当于操作引用指向的对象。也就从根本是杜绝了引用篡改内存的能力。

```c++
int i=20;
int& refI=i;//类似于int* const pi=&i;
```

------

### (**)左值，右值，左值引用，右值引用

**1.左值和右值**

C++任何一个对象要么是左值，要么是右值。比如int i = 10，i和10都是对象

**左值：**拥有地址属性的对象就叫左值，左值可以放在等号右边，也可以放在等号左边

**右值：**不是左值的对象就是右值。无法操作地址属性的对象就是右值。比如临时对象，就都是右值，临时对象的地址属性无法使用。注意：左值也可以放在“=”右面，但右值绝对不可以放在等号左面

演示

```c++
int i=10;
int i2=i+1;//i+1为临时对象，有地址但无法使用地址
++i；//左值 i++为右值
```

**2.引用的分类**

(1) 普通左值引用：就是一个对象的别名，只能绑定左值，无法绑定常量对象。

(2) const左值引用：可以对常量起别名，可以绑定左值和右值

(3) 只能绑定右值的引用。

(4) 万能引用

```c++
int i=100;
int& refi=i;//只能绑定左值
const int& i;//绑定左值
const int& (i+1);//绑定右值
int&& rrefi=(i+1);//右值引用
int&& rrefi=i++;//右值引用
```

------

### (**)move函数，临时对象

**1.move函数**

(1) 右值看重对象的值而不考虑地址，move函数可以对一个左值使用，使操作系统不再在意其地址属性，将其完全视作一个右值。

(2) move函数让操作的对象失去了地址属性，**所以我们有义务保证以后不再使用该变量的地址属性，简单来说就是不再使用该变量，因为左值对象的地址是其使用时无法绕过的属性。**

```c++
int i=0;
int&& rrefi=std::move(i);//std::move(i)整体是个右值，i能继续赋值
```

**2.临时对象**

**右值都是不体现地址的对象。那么，还有什么能比临时对象更加没有地址属性呢？右值引用主要负责处理的就是临时对象。**

程序执行时生成的中间对象就是临时对象，注意，所有的临时对象都是右值对象，因为临时对象产生后很快就可能被销毁，使用的是它的值属性。

```c++
#include<iostream>
int getI()
{
    return 10;//return 的是一个临时对象，所有的临时对象都是右值
}
int main()
{
    int i=10;
    int&& rrefi=getI();//接收不到就会销毁
}
```

------

### (**)可调用对象

如果一个对象可以使用调用运算符“()”，()里面可以放参数，这个对象就是可调用对象。

**1.函数：**函数自然可以调用()运算符，是最典型的可调用对象

```c++
#include<iostream>
void test(int i)
{
	std::cout << i << std::endl;
	std::cout << "hello world" << std::endl;

}
using pf_type = void(*)(int);
void myFunc(pf_type pf, int i)
{
	pf(i);
}
int main()
{
myFunc(test,200);
return 0;
}
```



**2.仿函数：**具有operator()函数的类对象，此时类对象可以当做函数使用，因此称为仿函数。

```c++
#include<iostream>
class Test
{
public:
	void operator()(int i)
	{
		std::cout << i << std::endl;
		std::cout << "void operator()(int i)" << std::endl;
	}
};

int main()
{
	Test t;
	t(20);
    return 0;
}
```



**3.lambda表达式**

就是匿名函数，普通的函数在使用前需要找个地方将这个函数定义，于是C++提供了lambda表达式，需要函数时直接在需要的地方写一个lambda表达式，省去了定义函数的过程，增加开发效率。

**注意：lambda表达式很重要，**现代C++程序中，lambda表达式是大量使用的

 ```c++
 #include<iostream>
 int main()
 {
 	int i = 0;
 	[i] (int elem) {
 		std::cout << i << std::endl;
 		std::cout << elem << std::endl;
 		std::cout << "hello world" << std::endl;
 	}(200);
     return 0;
 }
 ```



**lambda表达式的格式**最少是“[] {}”，完整的格式为“[] () ->ret {}”。

lambda各组件介绍

lambda各个组件介绍

**1.**[]代表捕获列表：表示lambda表达式可以访问前文的哪些变量。

(1) []表示不捕获任何变量。

(2) [=]：表示按值捕获所有变量。

(3) [&]：表示按照引用捕获所有变量。

=，&也可以混合使用，比如

(4) [=, &i]：表示变量i用引用传递，除i的所有变量用值传递。

(5) [&, i]：表示变量i用值传递，除i的所有变量用引用传递。

当然，也可以捕获单独的变量

(6) [i]：表示以值传递的形式捕获i

(7) [&i]：表示以引用传递的方式捕获i

啊，part1结束，下周一定减少划水时间，多听课

<img src="https://cdn.jsdelivr.net/gh/jangfan/picb@main/屏幕截图 2023-03-06 160805.png" style="zoom:33%;" />

