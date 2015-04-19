---
layout: post
title: 编程之美2015资格赛 题目一
date: 2015-04-19 20:17:37
category: "C++"
---

题目1 : 2月29日

时间限制:2000ms

单点时限:1000ms

内存限制:256MB

描述

给定两个日期，计算这两个日期之间有多少个2月29日（包括起始日期）。

只有闰年有2月29日，满足以下一个条件的年份为闰年：

1. 年份能被4整除但不能被100整除

2. 年份能被400整除

输入

第一行为一个整数T，表示数据组数。

之后每组数据包含两行。每一行格式为"month day, year"，表示一个日期。month为{"January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November" , "December"}中的一个字符串。day与year为两个数字。

数据保证给定的日期合法且第一个日期早于或等于第二个日期。

输出

对于每组数据输出一行，形如"Case #X: Y"。X为数据组数，从1开始，Y为答案。

数据范围

1 ≤ T ≤ 550

小数据：

2000 ≤ year ≤ 3000

大数据：

2000 ≤ year ≤ 2×109

样例输入

{% highlight c++ %}
4
January 12, 2012
March 19, 2012
August 12, 2899
August 12, 2901
August 12, 2000
August 12, 2005
February 29, 2004
February 29, 2012

{% endhighlight %}

样例输出
{% highlight c++ %}
Case #1: 1
Case #2: 0
Case #3: 1
Case #4: 3

{% endhighlight %}


我的解决方案：
{% highlight c++ %}

#include <iostream>
#include <string>
#include <vector>

using namespace std;


vector<string> monthes;

int monthnum(string str)
{
	for(int i=0;i<monthes.size();i++)
	{
		if(monthes[i] == str)
		{
			return i;
		}	
	}
	return -1;
}

int days(int y, int m, int d)
{
	int num;
	num = y/4 -y/100 + y/400;
	if(m < 2)
	{
		if(y%400==0 || (y%4==0 && y%100 !=0))
		{
			num --;
		}
	}
	return num;
}

int main()
{
	
	monthes.push_back("January");
	monthes.push_back("February");
	monthes.push_back("March");
	monthes.push_back("April");
	monthes.push_back("May");
	monthes.push_back("June");
	monthes.push_back("July");
	monthes.push_back("August");
	monthes.push_back("September");
	monthes.push_back("October");
	monthes.push_back("November");
	monthes.push_back("December");
	
	int T, c;
	string s_month;
	int month, year, day;
	int num1, num2;

	cin >> T;
	for(c=0;c<T;c++)
	{
		cin >> s_month >> day;
		getchar();
		cin >> year;

		month = monthnum(s_month);
		num1 = days(year, month, day);

		cin >> s_month >> day;
		getchar();
		cin >> year;

		month = monthnum(s_month);
		num2 = days(year, month, day);
		if(month ==1 && day == 29)
		{
			num2++;
		}
		cout << "Case #" << c+1 << ": " << num2-num1  << endl;
	}
}



{% endhighlight %}