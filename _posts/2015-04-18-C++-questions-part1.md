---
layout: post
title: C++面试题（一）
date: 2015-04-19 13:00:00
category: "C++"
---

C++面试题（一）

## 关于空类型的size

定义一个空的类型，没有任何的成员变量和函数，sizeof的结果是1，不是0。原因是声明类的实例时，必须在内存中占用一定的空间。

如果添加一个构造函数和析构函数，sizeof的结果仍然是1。

如果构造函数或者析构函数标记为虚函数，sizeof的结果为4（32为机器），原因是该类型的每个实例中都要有一个指向虚函数表的指针。

## 赋值构造函数不可以值传递

下面的程序，将会编译失败，愿意那是赋值构造函数内会调用赋值构造函数。就会永无休止的调用赋值构造函数，导致栈溢出。

{% highlight c++ %}
#include <iostream>
class A
{
private:
	int v;
public:
	A(int a) { v = a; }
	A(A b) { v=b.v; };
	//正确形势如下
	//A(const A& b) { v=b.v; };
	void print()
	{
		std::cout << v <<std::endl;
	}
};

int main()
{
	A a=10;
	A b=a;
	b.print();
}
{% endhighlight %}
