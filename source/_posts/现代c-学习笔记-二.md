---
title: 现代c++学习笔记(二)
date: 2023-03-13 15:53:28
tags: c++
categories: 学习笔记
---

## 类

类的权限修饰：c++通过 **public、protected、private** 三个关键字来控制成员变量和成员函数的访问权限，它们分别表示公有的、受保护的、私有的，被称为成员访问限定符。所谓访问权限，就是你能不能使用该类中的成员。

在类的内部（定义类的代码内部），无论成员被声明为 public、protected 还是 private，都是可以互相访问的，没有访问权限的限制。在类的外部（定义类的代码之外），只能通过对象访问成员，并且通过对象只能访问 public 属性的成员，不能访问 private、protected 属性的成员。

## (*)类介绍，构造函数，析构函数

**1.类介绍：**

(1) 对面向对象和面向过程的理解

① 面向对象和面向过程是一个相对的概念。

② 面向过程是按照计算机的工作逻辑来编码的方式，最典型的面向过程的语言就是c语言了，c语言直接对应汇编，汇编又对应电路。

③ 面向对象则是按照人类的思维来编码的一种方式，C++就完全支持面向对象功能，可以按照人类的思维来处理问题。

④ 举个例子，要把大象装冰箱，按照人类的思路自然是分三步，打开冰箱，将大象装进去，关上冰箱。

要实现这三步，我们就要首先有人，冰箱这两个对象。人有给冰箱发指令的能	力，冰箱有能够接受指令并打开或关闭门的能力。

```c++
//c++
#inlcude<iostream>
class IceChest;
class Person
{
    public:
    void openIceChest(const IceChest& iceChest)
    {
        
    }
}
class IceChest:
{
    public:
        void openDoor()
        {
            
        }
        void closeDoor()
        {
        
        }
};
//c语言
struct Person
{
    
};
struct IceChest
{
    
};
void personOpenIceChest(const Person& person,const IceChest& iceChest)
void personCloseIceChest(const Person& person,const IceChest& iceChest)
```

但是从计算机的角度讲，计算机只能定义一个叫做人和冰箱的结构体。人有手这个部位，冰箱有门这个部位。然后从天而降一个函数，是这个函数让手打开了冰箱，又是另一个函数让大象进去，再是另一个函数让冰箱门关上。

 从开发者的角度讲，面向对象显然更利于程序设计。用面向过程的开发方式，程序一旦大了，各种从天而降的函数会非常繁琐，一些用纯c写的大型程序，实际上也是模拟了面向对象的方式。

 那么，如何用面向过程的c语言模拟出面向对象的能力呢？类就诞生了，在类中可以定义专属于类的函数，让类有了自己的动作。回到那个例子，人的类有了让冰箱开门的能力，冰箱有了让人打开的能力，不再需要天降神秘力量了，hh

 **2.构造函数**

 类是通过面向过程的机器实现的，类相当于定义了一个新类型，该类型生成在堆或栈上的对象时内存排布和c语言相同。但是c++规定，**C++有在类对象创建时就在对应内存将数据初始化的能力**，这就是构造函数。

```c++
#include<isotream>
struct CPPtest
{
    public:
    CPPTest(int i_,int i2_) :i(i_),i2(i2_){}//c++可以在类中构造函数，将参数对应内存初始化;若没有，则自动生成，但是没什么用
    int i;
    int i2;
};
int main(){
    CPPtest cppTest(1,2);//初始化i,i2
    std::cout<<cppTest.i<<std::endl;
    std::cout<<cppTest.i2<<std::endl;
    return 0;
}
```



构造函数有以下类型。

1. 普通构造函数

2. 复制构造函数：用另一个对象来初始化对象对应的内存

   ```c++
   CPPTest(const CPPTest& cppTest) :i(cppTest.i), i2(cppTest.i2){}
   ```

3. 移动构造函数：也是用另一个对象来初始化对象，具体内容会在Part3第13节详细讲解。

4. 默认构造函数：当类没有任何构造函数时，编译期会为该类生成一个默认的的构造函数，在最普通的类中，默认构造函数什么都没做，对象对应的内存没有被初始化。

```c++
CPPTest(){};//作用和构造函数一样
```

构造函数就是C++提供的**必须有的**在对象创建时初始化对象的方法，（默认的什么都不做也是一种初始化的方式）

**3.析构函数**

介绍:

类对象被销毁时，就会调用析构函数。栈上对象的销毁时机就是函数栈销毁时,堆上的对象销毁时机就是该堆内存被手动释放时，如果用new申请的这块堆内存，那调用delete销毁这块内存时就会调用析构函数。

```c++
#include<isotream>
struct CPPTest
{
    public:
    CPPTest(int i_,int i2_) :i(i_),i2(i2_){}
    CPPTest(const CPPTest& cppTest) :i(cppTest.i), i2(cppTest.i2){}
    int i;
    int i2;
    ~CPPTest()//如没有会自动添加
    {
        
    }
};
int main(){
    CPPTest cppTest(1,2);//初始化i,i2
    std::cout<<cppTest.i<<std::endl;
    std::cout<<cppTest.i2<<std::endl;
    CPPTest* pCppTest=new CppTest(1.2);//调用在堆上，自动销毁
    delete PCppTest;
    return 0;
}
```

但是再某些情况下，析构函数必须要干活

```c++
#include<isotream>
struct CPPTest
{
    public:
    CPPTest(int i_,int i2_) :i(i_),i2(i2_),pi(new int(i3)){}
    CPPTest(const CPPTest& cppTest) :i(cppTest.i), i2(cppTest.i2),pi(new int(*cppTest.pi)){}
    int i;
    int i2;
    int *pi;
    ~CPPTest()
    {
        delete pi;
    }//当栈上pi指向堆上的i3，如果析构函数中没有delete，那么只会把栈上的释放，堆上的数据没有释放，会造成内存泄漏
};
int main(){
}
```

总结，当类对象销毁时有一些我们必须手动操作的步骤时，析构函数就派上了用场。所以，几乎所有的类我们都要写构造函数，析构函数却未必需要

## (*)this,常成员函数与常对象

**1.this关键字：**

(1) this是什么：

① 编译器将this解释为指向函数所作用的对象的指针，这句话新手有些不好理解，用代码演示一下就好说了。C++类的本质就是C语言的结构体外加几个类外的函数，C++最后都要转化为C语言来实现，类外的函数就是通过this来指向这个类的。

 ```c++
 #include<iostream>
 clacc Test
 {
     public:
        Test(const std::string& name_, unsigned old_);
        ~Test();
     void output()const
     {
         std::cout<<"name="<<this->name<<"     old="<<this->old<<std::endl;
     }
     private:
        std::string name;
        unsigned old;
 };
 Test::Test(const std::string& name_,unsigned old_):name(name_),old(old_)
 {
 }
 Test::~Test()
 {
 }
 
 int main()
 {
     Test test("fanfan",20);
     test.outPut();
     return 0;
 }
 ```



② 当然，这么说并非完全准确，this是一个关键字，只是我们将它当做指针理解罢了。

 this有很多功能是单纯的指针无法满足的。比如每个类函数的参数根本没有名叫this的指针。这不过是编译器赋予的功能罢了。

```c++
#include<iostream>
clacc Test
{
    public:
       Test(const std::string& name_, unsigned old_);
       ~Test();
    void output()const
    {
        std::cout<<"name="<<this.name<<"     old="<<this.old<<std::endl;
    }
    private:
       std::string name;
       unsigned old;
};
Test::Test(const std::string& name_,unsigned old_):name(name_),old(old_)
{
}
Test::~Test()
{
}
void (Test::* pf)()=&Test::output;//空参数的指针能指向类的成员函数，说明类中没有this这个形参变量
int main()
{
    Test test("fanfan",20);
    (test.*pf)();//结果一样
    return 0;
}
```

**2.常成员函数和常对象**

某位大佬说：“常成员函数和常对象很多人并不在意，确实，都写普通变量也可以。但是，我还是要提一点，在大型程序中，尽量加上const关键字可以减少很多不必要的错误。**这一点，开发过大型程序的人应该深有体会，没开发过大型程序的人也不必在意，记住多用const，这是一个很好的习惯**。“

(1) 常成员函数就是无法修改成员变量的函数。可以理解为将this指针指向对象用const修饰的函数。(例子在本节的第一个代码演示)

常对象就是用const修饰的对象，定义好之后就再也不需要更改成员变量的值了。

(2) 常成员函数注意事项：

因为类的成员函数已经将this指针省略了，只能在函数后面加const关键字来实现无法修改类成员变量的功能了

① 注意：常函数无法调用了普通函数，无意义。

**②** **成员函数能写作常成员函数就尽量写作常成员函数，可以减少出错几率。**

③ 同名的常成员函数和普通成员函数是可以重载的，常量对象会优先调用常成员函数，普通对象会优先调用普通成员函数。

(3) 常对象注意事项：

① 常对象不能调用普通函数

② 常函数在大型程序中真的很重要，很多时候我们都需要创建好就不再改变的对象

**大佬强调：常对象和常函数要多用**

##  **inline，mutable，default，delete**

inline,mutable知道就行，default和delete需要掌握

**1.inline关键字**

(1) inline关键字的有什么作用：

① 在函数声明或定义中函数返回类型前加上关键字inline就可以把函数指定为**内联函数**。关键字inline必须与函数定义放在一起才能使函数成为内联，仅仅将inline放在函数声明前不起任何作用。

 ② 内联函数的作用，普通函数在调用时需要给函数分配栈空间以供函数执行，压栈等操作会影响成员运行效率，于是C++提供了**内联函数将函数体放到需要调用函数的地方**，用空间换效率。

```c++
#include<iostream>
class Test
{
  public:  
    inline void outPut();//默认有inline，后边的不加，这里加了也是无效的
};
inline void Test::outPut()
{
    std::cout<<"hello world"<<std::endl;
}
int main()
{
    Test test;
    test.outPut();
    return 0;
}
```



 (2) inline关键字的注意事项：inline关键字只是一个建议，开发者建议编译器将成员函数当做内联函数，一般适合搞内联的情况编译器都会采纳建议。（牺牲空间换取时间）

 (3) **总结:**使用inline关键字就是一种提高效率，但加大编译后文件大小的方式，现在随着硬件性能的提高，inline关键字用的越来越少了。

**2.mutable关键字**

(1) mutable关键字的作用：

① Mutable意为可变的，与const相对，被mutable修饰的成员变量，永远处于可变的状态，即便处于一个常函数中，该变量也可以被更改。

```c++
#include<iostream>
class Test
{
  public:
    void outPut() const;
    mutable unsigned outPutCallCount=0;
   };
void Test::outPut()const
{
    ++outPutCallCount;
    std::cout<<"hello world"<<std::endl;
}
int main()
{
    Test test;
    test.outPut();
    std::cout<<test.outPutCallCount<<std::endl;
    return 0;
}
```

这个关键字在现代C++中使用情况并不多，一般来说只有在统计函数调用次数时才会用到。

 (2) mutable关键字的注意事项

① mutable是一种万不得已的写法，一个程序不得不使用mutable关键字时，可以认为这部分程序是一个糟糕的设计。

② mutable不能修饰**静态成员变量和常成员变量**。

(3) **总结：**mutable关键字是一种没有办法的办法，设计时应该尽量避免，只有在统计函数调用次数这类情况下才推荐使用。

**3.default关键字**

(1) default关键字的作用：

① 在编译时不会生成默认构造函数时便于书写。

② 也可以对默认复制构造函数，默认的赋值运算符和默认的析构函数使用，表示使用的是系统默认提供的函数，这样可以使代码更加明显。

```c++
#include<iostream>
class Test
{
	Test(unsigned old_):old(old_){}
    Test() = default;//以前Test(){}
	Test(const Test& test)=default;//使用系统默认
    Test& operator=(const Test& test)=default;
    ~Test()=default;
	unsigned old;
};
```

**③** 现代C++中，哪怕没有构造函数，也推荐将构造函数用default关键字标记，可以让代码看起来更加直观，方便。

总结：default关键字还是推荐使用的，在现代C++代码中，如果需要使用一些默认的函数，推荐用default标记出来。

**4.delete关键字**

(1) Delete关键字的作用：C++会为程序生成默认构造函数，默认复制构造函数，默认重载赋值运算符。

```c++
#include<iostream>
class Test
{
	//没有Test(unsigned old_):old(old_){}，还不想默认生成
    Test()=delete；
    //Test& operator=(const Test& test)=default;使用系统默认，不想使用就将default换成delete
    ~Test()=default;//一般不会delete
	unsigned old;
};
```



 在很多情况下，我们并不希望这些默认的函数被生成，在C++11以前，只能有将此函数声明为私有函数或是将函数只声明不定义两种方式。

 C++11于是提供了delete关键字，只要在函数最后加上“=delete”就可以明确告诉编译期不要默认生成该函数。

 总结：delete关键字还是推荐使用的，在现代C++代码中，如果不希望一些函数默认生成，就用delete表示，这个功能还是很有用的，比如在单例模式中。

## 友元类和友元函数

**1.**介绍：友元就是可以让另一个类或函数访问私有成员的简单写法。

```c++
#include<iostream>
class Test
{
    friend class Test2;//友元类
    friend void outPut(const Test& test);//友元函数
 private:   
    std::string name;
    unsigned old;
}
class Test2
{
 public:
    void outPut(const Test& test)
    {
        std::cont<<test.name<<"  "<<test.old<<std::endl;//没有friend，类不能访问Test中的变量
    }
}
 void outPut(const Test& test)//上接友元函数
    {
        std::cont<<test.name<<"  "<<test.old<<std::endl;//没有friend，函数不能访问Test中的变量
    }
int main()
{
    
}
```

 **2.**注意：

(1) 友元会破坏**封装性**，一般不推荐使用，所带来的方便写几个接口函数就解决了。

```c++
class Test
{
    std::string getName()const{return name;};//
    unsigned getOld()const{return old;};//接口
 private:  
    std::string name;
    unsigned old;
}//同样效果
```

**(2)** 某些运算符的**重载**必须用到友元的功能，这才是友元的真正用途。

 **3.**大佬说：友元平常并不推荐使用，新手不要再纠结友元的语法了，只要可以用友元写出必须用友元的重载运算符就可以了。

## (**)重载运算符

**重载运算符在整个C++中拥有非常重要的地位**

**1.重载运算符的作用：**

(1) 很多时候我们想让类对象也能像基础类型的对象一样进行作基础操作，比如“+”，“-”，“*”，“\”，也可以使用某些运算符“=”，“()”，“[]”,“<<”，“>>”。但是一般的类即使编译器可以识别这些运算符，类对象也无法对这些运算符做出应对，我们必须对类对象定义处理这些运算符的方式。

(2) C++提供了定义这些行为的方式，就是**operator 运算符**来定义运算符的行为，operator是一个关键字，告诉编译器我要重载运算符了。

 **2.注意：**

(1) 我们只能重载C++已有的运算符，所有无法将“**”这个运算符定义为指数的形式，因为C++根本没有“**”这个运算符。

(2) C++重载运算符不能改变运算符的**元数**，“元数”这个概念就是指一个运算符对应的对象数量，比如“+”必须为“a + b”，也就是说“+”必须有两个对象，那么“+”就是二元运算符。比如“++”运算符，必须写为“a++”，也就是一元运算符。

 **3.重载运算符举例**

(1) 一元运算符重载

① “++”，“--”

```c++
#include<iosteam>
class Test
{
    public:
    void opterator++()
    {
        ++count;
    }
    void opterator--()
    {
        --count;
    }
    unsigned count = 0;
}
int main()
{
    Test test;
    ++test;
    std::cout<<test.count<<std::endl;
    --test;
    std::cout<<test.count<<std::endl;
    return 0;
}
```

② “[]”

```c++
#include<iosteam>
#include<vector>//容器
class Test
{
    public:
    int operator[](unsigned i)const
    {
        if(i>0&&i<ivec.size)
            return ivec[i];
    }
    std::vector<int> ivec{1,2,3,4,5,6}
}
int main()
{
    Test test;
    std::cout<<test[3]<<std::endl;
}
```

③ “()”

```c++
#include<iosteam>
class Test
{
    public:
    void operator()()const
    {
        std::cout<<"hello world"<<std::endl;
    }
    void operator()(const std::string& str)const//可以重载
    {
        std::cout<<str<<std::endl;
    }
    
}
int main()
{
    Test test;
    test();//输出hello world
    test("abcde")//输出abcde
        return 0;
}
```

④ “<<”，“>>”

```c++
#include<iosteam>
class Test
{//记住怎么写
  friend std::ostream& operator<< (std::ostream& os,const Test& test);
  friend std::istream& operator>> (std::istream& os,Test& test);
}
std::ostream& operator<< (std::ostream& os,const Test& test)
{
    os<<test.name<<std::endl;
    return os;//返回ostream的引用
}
std::istream& operator>> (std::istream& os,Test& test)
{
    is >>test.name;
    return is;
}
int main()
{
    Test test;
    std::cin>>test;
    std::cout<<test<<std::endl;
    return 0;
}
```



(2) 二元运算符重载

① “+”，“-”，“*”，“/”

```c++
#include<iosteam>
class Test
{
    public:
    Test opreator+(const Test& test)
    {
        count +=test.count;
        return *this;
    }
    unsigned count = 0;
    std::string name;
}
int main()
{
    Test test;
    std::cout<<test.count<<std::endl;//输出0
    Test test2;
    test2.count=20;
    Test test3=+test2;
    std::test3<<test3.count<<std::endl;//输出20
    return 0;
}
```

② “=”，

```c++
#include<iosteam>
class Test
{
    Test& opreator=(const Test& test)//使用引用的原因是返回的还是自己
    {
        if(this==&test)//地址相同
        {
            return *this;
        }
        count =test.count;
        name=test.name;
    }
    //Test& opterator= (const Test& test)=default 默认重载
    unsigned count = 0;
    std::string name;
    return *this；
}
int main()
{
    Test test;
    Test test2;
    test=test2;//返回是是自己，所以要用引用
    return 0;
}
```

③ “>”，“<”，“==”

```c++
bool operator< (const Test& test)
{
    return count<test.count;
}
```

 至于唯一的三元运算符“?:”，不能重载

(3) 类类型转化运算符：“operator 类型”

(4) 特殊的运算符：new，delete，new[]，delete[]

 注意：“=”类会默认进行重载，如果不需要可以用“delete关键字进行修饰”。

 **总结：重载运算符非常重要，C++类中几乎都要定义各种各种的重载运算符。**

## (*)普通继承及其实现原理

**C++面向对象的三大特性：分装，继承，多态。分装就是类的权限管理，很简单，就不讲了。继承这节课讲，继承很重要，有些地方也是需要重点理解的。**

 **1.**C++继承介绍：C++非继承的类相互是没有关联性的，假设现在需要设计医生，教师，公务员三个类，需要定义很多重复的内容而且相互没有关联，调用也没有规律。如果这还算好，那一个游戏有几千件物品，调用时也要写几千个函数。这太要命了。于是继承能力就应运而生了。

```c++
#include<iostream>
//class FireSpear麻烦
//{
//   std::string name;
//    std::string icon;
//};
//class IceSpear
//{
//   std::string name;
//   std::string icon;
//};
class Spear//父类
{
    protected://外界无法访问。但是子类继承到，可以访问
        std::string name;
        std::string icon;
    public:
    Spear(const std::string& name_,const std:: string& icon_):name(name_),icon(icon_)
    {
        std::cout<<"Spear()"<<std::endl;
    }
};
class FireSpear: public Spear
{
   public:
    FireSpear(const std::string& name_,const std:: string& icon_,int i_):Spear(name_,icon_),i(i)
    {
        std::cout<<"FireSpear()"<<std::endl;
    }//先初始化父类的部分，在初始化子类的部分
    private:
    int i;
};
class IceSpear: public Spear
{
    
};
```

**2.**C++继承原理：C++的继承可以理解为在创建子类成员变量之前先创建父类的成员变量，实际上，C语言就是这么模仿出继承功能的。

**3.**C++继承的注意事项。

(1) C++子类对象的构造过程。先调用父类的构造函数，再调用子类的构造函数，也就是说先初始化父类的成员，再初始化子类的成员。

(2) 若父类没有默认的构造函数，子类的构造函数又未调用父类的构造函数，则无法编译。

(3) C++子类对象的析构过程。先调用子类的析构函数，再调用父类的析构函数。

```c++
#include<iostream>
class Spear
{
public:
	Spear(const std::string& name_, const std::string& icon_) :name(name_), icon(icon_)
	{
		std::cout << "Spear()" << std::endl;
	}
	~Spear()
	{
		std::cout << "~Spear()" << std::endl;
	}
	 virtual void openFire()const
	{
		std::cout << "Spear::openFire" << std::endl;
	}
protected:
	std::string name;
	std::string icon;

};
class FireSpear :public Spear
{
public:
	FireSpear(const std::string& name_, const std::string& icon_, int i_) :Spear(name_, icon_), i(i)
	{
		std::cout << "FireSpear()" << std::endl;
	}
	~FireSpear()
	{
		std::cout << "~FireSpear()" << std::endl;
	}
private:
	int i;
};
class IceSpear: public Spear
{
public:
	void openFire()const
	{
		std::cout << "IceSpear::openFire" << std::endl;
	}
};
void openFire(const Spear* pSpear)
{
	pSpear->openFire();
	delete pSpear;
}
int main()
{
	FireSpear fireSpear("fanfan", "sad", 9);
	std::cout << "-------------------" << std::endl;
	openFire(new FireSpear("fanfan", "love", 10));
	std::cout << "-------------------" << std::endl;
	return 0;
}
```

```bash
Spear()
FireSpear()
-------------------
Spear()
FireSpear()
Spear::openFire
~Spear()
-------------------
~FireSpear()
~Spear()
```



总结：面向对象三大特性的继承就这么简单，很多人觉得类继承很复杂，其实完全不是这样的，只要明白子类在内存上其实就相当于把父类的成员变量放在子类的成员变量前面罢了。构造和析构过程也是为了这个机制而设计的。

## (**)虚函数及其实现原理，override 关键字

**1.虚函数介绍：**

(1) 虚函数就是面向对象的第三大特点：**多态**。多态非常的重要，它完美解决了上一课设计游戏装备类的问题，我们可以只设计一个函数，函数参数是基类指针，就可以调用子类的功能。比如射击游戏，所有的枪都继承自一个枪的基类，人类只要有一个开枪的函数就可以实现所有枪打出不同的子弹。

(2) 父类指针可以指向子类对象，这个是自然而然的，**因为子类对象的内存前面就是父类成员，类型完全匹配。**

(3) 当父类指针指向子类对象，且子类重写父类某一函数时。父类指针调用该函数，就会产生以下的可能

**①** **该函数为虚函数：父类指针调用的是子类的成员函数。**

**②** **该函数不是虚函数：父类指针调用的是父类的成员函数。**

```c++
#include<iostream>
class Spear
{
public:
	Spear(const std::string& name_, const std::string& icon_) :name(name_), icon(icon_)
	{
		std::cout << "Spear()" << std::endl;
	}
	~Spear()
	{
		std::cout << "~Spear()" << std::endl;
	}
	 virtual void openFire()const//
	{
		std::cout << "Spear::openFire" << std::endl;
	}
protected:
	std::string name;
	std::string icon;

};
class FireSpear :public Spear
{
public:
	FireSpear(const std::string& name_, const std::string& icon_, int i_) :Spear(name_, icon_), i(i)
	{
		std::cout << "FireSpear()" << std::endl;
	}
	virtual ~FireSpear()
	{
		std::cout << "~FireSpear()" << std::endl;
	}
	virtual void openFire()const override
	{
		std::cout << "FireSpear::openFire" << std::endl;
	}
private:
	int i;
};
class IceSpear: public Spear
{
public:
	virtual void openFire()const//加了virtual动态绑定
	{
		std::cout << "IceSpear::openFire" << std::endl;
	}
};
void openFire(const Spear* pSpear)
{
	pSpear->openFire();
	delete pSpear;
}
int main()
{
	FireSpear fireSpear("fanfan", "sad", 9);
	std::cout << "-------------------" << std::endl;
	openFire(new FireSpear("fanfan", "love", 10));
	std::cout << "-------------------" << std::endl;
	return 0;
}

```

```bash
Spear()
FireSpear()
-------------------
Spear()
FireSpear()
FireSpear::openFire
~Spear()
-------------------
~FireSpear()
~Spear()
```



**2.**虚函数的注意事项：

(1) 子父类的虚函数必须完全相同，为了防止开发人员一不小心将函数写错，于是C++11添加了override关键字。

**(2)** **父类的析构函数必须为虚函数：当父类对象指向子类对象时，容易使独属于子类的内存泄露。会造成内存泄露的严重问题。**

**3.**overide关键字的作用：前面已经说过了，为了防止开发人员将函数名写错了，加入了override关键字。

```c++
#include<iostream>
class Spear
{
public:
	Spear(const std::string& name_, const std::string& icon_) :name(name_), icon(icon_)
	{
		std::cout << "Spear()" << std::endl;
	}
	~Spear()
	{
		std::cout << "~Spear()" << std::endl;
	}
	 virtual void openFire()const//
	{
		std::cout << "Spear::openFire" << std::endl;
	}
protected:
	std::string name;
	std::string icon;

};
class FireSpear :public Spear
{
public:
	FireSpear(const std::string& name_, const std::string& icon_, int i_) :Spear(name_, icon_), i(i)
	{
		std::cout << "FireSpear()" << std::endl;
	}
	virtual ~FireSpear()
	{
		std::cout << "~FireSpear()" << std::endl;
	}
	virtual void openFire()const override
	{
		std::cout << "FireSpear::openFire" << std::endl;
	}
private:
	int i;
};
class IceSpear: public Spear
{
public:
	virtual void openFire()const//加了virtual动态绑定
	{
		std::cout << "IceSpear::openFire" << std::endl;
	}
};
void openFire(const Spear* pSpear)
{
	pSpear->openFire();
	delete pSpear;
}
int main()
{
	FireSpear fireSpear("fanfan", "sad", 9);
	std::cout << "-------------------" << std::endl;
	openFire(new FireSpear("fanfan", "love", 10));
	std::cout << "-------------------" << std::endl;
	Spear* pSpear = new FireSpear("fanfan", "happy", 11);
	std::cout << "-----------------" << std::endl;
	return 0;
}

```

```bash
Spear()
FireSpear()
-------------------
Spear()
FireSpear()
FireSpear::openFire
~Spear()
-------------------
Spear()
FireSpear()
-----------------
~FireSpear()
~Spear()

```



**4.** **虚函数实现多态的原理介绍**

(1) 动态绑定和静态绑定：

① 静态绑定：程序在编译时就已经确定了函数的地址，比如非虚函数就是静态绑定。

② 动态绑定：程序在编译时确定的是程序寻找函数地址的方法，只有在程序运行时才可以真正确定程序的地址，比如虚函数就是动态绑定。

(2) 虚函数是如何实现动态绑定的呢？

① 每个有虚函数的类都会有一个**虚函数表**，对象其实就是指向虚函数表的指针，编译时编译器只告诉了程序会在运行时查找虚函数表的对应函数。**每个类都会有自己的虚函数表，所以当父类指针引用的是子类虚函数表时，自然调用的就是子类的函数。**

总结：虚函数是C++类的重要特性之一，很简单，但使用频率非常高，至于如何实现的也要掌握。

## 静态成员变量与静态函数

**1.**静态成员变量：

(1) Part2的第六节课就讲过C语言的静态成员变量，在编译期就已经在静态变量区明确了地址，所以生命周期为程序从开始运行到结束，作用范围为与普通的成员变量相同。这些对于类的静态成员变量同样适用。

```c++
#include<iostream>
class Test
{
    public:
    static unsigned i;
}
unsigned Test::i=20;//必须在类外进行初始化
int main(){
    std::cout<<Test::i<<std::endl;//类名调用
    return 0;
}
```

(2) 类的静态成员变量因为创建在静态变量区，所以直接属于类，也就是我们可以直接通过类名来调用，当然通过对象调用也可以。

```c++
#include<iostream>
class Test
{
    public:
    static unsigned i;
}
unsigned Test::i=20;//必须在类外进行初始化
int main(){
    Test test;
    std::cout<<test.i<<std::endl;//类对象调用
    return 0;
}
```

**2.**静态成员变量的注意项：

(1) 静态成员变量必须在类外进行初始化，否则会报未定义的错误，不能用构造函数进行初始化。因为静态成员变量在静态变量区，只有一份，而且静态成员变量在编译期就要被创建，成员函数那都是运行期的事情了

**3.**静态成员函数的特点：静态成员函数就是为静态成员变量设计的，就是为了维持封装性。

 ```c++
 #include<iostream>
 class Test
 {
 public:
     static unsigned getI() { return i; }
 private:
     static unsigned i;
 };
 unsigned Test::i = 20;//必须在类外进行初始化
 int main() {
     Test test;
     std::cout << Test::getI() << std::endl;//类对象调用
     return 0;
 }
 ```

## 纯虚函数

**1.**纯虚函数介绍：

(1) 还是那个枪械射击的例子，基础的枪类有对应的对象吗？没有。它唯一的作用就是被子类继承。

(2) 基类的openfire函数实现过程有意义吗？没有。它就是用来被重写的。

(3) 所以纯虚函数的语法诞生了，只要将一个虚函数写为纯虚函数，那么该类将被认为无实际意义的类，无法产生对象。纯虚函数也不用去写实际部分。写了编译期也会自动忽略。

```c++
#include<iostream>
class Spear//父类
{
    
protected:
    std::string name;
    std::string icon;
public:
    FireSpear(const std::string& name_, const std::string& icon_,int i) Spear(name_,icon_), i(i)
    {
        std::cout << "Spear()" << std::endl;
    }
    virtual void openFire()const=0//变成纯虚函数，不需要实现，后边不用写，但是不能把虚构函数和虚构函数忽略
    {
        std::cout << "Spear::openFire" << std::endl;
    }
pirvate:
    int i;
};
class FireSpear : public Spear
{
public:
    void openFire()const
    {
        std::cout<<"FireSpear::openfire" << std::endl;
    }
};
 virtual void openFire(const Spear* pSpear)//加了virtual动态绑定
{
    pSpear->openFire();
    delete pSpear;
}
int main()
{
    openFire(new FireSpear("acd", "sad", 10));
    return 0;
}
```

 总结：纯虚函数的特点就是语法简单，却经常使用，必会。

## RTTL

**1.**RTTI介绍：

(1) RTTI（Run Time Type Identification）即通过运行时类型识别，程序能够通过基类的指针或引用来检查这些指针或引用所指向的对象的实际派生类。

(2) C++为了支持多态，C++的指针或引用的类型可能与它实际指向对象的类型不相同，这时就需要rtti去判断类的实际类型了，**rtti是C++判断指针或引用实际类型的唯一方式。**

 **2.**RTTI的使用场景：**可能有很多人会疑惑RTTI的作用，所以单独拿出来说一下。**

(1) 异常处理：这是RTTI最主要的使用场景，具体作用在异常处理章节会详细讲解。

(2) IO操作：具体作用等到IO章节会详细讲解。

**3.**RTTI的使用方式：RTTI的使用过程就两个函数

(1) typeid函数：typeid函数返回的一个叫做type_info的结构体，该结构体包括了所指向对象的实际信息，其中name()函数就可以返回函数的真实名称。type_info结构体其他函数没什么用.

 ```c++
 std::cout<<typeid(*指针).name()<<std::endl;
 ```

(2) dynamic_cast函数：C++提供的将父类指针转化为子类指针的函数。

```c++
FirSpear* pFirSpear=dunamic_cast<FireSpear*>(pSpear);//指针和引用可以，转化成功返回对应指针，不成功NONE
```

重要写法

```c++
if(std::string(typeid(*pSpear).name())=="class FirSpear")
{
    FirSpear* pFirSpear=dunamic_cast<FirSpear*>(pSpear);
}
```

**1.RTTI的注意事项：**

**当使用typeid函数时，父类和子类必须有虚函数（父类有了虚函数，子类自然会有虚函数），否则类型判断会出错。**

RTTI总结：就是C++在运行阶段判断对象实际类型的唯一方式。

## 多继承

多继承了解一下就可以了。

**1.**多继承的概念：就是一个类同时继承多个类，在内存上，该类对象前面依次为第一个继承的类，第二个继承的类，依次类推。

```c++
##include<iostream>
class Base1
{
public:
	Base1(int base1I_) :base1I(base1I_) { std::cout << "Base1()" << std::endl; }
protected:
	int base1I;
};
class Base2
{
public:
	Base2(int base2I_) :base2I(base2I_) { std::cout << "Base2()" << std::endl; }
protected:
	int base2I;
};
class Dervied : public Base1, public Base2
{
public:
	Dervied(int base1I_, int base2I_, int i_) :Base1(base1I_), Base2(base2I_), i(i_)
	{
		std::cout << "Derived" << std::endl;
	}
private:
	int i;
};
int main()
{
	Dervied dervied(10, 20, 30);
	
	return 0;
}
```

```c++
Base1()
Base2()
Derived//优先调用子类的参数，但是如果调用父类们相同的参数，就会出错
```

**2.**多继承的注意点：

(1) 多继承最需要注意的点就是重复继承的问题

(2) 多继承会使整个程序的设计更加复杂，平常不推荐使用。C++语言中用到多继承的地方主要就是借口模式。相较于C++，java直接取消了多继承的功能，添加了借口。

**3.**多继承的总结：多**继承这个语法虽然在某些情况下使代码写起来更加简洁，但会使程序更加复杂难懂，一般来说除了借口模式不推荐使用。**

## 虚继承及其实现原理

**1.**虚继承的概念：虚继承就是为了避免多重继承时产生的二义性问题。虚继承的问题用语言不好描述，但用代码非常简单。

```c++
#include<iostream>
class TrueBase
{
public:
	std::string icon;
	int i = 100;
};
class Base1 :  virtual public TrueBase//虚继承
{
public:
	Base1(int base1I_) :base1I(base1I_) { std::cout << "Base1()" << std::endl; }
	int base1I;
};
class Base2 : virtual public TrueBase//虚继承
{
public:
	Base2(int base2I_) :base2I(base2I_) { std::cout << "Base2()" << std::endl; }
	int base2I;
};
class Dervied :virtual public Base1, virtual public Base2//这里可加可不加，不太理解就加上
{
public:
	Dervied(int base1I_, int base2I_, int i_) :Base1(base1I_), Base2(base2I_)
	{
		std::cout << "Derived" << std::endl;
	}
};
int main()
{
	Dervied derived(10, 20, 30);
	std::cout << derived.i << std::endl;
	
	return 0;
}
```

```bash
Base1()
Base2()
Derived
100

```

**2.**虚继承的实现原理介绍：

(1) 使用了虚继承的类会有一个虚继承表，表中存放了父类所有成员变量相对于类的偏移地址。

(2) 按照刚才的代码，B1，B2类同时有一个虚继承表，当C类同时继承B1和B2类时，每继承一个就会用虚继承表进行比对，发现该变量在虚继承表中偏移地址相同，就只会继承一份。

**4.**虚继承的总结：这个语法就是典型的语法简单，但在游戏开发领域经常使用的语法，其它领域使用频率会低很多。

##  (**)移动构造函数与移动赋值运算符

**1.**对象移动的概念：

(1) 对一个体积比较大的类进行大量的拷贝操作是非常消耗性能的，因此C++11中加入了“对象移动”的操作

(2) 所谓的对象移动，其实就是把该对象占据的内存空间的访问权限转移给另一个对象。比如一块内存原本属于A，在进行“移动语义”后，这块内存就属于B了。

 **2.**移动语义为什么可以提高程序运行效率。因为我们的各种操作经常会进行大量的“复制构造”，“赋值运算”操作。这两个操作非常耗费时间。移动构造是直接转移权限，这是不是就快多了。

**注意：在进行转移操作后，被转移的对象就不能继续使用了，所以对象移动一般都是对临时对象进行操作（因为临时对象很快就要销毁了）。**

```c++
#include<iostream>
class Test
{
public:
	Test() = default;//默认构造函数
	Test(const Test& test)
	{
		if (test.str)
		{
			str = new char[strlen(test.str) + 1]();//加1不能省略
			strcpy_s(str, strlen(test.str) + 1, test.str);
		}
		else
		{
			str = nullptr;
		}
	}
	Test(Test&& test)//移动构造函数
	{
		if (test.str)
		{
			str = test.str;
			test.str = nullptr;
		}
		else
		{
			str = nullptr;
		}
	}
	Test& operator=(const Test& test)
	{
		if (this == &test)
		{
			return *this;
		}
		if (str) {//是否为空字符串
			delete[] str;
			str = nullptr;
		}
		if (test.str)
		{
			str = new char[strlen(test.str) + 1]();
			strcpy_s(str, strlen(test.str) + 1, test.str);
		}
		else
		{
			str = nullptr;
		}
		return *this;
	}
	Test& operator = (Test&& test)
	{
		if (this == &test)
		{
			return *this;
		}
		if (str) {
			delete[] str;
			str = nullptr;
		}
		if (test.str)
		{
			str = test.str;//转移权限
			test.str;
		}
		else
		{
			str = nullptr;
		}
		return *this;
	}
private:
	char* str = nullptr;//规范写法
};
Test makeTest()
{
	Test t;
	return t;
}
int main() {
	Test t = makeTest();
	return 0;
}

```

 注意这里的右值引用不能是const的，因为你用右值引用函数参数就算为了让其绑定到一个右值上去的！就是说这个右值引用是一定要变的，但是你一旦加了const就没法改变该右值引用了。

**3.** **默认移动构造函数和默认移动赋值运算符**

 会默认生成移动构造函数和移动赋值运算符的条件：

 **只有一个类没有定义任何自己版本的拷贝操作（拷贝构造，拷贝赋值运算符），且类的每个非静态成员都可以移动，系统才能为我们合成。**

 **可以移动的意思就是可以就行移动构造，移动赋值。所有的基础类型都是可以移动的，有移动语义的类也是可以移动的。**

hhh,周五成功把part3结束，但是移动构造函数还是不太懂，要复习喽

 



