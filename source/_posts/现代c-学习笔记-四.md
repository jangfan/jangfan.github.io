---
title: 现代c++学习笔记(四)
date: 2023-03-22 09:43:57
tags: c++
categories: 学习笔记
---

# 模板与泛型编程

## 模板介绍，类模板与模板实现原理

**1.** **模板的重要性：模板是C++最重要的模块之一，很多人对模板的重视不够，这一章一定要好好学，所有课时都是重点。**

**C++的三大模块，面向过程，面向对象，模板与泛型。面向过程就是C语言，面向对象就是类，现在轮到模板与泛型了。**

**2.** **模板的介绍：**

(1) 模板能够实现一些其他语法难以实现的功能，但是理解起来会更加困难，容易导致新手摸不着头脑。

(2) 模板分为类模板和函数模板，函数模板又分为普通函数模板和成员函数模板。

 **3.** **类模板基础：**

类模板的写法与使用十分固定

 ```c++
 #pragma once
 template<typename T>
 class MyArray
 {
 	using iterator = T*;//定义新的类型
 	using const_iterator = const T*;
 public:
 	MyArray(size_t count);
 	~MyArray();
 	//{ 写在类里面
 	//	if (count)
 	//	{
 	//		data = new T[count]();
 	//	}
 	//	else
 	//	{
 	//		data = nullptr;
 	//	}
 	//}
 
 //{//类内定义
 //	if (data)
 //	{
 //		delete[] data;
 //	}
 //}
 	iterator begin()const
 	{
 		return data;
 	}
 	const_iterator cbegin()const
 	{
 		return data;
 	}
 
 private:
 	T* data;//在类中复杂情况可以用智能指针，在老版本不支持创建数组
 };
 template<typename T>//写在类外面，一般都在类外定义
 MyArray<T>::MyArray(size_t count)
 {
 	if (count)
 	{
 		data = new T[count]();
 	}
 	else
 	{
 		data = nullptr;
 	}
 }
 template<typename T>////类外定义需要先把模板头写上去
 MyArray<T>::~MyArray()
 {
 	if (data)
 	{
 		delete[] data;
 	}
 }
 template<typename T>//begin的类外定义
 typename MyArray<T>::iterator MyArray<T>::begin() const//::前面的名称一定是类名或者明明空间
 {
 	return data;
 }
 ```

**注意，这段代码非常有代表性，在下一课补完后，一定要掌握，多看几遍。**

**4.** **模板的实现原理：**

模板需要编译两次，在第一次编译时仅仅检查最基本的语法，比如括号是否匹配。等函数真正被调用时，才会真正生成需要的类或函数。所以这直接导致了一个结果，就是不论是模板类还是模板函数，声明与实现都必须放在同一个文件中。因为在程序在编译期就必须知道函数的具体实现过程。如果实现和声明分文件编写，需要在链接时才可以看到函数的具体实现过程，这当然会报错。

 于是人们发明了.hpp文件来存放模板这种声明与实现在同一文件的情况。

## (*)initializer_list与typename

1.initializer_list的用法

(1) initializer_list介绍：initializer_list其实就是初始化列表，我们可以用初始化列表初始化各种容器，比如“vector”，“数组”。

 2.typename的用法

(1) 在定义模板时表示这个一个待定的类型

(2) 在类外表明自定义类型时使用

```c++
#.hpp
#pragma once
#include<type_traits>//萃取技术判断是否是指针
template<typename T>//模板特化
struct get_type
{
	using type = T;
};
template<typename T>
struct get_type<T*>
{
	using type = T;
};
template<typename T>
class MyArray
{
	using iterator = T*;//定义新的类型
	using const_iterator = const T*;
public:
	MyArray(size_t count);
	MyArray(const std::initializer_list<T>& list);
	#pragma once
template<typename T>
class MyArray
{
	using iterator = T*;//定义新的类型
	using const_iterator = const T*;
public:
	
	MyArray(const std::initializer_list<T>& list);
	MyArray(std::initializer_list<T>&& list);
	MyArray(size_t count);
	~MyArray();

	iterator begin()const
	{
		return data;
	}
	const_iterator cbegin()const
	{
		return data;
	}

private:
	T* data;//在类中复杂情况可以用智能指针，在老版本不支持创建数组
};
template<typename T>//写在类外面，一般都在类外定义
MyArray<T>::MyArray(size_t count)
{
	if (count)
	{
		data = new T[count]();
	}
	else
	{
		data = nullptr;
	}
}
template<typename T>////类外定义需要先把模板头写上去
MyArray<T>::~MyArray()
{
	if (data)
	{
		delete[] data;
	}
}
template<typename T>//begin的类外定义
typename MyArray<T>::iterator MyArray<T>::begin() const//::前面的名称一定是类名或者明明空间
{
	return data;
}
	~MyArray();
	iterator begin()const
	{
		return data;
	}
	const_iterator cbegin()const
	{
		return data;
	}
	T& operator[](unsigned count)const
	{
		return data[count];//重载
	}

private:
	std::vector<T> data;
};
template<typename T>//写在类外面，一般都在类外定义
MyArray<T>::MyArray(size_t count)
{
	if (count)
	{
		data = new T[count]();
	}
	else
	{
		data = nullptr;
	}
}
template<typename T>////类外定义需要先把模板头写上去
MyArray<T>::~MyArray()
{
	if (data)
	{
		delete[] data;
	}
}
template<typename T>
MyArray<T>::MyArray(const std::initializer_list<T>& list)
{
	if (list.size())
	{
		unsigned count = 0;
		data = new T[list.size()]();
		if (std::is_pointer<T>::value)
		{
			for (auto elem : list)
			{
				data[count++] = new typename get_type<T>::type(*elem);//相当于两层指针，在删除时，只删除第一层，会出现内存泄漏
			}
		}
		else
		{
			for (const auto& elem; list)
			{
				data[count++] = elem;//存在bug，如果存放的是指针，很变成浅复制
			}
		}
	}
	else
	{
		data = nullptr;
	}
}
template<typename T>
MyArray<T>::MyArray(std::initializer_list<T>&& list)//右值引用
{
	if (list.size())
	{
		unsigned count = 0;
		data = new T[list.size()]();
		for (const auto& elem; list)
		{
			data[count++] = elem;//存在bug，如果存放的是指针，很变成浅复制
		};
	}
	else
	{
		data = nullptr;
	}
}
```



```c++
#.cpp
#include<iostream>
#include<vld.h>
#include<vector>
#include"myArray.hpp"

int main()
{
	//std::initializer_list<int> iList{ 1,2,3,4,5 };
	//std::vector<int> ivec(iList);//左值类型的初始化,右值类型初始化ivec{1,2,3,4,5}
	int i1 = 10;
	int i2 = 20;
	int i3 = 30;
	int i4 = 40;
	std::initializer_list<int*> iList{ &i1,&i2,&i3,&i4 };
	MyArray<int*> arrayPi(iList);
	for (unsigned i = 0; i < 4; ++i)
	{
		std::cout << arrayPi[i] << std::endl;
	}
	return 0;
}
```





**在C++的早期版本，为了减少关键字数量，用class来表示模板的参数，但是后来因为第二个原因，不得不引入typename关键字。**

## (*)函数模板，成员函数模板

1.普通函数模板的写法与类模板类似

```c++
#include<iostream>
#include<vld.h>
#include<vector>
#include<memory>
#include"myArray.hpp"
namespace mystd
{
	template<typename iter_type ,typename func_type>//普通函数模板
	void for_each(iter_type first, iter_type last, func_type func)
	{
		for (auto iter = first; iter != last; ++iter)
		{
			func(*iter);
		}
	}
 }
int main() {
	std::vector<int> ivec{ 1,2,3,4,5 };
	mystd::for_each(ivec.begin(), ivec.end(), [](int& elem) {
		++elem;
		});
	for (int elem : ivec)
	{
		std::cout << elem << std::endl;
	}
	return 0;
 }
```

 

**在现代C++中，函数模板一直普遍使用，一定要掌握。**

 2.成员函数模板

 ```c++
 #include<iostream>
 #include<vld.h>
 #include<vector>
 #include<memory>
 #include"myArray.hpp"
 
 namespace mystd
 {
 	template<typename iter_type, typename func_type>
 	void for_each(iter_type first, iter_type last, func_type func)
 	{
 		for (auto iter = first; iter != last; ++iter)
 		{
 			func(*iter);
 		}
 	}
 	template<typename T>//成员函数模板
 	class MyVector
 	{
 	public:
 		template<typename T2>
 		void outPut(const T2& elem);
 
 	};
 	template<typename T>
 	template<typename T2>
 	void MyVector<T>::outPut(const T2& elem)
 	{
 		std::cout << elem << std::endl;
 	}
 }
 
 
 int main() {
 	mystd::MyVector<int> myVec;
 	myVec.outPut<int>(20);
 	return 0;
  }
 ```



成员函数模板使用情况也不少，需要掌握的

## (*)默认模板参数

默认模板参数：

(1) 默认模板参数是一个经常使用的特性，比如在定义vector对象时，我们就可以使用		默认分配器。

<img src="https://cdn.jsdelivr.net/gh/jangfan/picb@main/屏幕截图 2023-03-29 135107.png"  />

(2) 模板参数就和普通函数的默认参数一样，一旦一个参数有了默认参数，它之后的参	  数都必须有默认参数

(3) 函数模板使用默认模板参数

(2) 类模板使用模板参数

 ```c++
 #include<iostream>
 #include<vld.h>
 #include<vector>
 #include<functional>
 #include<algorithm>
 #include<memory>
 #include"myArray.hpp"
 
 namespace mystd
 {
 	using void_int_func_type = std::function<void(int&)>;
 	template<typename iter_type, typename func_type=void_int_func_type>//函数模板使用模板参数,typename这里有默认值，下边都要有默认值
 	void for_each(iter_type first, iter_type last, func_type func = [](int& elem){
 			++elem;
 		})
 	{
 		for (auto iter = first; iter != last; ++iter)
 		{
 			func(*iter);
 		}
 	}
 	template<typename T, typename allocator = std::allocator<T>>//类模板使用默认模板参数
 	class MyVector
 	{
 	public:
 		template<typename T2>
 		void outPut(const T2& elem);
 
 	};
 	template<typename T,typename allocator>
 	template<typename T2>
 	void MyVector<T,allocator>::outPut(const T2& elem)
 	{
 		std::cout << elem << std::endl;
 	}
 }
 
 
 int main() 
 {
 	std::vector<int> ivec{ 1,2,3,4,5 };
 	mystd::for_each(ivec.begin(), ivec.end());
 	for (auto elem : ivec)
 	{
 		std::cout << elem << std::endl;
 	};
 	return 0;
 }
 ```

几乎stl库都在使用模板参数

## (*)模板的重载，全特化和偏特化

1.模板的重载

(1) 函数模板是可以重载的（类模板不能被重载），通过重载可以应对更加复杂的情况。比如在处理char*和string对象时，虽然都可以代表字符串，但char*在复制时直接拷贝内存效率明显更高，string就不得不依次调用构造函数了。所以在一些比较最求效率的程序中对不同的类型进行不同的处理还是非常有意义的。

 ```c++
 #include<iostream>
 
 template<typename T>
 void test(const T& parm)
 {
 	std::cout << "void test(const T& parm)" << std::endl;
 }
 template<typename T>
 void test(T* parm)//若加上const，test(&10)则会调用第一个
 {
 	std::cout << "void test（T* parm)" << std::endl;
 }
 
 void test(double parm)
 {
 	std::cout << "void test(double parm)" << std::endl;
 }
 int main()
 {
 	test(100);
 	int i = 100;
 	test(&i);
 	test(2.2);
 	return 0;
 };
 ```

```bash
void test(const T& parm)
void test（T* parm)
void test(double parm)
```



其实函数模板的重载和普通函数的重载没有什么区别。

2.模板的特化

(1) 模板特化的意义：函数模板可以重载以应对更加精细的情况。类模板不能重载，但可以特化来实现类似的功能。

(2) 模板的特化也分为两种，全特化和偏特化。模板的全特化：就是指模板的实参列表与与相应的模板参数列表一一对应。

(3) 模板的偏特化：偏特化就是介于普通模板和全特化之间，只存在部分类型明确化，而非将模板唯一化。

 ```c++
 #include<iostream>
 
 template<typename T1,typename T2>
 class Test
 {
 public:
 	Test() {
 		std::cout << "common template" << std::endl;
 	}
 };
 template<typename T1, typename T2>
 class Test<T1*, T2*>
 {
 public:
 	Test() {
 		std::cout << "point semi-template" < , std::endl;
 	}
 };
 template<typename T2>
 class Test<int, T2>//只写一部分叫偏特化
 {
 public:
 	Test() {
 		std::cout << "int ssssemi-special" << std::endl;
 	}
 };
 template<>
 class Test<int, int>
 {
 public:
 	Test() {
 		std::cout << "int,int complete special" << std::endl;
 	}
 };
 int main()
 {
 	Test<int*, int*> test;
 	return 0;
 };
 ```



**(4)** **其实对于函数模板来说，特化与重载可以理解为一个东西。**

 **总结：函数模板的重载，类模板的特化。还是比较重要的知识点，应当掌握，在一些比较复杂的程序中，模板重载与特化是经常使用的。**
