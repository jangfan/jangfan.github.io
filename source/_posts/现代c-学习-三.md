---
title: 现代c++学习笔记(三)
date: 2023-03-17 18:19:37
tags: c++
categories: 学习笔记
---

# 智能指针

## 智能指针概述

 **1.** 为什么要有智能指针：在Part2的第二节课已经讲过，直接使用new和delete运算符极其容易导致内存泄露，而且非常难以避免。于是人们发明了智能指针这种可以自动回收内存的工具。

  **2.** 智能指针一共就三种：普通的指针可以单独一个指针占用一块内存，也可以多个指针共享一块内存。

(1) 共享型智能指针：shared_ptr，同一块堆内存可以被多个shared_ptr共享。

```c++
#include<iostream>
int main()
{
	int* pi = new int(10);
	int* pi2(pi);//pi2和pi共享一段内存
	return 0;
}
```

(2) 独享型智能指针：unique_ptr，同一块堆内存只能被一个unique_ptr拥有。无法拷贝和制造

(3) 弱引用智能指针：weak_ptr，也是一种共享型智能指针，可以视为对共享型智能指针的一种补充

 **3.** **（*）智能指针注意事项：**

**智能指针和裸指针不要混用，接下来的几节课会反复强调这一点。**

## (*)shared_ptr

**1.shared_ptr的工作原理** 

 (1)我们在动态分配内存时，堆上的内存必须通过栈上的内存来寻址。也就是说栈上的指针（堆上的指针也可以指向堆内存，但终究是要通过栈来寻址的）是寻找堆内存的唯一方式。
 (2)所以我们可以给堆内存添加一个引用计数，有几个指针指向它，它的引用计数就是几，当引用计数为0时，操作系统会自动释放这块堆内存。
 **2.Shared_ptr的常用操作** 

**(1)shared_ptr的初始化** 

 ①使用new运算符初始化， 一般来说不推荐使用new进行初始化，因为C++标准提供了专门创建shared_ptr的函数“make_shared”，该函数是经过优化的，效率更高。
 ②使用make_shared函数进行初始化：  

```c++
std::shared_ptr<int>sharedI = std::make_shared<int>(100);
std::shared_ptr<int>sharedI2(sharedI);//允许多个智能指针指向同一块内存
```

注意：千万不要用裸指针初始化shared_ptr，容易出现内存泄露的问题。

 ③当然使用复制构造函数初始化也是没有问题的。
 代码演示:   

(2)shared_ptr的引用计数：  智能指针就是通过引用计数来判断释放堆内存时机的。
use_count()函数可以得到shared_ptr对象的引用计数。

```c++
#include<iostream> 
#include<vld.h> 
#include<memory> 
int main() 
{ 	
	std::shared_ptr<int>sharedI = std::make_shared<int>(100); 	
	std::cout << sharedI.use_count() << std::endl; 	
	std::shared_ptr<int>shareI2(sharedI); 	
	std::cout << sharedI.use_count() << std::endl; 	
	shareI2.reset(); 	
	std::cout << sharedI.use_count() << std::endl;
	return 0; 
}//结果输出121
```

**3.智能指针可以像普通指针那样使用，”share_ptr”早已对各种操作进行了重载，就当它是普通指针就可以了.** 

**4.Shared_ptr的常用函数 **

(3)unique函数：判断该shared_ptr对象是否独占若独占，返回true。否则返回false。

```c++
#include<iostream> 
#include<vld.h> 
#include<memory> 
int main() 
{ 	
	std::shared_ptr<int>sharedI = std::make_shared<int>(100); 	
	std::cout << sharedI.unique() << std::endl;//独占返回1 	
	std::shared_ptr<int>shareI2(sharedI); 	
	std::cout << sharedI.unique() << std::endl;//独占返回0 	
	shareI2.reset(); 	
	std::cout << sharedI.unique() << std::endl;//返回1
	return 0; 
}
```

 (4)reset函数： 

①当reset函数有参数时，改变此shared_ptr对象指向的内存。
②当reset函数无参数时，将此shared_ptr对象置空，也就是将对象内存的指针设置为nullptr。

```c++
#include<iostream> 
#include<vld.h> 
#include<memory> 
int main() 
{ 	
	std::shared_ptr<int>sharedI = std::make_shared<int>(100); 	
	std::cout << sharedI.unique() << std::endl;	
	sharedI.reset(new int(1000));//原有堆内存被释放，指向新的堆内存,写法不能写成shared_ptr的初始化
	std::shared_ptr<int>sharedI2 = std::make_shared<int>(100);
	sharedI = sharedI2;
	sharedI.reset();//置为空
	return 0; 
}
```

  

(5)get函数，强烈不推荐使用

(6)swap函数：交换两个智能指针所指向的内存

①std命名空间中全局的swap函数 

②shared_ptr类提供的swap函数

 ```c++
 #include<iostream> 
 #include<vld.h> 
 #include<memory> 
 int main() 
 { 	
 	std::shared_ptr<int>sharedI = std::make_shared<int>(100); 	
     std::shared_ptr<int>sharedI2 = std::make_shared<int>(1000);
 	sharedI.swap(sharedI2);//std::swap(sharedI,sharedI2)
 	std::cout << *sharedI << std::endl;
 	std::cout << *sharedI2 << std::endl;
 	return 0; 
 }
 ```

![](%E7%8E%B0%E4%BB%A3c-%E5%AD%A6%E4%B9%A0-%E4%B8%89/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-03-20%20165343-1679302470989-3.png)

 5.关于智能指针创建数组的问题。
```c++
std::shared_ptr<int> sharedI(new int[100]());
std::cout<<sharedI.get()[10]<<std::endl;//不能像普通数组一样使用
void myFunc(std::shared_ptr<int> sharedI)//不用写const
```

6.用智能指针作为参数传递时直接值传递就可以了。shared_ptr的大小为固定的8或16字节（也就是两倍指针的的大小，32位系统指针为4个字节，64位系统指针为8个字节，shared_ptr中就两个指针），所以直接值传递就可以了。

 shared_ptr总结：在现代程序中，当想要共享一块堆内存时，优先使用shared_ptr，可以极大的减少内存泄露的问题。

## (*)weak_ptr

**1.** **weak_ptr介绍：**

(1) 这个智能指针是在C++11的时候引入的标准库，它的出现完全是为了弥补shared_ptr天生有缺陷的问题，其实shared_ptr可以说近乎完美。

(2) 只是通过引用计数实现的方式也引来了引用成环的问题，这种问题靠它自己是没办法解决的，所以在C++11的时候将shared_ptr和weak_ptr一起引入了标准库，用来解决循环引用的问题。

```c++
#include<iostream> 
#include<vld.h> 
#include<memory> 
int main() 
{ 	
	std::shared_ptr<int>sharedI = std::make_shared<int>(100);
	std::cout << sharedI.use_count() << std::endl;
	std::weak_ptr<int>weakI(sharedI);//weak_ptr指向同一个内存不增加引用计数
	std::cout << sharedI.use_count() << std::endl;
	return 0; 
}
```

**2.** **shared_ptr的循环引用问题：**

```c++
#include<iostream> 
#include<vld.h> 
#include<memory> 
class A
{
public:
	std::weak_ptr<B> weakB;
};
class B
{
public:
	std::shared_ptr<A> sharedA;
};
int main() 
{ 	
	std::shared_ptr<A>sharedA = std::make_shared<A>();
	std::shared_ptr<B>shareB = std::make_shared<B>();
	sharedA->weakB = weakB;
	weakB->sharedA = sharedA;//如果为两个shared_ptr相互指引导致内存无法释放,所以将其中一个改为weak_ptr
	return 0; 
}
```

**3.** **weak_ptr的作用原理：**weak_ptr的对象需要绑定到shared_ptr对象上，作用原理是weak_ptr不会改变shared_ptr对象的引用计数。只要shared_ptr对象的引用计数为0，就会释放内存，weak_ptr的对象不会影响释放内存的过程。

总结：**weak_ptr使用较少，就是为了处理shared_ptr循环引用问题而设计的。**

## (*)unique_ptr

**1.** **uniqe_ptr介绍：**独占式智能指针，在使用智能指针时，我们一般优先考虑独占式智能指针，因为消耗更小。如果发现内存需要共享，那么再去使用“shared_ptr”。

**2** **unique_ptr的初始化**：和shared_ptr完全类似

(1) 使用new运算符进行初始化

(2) 使用make_unique函数进行初始化

 ```c++
 #include<iostream> 
 #include<vld.h> 
 #include<memory> 
 int main()
 {
 	std::unique_ptr<int>uniqueI2(new int(100));
 	std::unique_ptr<int> uniqueI = std::make_unique<int>(100);
 	return 0;
 
 }
 ```

**3.** **unique_ptr的常用操作**

(1) unque_ptr禁止复制构造函数，也禁止赋值运算符的重载。否则独占便毫无意义。

(2) unqiue_ptr允许移动构造，移动赋值。移动语义代表之前的对象已经失去了意义，移动操作自然不影响独占的特性。

```c++
#include<iostream> 
#include<vld.h> 
#include<memory> 
int main()
{
	std::unique_ptr<int> uniqueI = std::make_unique<int>(100);
    std::unique_ptr<int>uniqueI2=std::make_unique<int>(200);
    uniqueI2=std::move(uniqueI);
	return 0;
}
```

(3) reset函数：

① 不带参数的情况下：释放智能指针的对象，并将智能指针置空。

② 带参数的情况下：释放智能指针的对象，并将智能指针指向新的对象。

 **和shared_ptr使用方法一样**

**4.** **将unque_ptr的对象转化为shared_ptr对象，**当unique_ptr的对象为一个右值时，就可以将该对象转化为shared_ptr的对象。

**这个使用的并不多，需要将独占式指针转化为共享式指针常常是因为先前设计失误。**

**注意：shared_ptr对象无法转化为unique_ptr对象。**

```c++
#include<iostream> 
#include<vld.h> 
#include<memory> 
void myfunc(std::unique_ptr<int> uniqueI)//需要共享操作时
{
    std::shared_ptr<int>sharedI(std::move(uniqueI));//转换
}
int main()
{
	std::unique_ptr<int> uniqueI = std::make_unique<int>(100);
    std::unique_ptr<int>uniqueI2=std::make_unique<int>(200);
    uniqueI2=std::move(uniqueI);
	return 0;
}
```



## (**)只能指针适用范围

**1.** **能使用智能指针就尽量使用智能指针，那么哪些情况属于不能使用智能指针的情况  呢？**

 有些函数必须使用C语言的指针，这些函数又没有替代，这种情况下，才使用普通的指针，其它情况一律使用智能指针。

 必须使用C语言指针的情况包括：

**（1）** **网络传输函数**，比如windows下的send，recv函数，只能使用c语言指针，无法替代.

**（2）** **c语言的文件操作部分**。这方面C++已经有了替代品，C++的文件操作完全支持智能指针，**所以在做大型项目时，推荐使用C++的文件操作功能。**

**除了以上两种情况，剩下的均推荐使用智能指针。**

**2.** **我们应该使用哪个智能指针呢？**

**(1)** **优先使用unique_ptr，内存需要共享时再使用shared_ptr。**

**当使用shared_ptr时，如果出现了循环引用的情况，再去考虑使用weak_ptr**
