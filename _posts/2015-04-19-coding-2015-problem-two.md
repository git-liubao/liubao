---
layout: post
title: 编程之美2015资格赛 题目二
date: 2015-04-19 20:17:40
category: Language
tags:
- Language
- C++
- code
---

题目2 : 回文字符序列

时间限制:2000ms

单点时限:1000ms

内存限制:256MB

描述

给定字符串，求它的回文子序列个数。回文子序列反转字符顺序后仍然与原序列相同。例如字符串aba中，回文子序列为"a", "a", "aa", "b", "aba"，共5个。内容相同位置不同的子序列算不同的子序列。

输入

第一行一个整数T，表示数据组数。之后是T组数据，每组数据为一行字符串。

输出

对于每组数据输出一行，格式为"Case #X: Y"，X代表数据编号（从1开始），Y为答案。答案对100007取模。

数据范围

1 ≤ T ≤ 30

小数据

字符串长度 ≤ 25

大数据

字符串长度 ≤ 1000



样例输入
{% highlight c++ %}
5
aba
abcbaddabcba
12111112351121
ccccccc
fdadfa
{% endhighlight %}
样例输出
{% highlight c++ %}
Case #1: 5
Case #2: 277
Case #3: 1333
Case #4: 127
Case #5: 17
{% endhighlight %}

我的解决方案：
{% highlight c++ %}
#include <iostream>
#include <string>
#include <vector>

using namespace std;

int num[1005][1005]={0};


int main()
{
	int T, c, len;
	string s;
	cin >> T;
	getchar();
	for(c=1;c<=T;c++)
	{
		cin >> s;
		getchar();
		len = s.length();
		for(int i=0;i<len;i++)
		{
			for(int j=0;j<i;j++)
			{
				num[j][i]=0;
			}
			num[i][i]=1;
		}
		for(int i=0;i<len;i++)
		{
			for(int j=i-1;j>=0;j--)
			{
				num[j][i] = num[j+1][i] +num[j][i-1] - num[j+1][i-1];
				if(s[i] == s[j])
				{
					num[j][i] += num[j+1][i-1] +1;
				}
			}
		}
		std::cout << "Case #" << c << ": " << num[0][len-1]  << endl;
	}
}


{% endhighlight %}