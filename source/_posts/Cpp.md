---
title: C++学习
date: 2023-08-08 14:06:27
tags:
- C++
categories: 
- C++
index_img: /imgs/Cpp.png
---

# C++学习

## 数组

> 方便存放同类型的元素

### 一维数组

一维数组数组名代表数组的首地址

一维数组名可以计算出数组在内存空间所占内存大小

### 二维数组

二维数组名代表二维数组的首地址，也可以查看某行的首地址

二维数组可以计算出数组在内存空间所占内存大小，也可以计算出某行所占内存大小

## 指针

> 作用：保存地址

指针的定义：数据类型	*指针变量名

32位操作系统指针所占内存空间：4个字节

64位操作系统指针所占内存空间：8个字节

### 指针的两大类型

空指针：

1. 指针变量指向内存编号为0的空间
2. **空指针指向内存无法访问**，0~255内存编号给系统占用的，无法访问
3. 可用于给指针变量初始化

野指针：指针变量指向非法空间，要尽量避免

### const与指针结合

const修饰指针称为常量指针:**指针指向的值不能变，指针的指向可以变**

```cpp
#include <iostream>
using namespace std;
int main(int argc, char** argv) {
//	常量指针：指针指向的值不能变，指针的指向可以变
	int a=20;
	const int *p=&a;
	int b=30;
	p=&b;//指针的指向可以改，由指向a修改为指向b 
//	*p=b; 指向的值不能改 
	cout<<*p<<endl;
	return 0;
}
```

const修饰常量称为指针常量：**指针指向的值可以改，指针的指向不可以修改**

```cpp
#include <iostream>
using namespace std;
int main(int argc, char** argv) {
//	指针常量：指针指向的值可以改，指针的指向不可以修改
	int a=20;
	int * const p=&a;
	int b=30;
	*p=b; //指向的值可以改
//	p=&b;//指针的指向不能改
	cout<<*p<<endl;
	return 0;
}
```

const既修饰常量又修饰指针：指针的指向不能改，指针指向的值也不能改

```cpp
#include <iostream>
using namespace std;
int main(int argc, char** argv) {
//	const既修饰常量又修饰指针：指针的指向不能改，指针指向的值也不能改
	int a=20;
	const int * const p=&a;
	int b=30;
//	*p=b; //指向的值不可以改
//	p=&b;//指针的指向不能改
	cout<<*p<<endl;
	return 0;
}
```

### 指针与数组

```cpp
#include <iostream>
using namespace std;
int main(int argc, char** argv) {
//	访问数组的第一种方式
	int array[10]= {1,2,3,4,5,6,7,8,9,10};
	int *p;
	for(int i=0; i<10; i++) {
		cout<<array[i]<<endl;
	}
	cout<<"利用指针访问数组元素:"<<endl; 
	p=array;
//	利用指针访问数组元素
	for(int i=0; i<10; i++) {
		cout<<*p<<endl;
		p++;
	}
	return 0;
}
```

### 指针与函数

值传递：不会改变实参的值

地址传递（引用传递）：会改变实参的值

```cpp
#include <iostream>
using namespace std;
void swap(int a,int b);
void swap2(int *a,int *b);
int main(int argc, char** argv) {
//	指针函数
//	值传递:不改变实参的值
	int a=10,b=20;
	cout<<"交换前:a:"<<a<<" b:"<<b<<endl;
	swap(a,b);
	cout<<"交换后:a:"<<a<<" b:"<<b<<endl;
//	引用传递：改变实参的值
	cout<<"交换前:a:"<<a<<" b:"<<b<<endl;
	swap2(&a,&b);
	cout<<"交换后:a:"<<a<<" b:"<<b<<endl;
	return 0;
}
void swap(int a,int b) {
	int temp;
	temp=a;
	a=b;
	b=temp;
}
void swap2(int *a,int *b) {
	int temp;
	temp=*a;
	*a=*b;
	*b=temp;
}
```

## 结构体

> 属于用户自定义的数据类型

结构体定义语法：struct 结构体名 {结构体成员列表}；

```cpp
#include <iostream>
#include <string>
using namespace std;
struct people {
	string name;
	int age;
	int sex;
} p3;
void display(people p);
int main(int argc, char** argv) {
//	1. struct 结构体名	变量名；
	struct people p1;
	p1.name="张三";
	p1.age=20;
	p1.sex=0;
	display(p1);
//	2. struct 结构体名	变量名={值1，值2}；
	struct people p2= {"李四",22,1};
	display(p2);
//	3. 定义结构体时顺便创建变量；
	p3.name="王五";
	p3.age=26;
	p3.sex=3;
	display(p3);
	return 0;
}
void display(people p) {
	cout<<"name:"<<p.name<<" age:"<<p.age<<" sex:"<<p.sex<<endl;
}
```

### 结构体数组

```cpp
#include <iostream>
#include <string>
using namespace std;
struct people {
	string name;
	int age;
};
int main(int argc, char** argv) {
//	结构体数组定义
	people ps[2]= {
		{"刘邦",200},
		{"周瑜",2000}
	};
//	结构体遍历 
	for(int i=0; i<2; i++) {
		cout<<"name:"<<ps[i].name<<" age:"<<ps[i].age<<endl;
	}
	return 0;
}
```

### 结构体指针

作用：通过指针访问结构体中的成员

```cpp
#include <iostream>
#include <string>
using namespace std;
struct people {
	string name;
	int age;
};
int main(int argc, char** argv) {
	people p1= {"张三",100};
	people *p=&p1;
//	利用结构体指针进行访问
	cout<<"name:"<<p->name<<" age:"<<p->age<<endl;
	return 0;
}
```

## 内存四区

### 代码区

用途：存放**函数体的二进制代码**，由操作系统进行管理

代码区是共享的，共享是针对需要频繁执行的程序，只需要在内存中有一份代码即可

代码区是只读的，只读为了防止程序意外修改了指令

### 栈区

用途：有编译器自动分配释放，存放**函数的参数值、局部变量**等

### 堆区

用途：由程序员分配和释放，若程序员不分配释放，程序结束时由操作系统回收

使用new关键字进行内存申请，使用delete关键字进行内存释放，数组需要使用delete[]进行释放

### 全局区

用途：存放**全局变量、静态变量以及常量**

包含常量区、字符串常量和其他常量

包括**全局变量、全局常量、静态变量和字符串常量**

```cpp
#include <iostream>
using namespace std;
//	全局变量
int g=10;
int h=10;
//	全局常量
const int i=10;
const int j=10;
int main(int argc, char** argv) {
//	局部变量
	int a=10;
	int b=10;
	cout<<"&a="<<(int *)&a<<" &b="<<(int *)&b<<endl;
//	局部常量
	const int c=20;
	const int d=20;
	cout<<"&c="<<(int *)&c<<" &d="<<(int *)&d<<endl;
//	静态变量
	static int e=10;
	static int f=10;
	cout<<"&e="<<(int *)&e<<" &f="<<(int *)&f<<endl;
	cout<<"&g="<<(int *)&g<<" &h="<<(int *)&h<<endl;
	cout<<"&i="<<(int *)&i<<" &j="<<(int *)&j<<endl;
//	字符串常量
	cout<<"&str="<<(int *)&"hello world"<<endl;
	return 0;
}
```

## C++特点

### 引用

> 给内存地址取别名，本质是指针常量

基本语法：数据类型 &别名=原名

注意事项：引用必须初始化，一旦初始化不可更改

引用做函数参数：可以简化指针修改实参

引用做函数返回值：可以作为函数左值，但是不要返回局部变量的引用

常引用：可以防止形参改变实参，即变为只读状态

### 函数进阶

函数默认参数：返回值类型	函数名（参数=默认值）{}，如果某个参数有默认值那么从这和默认值开始从左向右都得有默认值；函数声明有默认值，那么实现就不能有默认值。

函数的占位参数：调用时必须传递实参。

函数重载：同一个作用域，函数名称相同，函数的参数类型不同或者个数不同或者顺序不同；返回值不作为函数重载的条件。

## 面向对象

### 封装

ex:设计圆类，通过半径求周长

```cpp
#include <iostream>
#define PI 3.14
using namespace std;
class Circle {
	public :
		//半径
		float radius;
		float calc() {
			return 2*PI*radius;
		}
};
int main(int argc, char** argv) {
	Circle c;
	c.radius=10;
	cout<<"圆的周长为："<<c.calc()<<endl;
	return 0;
}
```

访问权限：

| 权限类型  | 类内 | 类外   |
| --------- | ---- | ------ |
| public    | 允许 | 允许   |
| protected | 允许 | 不允许 |
| private   | 允许 | 不允许 |

对于protected儿子可以访问父亲的保护内容

对于private儿子不可以访问父亲的保护内容

### 对象特性

> 空对象占用1bit的内存空间，编译器会给每个空对象也分配一个字节，是为了区分空对象的位置

#### 构造函数和析构函数

不提供构造函数和析构函数，编译器会默认提供空实现的构造函数和析构函数。

构造函数（进行初始化操作）：类名（）{}

构造函数的分类：

1. 有参构造和无参构造
2. 普通构造和拷贝构造

调用方法：

1. 括号法：调用无参构造函数时别带小括号，否则编译器会认为是函数的声明
2. 显示法：不能用拷贝构造函数初始化匿名对象，编译器会默认忽视小括号，从而造成重定义
3. 隐式法

调用规则：

1. 如果用户自定义有参构造函数，C++就不再提供默认无参构造函数，但是会提供默认拷贝构造函数
2. 如果用户自定义拷贝构造函数，C++不在提供其他构造函数

析构函数（进行销毁操作）：~类名（）{}

```cpp
#include <iostream>
#include <string>
using namespace std;
class Man {
	public:
		Man() {
			cout<<"无参构造函数调用"<<endl;
		}
		Man(string name) {
			this->name=name;
			cout<<"有参构造函数调用"<<endl;
		}
		Man(const Man &m) {
			this->name=m.name;
			this->age=m.age;
			cout<<"拷贝构造函数调用"<<endl;
		}
		~Man() {
			cout<<"析构函数调用"<<endl;
		}
	private:
		string name;
		int age;
};
int main(int argc, char** argv) {
	/**
		调用方式:
		括号法
		显示法
		隐式法
	*/
//	方式一:括号法
	Man m;
	Man m1("张三");
	Man m2(m1);
	/*注意：调用无参构造函数时别带小括号，这一点可Java有所不同，
			同时使用this->而不是this.*/
//	方式二：显示法
	Man m3=Man();
	Man m4=Man("李四");
	Man m5=Man(m4);
//	匿名对象
	Man("赵四");
	/*不能用拷贝构造函数初始化匿名对象
	Man(m5)==Man m5,编译器会忽略小括号
	*/
//	方式三：隐式法
	Man m6=m5;
	return 0;
}
```

拷贝构造函数的作用：

1. 使用一个已经创建完毕的对象来初始化新对象
2. 值传递的方式传参
3. 值方式返回局部对象

```cpp
#include <iostream>
#include <string>
using namespace std;
class People {
	public:
		People() {
			cout<<"默认构造函数调用"<<endl;
		}
		People(string name,int age) {
			this->name=name;
			this->age=age;
			cout<<"有参构造函数调用"<<endl;
		}
		People(const People &p) {
			this->name=p.name;
			this->age=p.age;
			cout<<"拷贝构造函数调用"<<endl;
		}
		string name;
		int age;
};
void test(People p) {

}
People test1() {
	People p("王五",10);
	return p;
}
int main(int argc, char** argv) {
//	方式一：用一个已经创建好的对象来初始化新对象
//	People p("张三",18);
//	People p1(p);
//	方式二：用于函数传参
//	test(p);
//	方式三：用于返回局部对象
	People p1=test1();
	cout<<"name:"<<p1.name<<" age:"<<p1.age<<endl;
	return 0;
}
```

#### 深拷贝与浅拷贝

浅拷贝：简单的赋值操作，会导致内存重复释放

深拷贝：在堆区重新申请空间，进行拷贝操作

```cpp
#include <iostream>
#include <string>
using namespace std;
class People {
	public:
		People() {
			cout<<"无参构造函数"<<endl;
		}
		People(string name,int age,int height) {
			this->name=name;
			this->age=age;
			this->height=new int(height);
			cout<<"有参构造函数"<<endl;
		}
		People(const People &p) {
			this->name=p.name;
			this->age=p.age;
//			this->height=p.height;//为了解决浅拷贝所带来的内存重复释放的问题，在拷贝时主动向堆区申请一块内存空间
			this->height=new int(*p.height);//此语句解决浅拷贝所带的问题 
			cout<<"拷贝构造函数"<<endl;
		}
		~People() {
			if(height!=NULL) {
				delete height;
				height=NULL;
			}
			cout<<"析构函数调用"<<endl;
		}
		string name;//姓名
		int age;//年龄
		int *height;//身高
};
int main(int argc, char** argv) {
	People p("张胜男",18,170);
	cout<<"name:"<<p.name<<" age:"<<p.age<<" height:"<<*p.height<<endl;
	People p1(p);
	cout<<"name:"<<p1.name<<" age:"<<p1.age<<" height:"<<*p1.height<<endl;
	return 0;
}
```

#### 初始化列表

```cpp
#include <iostream>
using namespace std; 
class People{
	public:
		People(int age,int sex):age(age),sex(sex){
		}
		int age;
		int sex;
};
int main(int argc, char** argv) {
	People p(18,0);
	return 0;
}
```

#### 静态成员

- 静态成员变量

  - 所有对象共享一份数据
  - 在编译阶段分配内存
  - 类内声明，类外初始化

- 静态成员函数：可以通过对象或类名访问

  - 所有对象共享同一个函数
  - 静态成员函数只能访问静态成员变量，不能访问非静态成员变量

  ```cpp
  #include <iostream>
  using namespace std;
  class People {
  	public:
  		People() {
  		}
  		People(int age) {
  			this->age=age;
  		}
  		int age;
  		static int sex;
  		static void func() {
  			cout<<"性别是:"<<sex<<endl;
  		}
  };
  //  类外初始化
  int People::sex=1;
  int main(int argc, char** argv) {
  //	方式一：对象名
  	People p(10);
  	p.func();
  //	方式二：类名
  	People::func(); 
  	return 0;
  }
  ```


#### this指针与空指针

this的用途：解决命名冲突和制造链式调用

空指针也能访问成员函数，需要加以判断保证代码的健壮性

#### 常函数和常对象

1. 常函数内不可以修改成员属性
2. 成员属性加mutable后就可以修改属性

常对象只能调用常函数

```cpp
#include <iostream>
using namespace std;
class People {
	public:
		People(int age,int sex) {
			this->age=age;
			this-sex=sex;
		}
//		常函数 ，加const后实际上修饰的是this，则this指向的内容也不可以改 
		void print() const {
			cout<<"age:"<<age<<" sex:"<<sex<<endl;
		}
		int age;
		int sex;
};
int main(int argc, char** argv) {
	return 0;
}
```

### 友元

#### 全局函数做友元

```cpp
#include <iostream>
#include <string>
using namespace std;
class People {
//	告诉编译器goodBrother是People的友元函数，可以访问private变量
		friend void goodBrother(People &p);
	public:
		People() {
			this->sitroom="客厅";
			this->bedroom="卧室";
		}
	public:
		string sitroom;
	private:
		string bedroom;
};
void goodBrother(People &p) {
	cout<<"好兄弟正在进入你的"<<p.sitroom<<endl;
	cout<<"好兄弟正在进入你的"<<p.bedroom<<endl;
}
int main(int argc, char** argv) {
	People p;
	goodBrother(p);
	return 0;
}
```

#### 友元类

```cpp
#include <iostream>
#include <string>
using namespace std;
class Brother {
//		声明People是Brother的友元类
		friend class People;
	public:
		Brother() {
			wc="厕所";
			bathroom="浴室";
		}
		string wc;
	private:
		string bathroom;
};
class People {
	public:
		People() {
			sittingroom="客厅";
			bedroom="卧室";
		}
		string sittingroom;
		void visit(Brother &b) {
			cout<<"正在访问好兄弟的"<<b.bathroom<<endl;
		}
	private:
		string bedroom;
};
int main(int argc, char** argv) {
	Brother b;
	People p;
	p.visit(b);
	return 0;
}
```

#### 成员函数做友元

```cpp
#include<iostream>
using namespace std;
#include<string>
//成员函数做友元
class Building;
class GoodGay
{
public:
    GoodGay();
    ~GoodGay();
    void visit();// 让 visit 访问Building中的私有函数
    void visit2();// 让 visit2 不可以访问Building中的私有函数
    Building *building;//指向对象的指针，在构造函数中要初始化指针
};
class Building
{
    //GoodGay类下的visit成员函数作为本类的好朋友，可以访问私有成员变量
    friend void GoodGay::visit();
public:
    string m_SittingRoom;
    Building();
private:
    string m_BedRoom;
};
Building::Building()
{
    m_SittingRoom = "客厅";
    m_BedRoom = "卧室";
}
GoodGay::GoodGay()
{
    //创建一个建筑物的对象
    //用new再堆区中申请一片空间返回的是一个指针，用building来接受这个指针，完成building初始化
    building = new Building; 
}
GoodGay::~GoodGay()
{
    delete building;
}
void GoodGay::visit()
{
    cout << "visit函数: Public " << building->m_SittingRoom << endl;
    cout << "visit函数: Private " << building->m_BedRoom << endl;
}
void GoodGay::visit2()
{
    cout << "visit2函数: Public " << building->m_SittingRoom << endl;
    //visit2函数不是Building的友元函数，不能访问其私有变量m_BedRoom
}
void test01()
{
    GoodGay gg;
    gg.visit();
    gg.visit2();
}
int main()
{
    test01();
    return 0;
}
```

### 运算符重载

#### 加号运算符重载

成员函数重载+运算符和全局函数重载+运算符

```cpp
#include <iostream>
#include <string>
using namespace std;
//	加号运算符重载
class Number {
	public:
		Number() {
		}
		Number(int n) {
			this->num=n;
		}
//		成员函数重载+
//		Number operator+(Number &n) {
//			Number temp;
//			temp.num=this->num+n.num;
//			return temp;
//		}
		int num;
};
//	全局函数重载+
Number operator+(Number &n1,Number &n2) {
	Number n3;
	n3.num= n1.num+n2.num;
	return n3;
}
int main(int argc, char** argv) {
	Number n(10);
	Number n1(15);
	Number n2=n+n1;
	cout<<n2.num<<endl;
	return 0;
}
```

#### 左移运算符重载

只能利用全局函数重载左移运算符

```cpp
#include <iostream>
#include <string>
using namespace std;
//左移运算符重载<<
class Number {
//	如果数据私有可以使用友元
	public:
		Number(int n) {
			this->num=n;
		}
		int num;

};
//	全局函数重载<<运算符
ostream& operator<<(ostream &cout,Number &n) {
	cout<<n.num;
	return cout;
}
int main(int argc, char** argv) {
	Number n1(10);
	cout<<n1<<endl;
	return 0;
}
```

#### 递增运算符重载

前置++运算符重载

```c++
#include <iostream>
#include <string>
using namespace std;
class Number {
	public :
		Number() {

		}
		Number(int num) {
			this->num=num;
		}
		int num;
//		前置++重载
		Number& operator++() {
			++num;
			return *this;
		}
};
ostream& operator<<(ostream &cout,Number n) {
	cout<<n.num;
	return cout;
}
int main(int argc, char** argv) {
	Number n(10);
	cout<<++(++n)<<endl;
	cout<<n<<endl;
	return 0;
}
```

后置++运算符重载

```cpp
#include <iostream>
#include <string>
using namespace std;
class Number {
		friend ostream& operator<<(ostream &cout,Number n);
	public :
		Number() {

		}
		Number(int num) {
			this->num=num;
		}
//		前置++重载
		Number& operator++() {
			++num;
			return *this;
		}
//		后置 ++重载
		Number operator++(int) {
			Number temp=*this;
			num++;
			return temp;
		}
		int num;
};
ostream& operator<<(ostream &cout,Number n) {
	cout<<n.num;
	return cout;
}
int main(int argc, char** argv) {
	Number n(10);
	cout<<n++;
	return 0;
}
```

#### 赋值运算符重载

```cpp
#include <iostream>
#include <string>
using namespace std;
class People {
	public:
		People(int age) {
			p_age=new int(age);
		}
		~People() {
			if(p_age!=NULL) {
				delete p_age;
				p_age=NULL;
			}
		}
//		=运算符重载
		People& operator=(People &p) {
//			如果自身有内存先释放干净
			if(p_age!=NULL) {
				delete p_age;
				p_age=NULL;
			}
			this->p_age= new int(*p.p_age);
			return *this;
		}
		int *p_age;
};
int main(int argc, char** argv) {
	People p1(10);
	People p2(18);
	p2=p1;
	cout<<"age:"<<*p2.p_age<<endl;
	return 0;
}
```

#### 关系运算符重载

```cpp
#include <iostream>
#include <string>
using namespace std;
class Person {
	public:
		Person() {

		}
		Person(int age) {
			this->age=age;
		}
//	关系运算符重载
		bool operator==(Person &p) {
			if(p.age==this->age) {
				return true;
			} else {
				return false;
			}
		}
		int age;
};
ostream& operator<<(ostream &cout,Person &p) {
	cout<<p.age;
	return cout;
}
int main(int argc, char** argv) {
	Person p(10);
	Person p1(10);
	if(p==p1) {
		cout<<"相等"<<endl;
	}
	return 0;
}
```

#### 函数调用运算符重载

又称为仿函数，写法很灵活

```cpp
#include <iostream>
#include <string>
using namespace std;
class Print {
	public:
		Print() {
		}
		void operator()(string valueString) {
			cout<<valueString<<endl;
		}
};
int main(int argc, char** argv) {
	Print print;
	print("hello world!");
	return 0;
}
```

### 继承

用处：去重，减少冗余

> 语法:class 派生类：继承方式 	父类

三种继承方式：（父类中的私有属性，子类无论哪种继承都访问不到）

1. 公共继承：父类的public和protected属性保持不变，私有属性访问不到
2. 保护继承：父类的public和protected属性变为子类的protected属性，私有属性访问不到
3. 私有继承：父类的public和protected属性变为子类的private属性，私有属性访问不到

**父类中所有的非静态成员属性都会被子类继承下去，只是私有属性被编译器隐藏了，所以访问不到**

查看对象模型：

```shell
cl /dl reportSingleClassLayout类名 文件名
```

#### 继承中的构造和析构的顺序

析构顺序与构造顺序相反

父类构造->子类构造->子类析构->父类析构

#### 子类继承访问同名成员的处理

1. 访问子类同名成员，直接访问即可
2. 访问父类同名成员，加作用域即可
3. 子类出现和父类同名的成员，子类会隐藏父类所有的同名成员函数

#### 子类继承访问静态成员的处理

两种访问方式：对象名和类名

1. 访问子类同名成员，直接访问即可
2. 访问父类同名成员，加作用域即可
3. 子类出现和父类同名的静态成员，子类会隐藏父类所有的同名静态成员函数

#### 多继承（不建议使用）

> 语法：class	子类：继承方式	父类1，继承方式	父类2

#### 菱形继承

概念：两个类继承一个基类，又有一个类继承这两个派生类

```cpp
#include <iostream>
#include <string>
using namespace std;
//动物类
//通过使用virtual关键字解决菱形继承问题
class Animal {
	public:
		int age;
};
//羊类
class Sheep:virtual public Animal {
};
//驼类
class Tuo:virtual public Animal {
};
//羊驼类
class SheepTuo:public Sheep,public Tuo {
};
int main(int argc, char** argv) {
	SheepTuo st;
	st.Tuo::age=28;
	st.Sheep::age=18;
	cout<<"st.Sheep::age:"<<st.Sheep::age<<endl;
	cout<<"st.Tuo::age:"<<st.Tuo::age<<endl;
	cout<<"st.age:"<<st.age<<endl;
	return 0;
}
```

### 多态

1. 静态多态：函数重载和运算符重载，编译阶段确定函数地址，地址早绑定
2. 动态多态：派生类和虚函数，运行阶段确定函数地址，地址晚绑定

#### 虚函数

```cpp
#include <iostream>
#include <string>
using namespace std;
class Animal {
	public:
//		此刻地址早绑定
//		void speak() {
//			cout<<"动物在说话"<<endl;
//		}
//		加virtual变成虚函数地址晚绑定
		virtual	void speak() {
			cout<<"动物在说话"<<endl;
		}
};
class Cat:public Animal {
	public :
		void speak() {
			cout<<"喵喵~~"<<endl;
		}
};
class Dog:public Animal  {
	public :
		void speak() {
			cout<<"汪汪~~"<<endl;
		}
};
int main(int argc, char** argv) {
	Cat cat;
	Dog dog;
//	Animal &animal=cat;
	Animal &animal=dog;
	animal.speak();
	return 0;
}
```

#### 纯虚函数和抽象类

有纯虚函数的类称为抽象类

语法：virtual 返回值类型	函数名（参数列表）=0；

抽象类特点：

1. 无法实例化
2. 子类必须重写抽象类的纯虚函数，否则子类也属于抽象类

```cpp
#include <iostream>
#include <string>
using namespace std;
class Animal {
	public:
//		变成纯虚函数 
		virtual	void speak()=0;
};
class Cat:public Animal {
	public :
		void speak() {
			cout<<"喵喵~~"<<endl;
		}
};
class Dog:public Animal  {
	public :
		void speak() {
			cout<<"汪汪~~"<<endl;
		}
};
int main(int argc, char** argv) {
	Cat cat;
	Dog dog;
//	Animal &animal=cat;
	Animal &animal=dog;
	animal.speak();
	return 0;
}
```

#### 虚析构和纯虚析构

有纯虚析构的类也称为抽象类

> 用处：如果子类属性开辟到堆区，那么父类指针在释放时无法调用子类的析构代码

虚析构语法：virtual ~类名（）{}

纯虚析构：virtual ~类名（）=0；必须要有具体实现

```cpp
#include <iostream>
#include <string>
using namespace std;
class Animal {
	public:
		Animal() {
			cout<<"Animal构造函数调用"<<endl;
		}
//		利用虚析构
//		virtual ~Animal() {
//			cout<<"Animal析构函数调用"<<endl;
//		}
//		利用纯虚析构
		virtual ~Animal()=0;
};
class Cat:public Animal {
	public :
		Cat() {
			cout<<"Cat构造函数调用"<<endl;
		}
		Cat(string name) {
			this->name=new string(name);
			cout<<"Cat有参构造函数调用"<<endl;
		}
		~Cat() {
			if(name!=NULL) {
				delete name;
				name=NULL;
			}
			cout<<"Cat析构函数调用"<<endl;
		}
		string *name;
};
Animal::~Animal() {
	cout<<"Animal析构函数调用"<<endl;
}
int main(int argc, char** argv) {
	Animal *animal=new Cat("小白");
	delete animal;
	return 0;
}
```

## C++泛型编程

### 模板

> 用途：提高复用性

#### 函数模板

语法：template<typename T>，typename可以用class代替

```cpp
#include <iostream>
#include <string>
using namespace std;
//利用模板
template <typename T>
void Swap(T &a,T &b) {
	T temp;
	temp=a;
	a=b;
	b=temp;
}
int main(int argc, char** argv) {
	int a=10,b=20;
	float c=10.9,d=10.6;
	Swap(a,b);
	cout<<a<<" ,"<<b<<endl;
	Swap(c,d);
	cout<<c<<" ,"<<d<<endl;
	return 0;
}
```

##### 普通函数与模板函数的区别

- 普通函数可以自动类型转换
- 函数模板如果使用自动类型，不会发生自动类型转换
- 函数模板如果使用指定类型，会发生自动类型转换

##### 普通函数与模板函数的调用规则

1. 函数模板和普通函数都可以，优先调用普通函数
2. 可以通过空模板参数列表来强制调用函数模板
3. 函数模板也可以重载
4. 如果函数模板能更好匹配，优先调用函数模板

##### 模板的局限性

对于数组和对象来说，可以使用template<>重写一个模板具体实现

#### 类模板

```cpp
#include <iostream>
#include <string>
using namespace std;
template <class NameType,class AgeType>
class People {
	public:
		People(NameType name,AgeType age) {
			this->name=name;
			this->age=age;
		}
		NameType name;
		AgeType age;
};
int main(int argc, char** argv) {
	People<string,int> p("张三",18);
	cout<<p.name<<p.age<<endl;
	return 0;
}
```

##### 类模板的成员函数的创建时机

类模板中成员函数只在调用时才去创建

##### 类模板对象做函数参数

传入方式

1. 指定传入方式
2. 参数模板化
3. 整个类模板化

##### 类模板继承

1. 当子类继承父类是一个类模板时，子类在声明时，要指出父类中的数据类型
2. 不指定，编译器无法给子类分配内存
3. 如果要灵活支出父类中的数据类型，子类也要变成类模板

### STL

> 查阅API即可，深入理解阅读《STL源码剖析》

#### 容器

##### String

> string本质上是一个类，内部封装了char *,是一个char *的容器

```cpp
#include <iostream>
#include <string>
using namespace std;
int main(int argc, char** argv) {
	string str="hello";
	const char *ch="china";
	string  str1(ch);
	string str2(10,'a');
	str+="world";
	cout<<str<<" "<<str1<<" "<<str2<<endl;
	return 0;
}
```

##### Vector

> 功能：类似数组，又称为单端数组，可以动态扩展（找更大的内存空间，将数据拷贝，释放原空间）

###### vector存放内置数据类型

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;
void print(int value) {
	cout<<value<<endl;
}
int main(int argc, char** argv) {
//	vector容器，类似于数组
	vector<int> v;
	v.push_back(10);
	v.push_back(20);
	v.push_back(30);
	v.push_back(40);
	v.push_back(50);
	vector<int>::iterator itBegin=v.begin();//起始迭代器，指向容器中第一个元素
	vector<int>::iterator itEnd=v.end();//结束迭代器，指向容器中最后一个元素的下一个位置
	cout<<"while循环遍历"<<endl;
	while(itBegin!=itEnd) {
		cout<<*itBegin<<endl;
		itBegin++;
	}
	cout<<"for循环遍历"<<endl;
	for(vector<int>::iterator it=v.begin(); it!=v.end(); it++) {
		cout<<*it<<endl;
	}
	cout<<"stl遍历算法"<<endl;
	for_each(v.begin(),v.end(),print);
	return 0;
}
```

###### vector存放自定义数据类型

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;
class People {
	public:
		People(string name,int age) {
			this->age=age;
			this->name=name;
		}
		string name;
		int age;
};
int main(int argc, char** argv) {
	People p1("李白",2000);
	People p2("达摩",1000);
	People p3("王昭君",900);
	People p4("公孙离",20);
	People p5("赵云",200);
	vector<People> v;
	v.push_back(p1);
	v.push_back(p2);
	v.push_back(p3);
	v.push_back(p4);
	v.push_back(p5);
	vector<People*> v1;
	v1.push_back(&p1);
	v1.push_back(&p2);
	v1.push_back(&p3);
	v1.push_back(&p4);
	v1.push_back(&p5);
	for(vector<People>::iterator it=v.begin(); it!=v.end(); it++) {
		cout<<"姓名:"<<it->name<<" 年龄："<<it->age<<endl;
	}
	cout<<"遍历自定义数据类型指针"<<endl;
	for(vector<People*>::iterator it=v1.begin(); it!=v1.end(); it++) {
		cout<<"姓名:"<<(*it)->name<<" 年龄："<<(*it)->age<<endl;
	}
	return 0;
}
```

###### vector容器嵌套

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;
//vector容器嵌套
int main(int argc, char** argv) {
	vector<vector<int> > v;
	vector<int> v1;
	vector<int> v2;
	vector<int> v3;
	vector<int> v4;
	for(int i=0; i<4; i++) {
		v1.push_back(i);
		v2.push_back(i+1);
		v3.push_back(i+2);
		v4.push_back(i+3);
	}
	v.push_back(v1);
	v.push_back(v2);
	v.push_back(v3);
	v.push_back(v4);
	for(vector<vector<int> >::iterator it=v.begin(); it!=v.end(); it++) {
		for(vector<int>::iterator vit=(*it).begin(); vit!=(*it).end(); vit++) {
			cout<<*vit<<" ";
		}
		cout<<endl;
	}
	return 0;
}
```

###### vector容器插入和删除

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;
void print(vector<int> &v) {
	for(vector<int>::iterator it=v.begin(); it!=v.end(); it++) {
		cout<<*it<<endl;
	}
}
int main(int argc, char** argv) {
	vector<int> v;
	v.push_back(10);
	v.push_back(20);
	v.push_back(30);
	v.push_back(40);
	v.push_back(50);
	print(v);
	v.pop_back();
	print(v);
	v.insert(v.begin(),60);
	print(v);
	v.erase(v.begin(),v.end());
	print(v);
	v.clear();
	print(v);
	return 0;
}
```

##### Deque

双端数组，可以对头端进行插入和删除

##### Stack（栈）：没啥可说的

##### Queue（队列）：没啥可说的

##### List

注意：list不支持随机存取，所以没有[]和at的访问方式

##### Set和Multiser

set结构会自动排序，底层是二叉树

- set中不允许有重复的值
- multiset允许有重复的值

##### Map

map中所以元素都是pair,底层为二叉树，可以根据key快速找到value

- map中不允许有重复key值元素
- multimap中允许有重复key值元素
