---
layout: post
title:  "bypasses declarations with initialization"
date:   2014-02-10 21:41:24
categories: [c++, c++11]
---

今天在准备看下c++11里面static初始化线程相关文章时，翻c++11 standard标准第6.7发现了如下这段话

>It is possible to transfer into a block, but not in a way that **bypasses declarations with initialization**. A program that jumps from a point where a variable with automatic storage duration is not in scope to a point where it is in scope is ill-formed unless the variable has scalar type, class type with a trivial default constructor and a trivial destructor, a cv-qualiﬁed version of one of these types, or an array of one of the
preceding types and is declared without an initializer.

这段实在有点晦涩不太懂， 自己英语不太好，老一点的c++ 标准好懂一点, 其实是一个意思

>It is possible to transfer into a block, but not in a way that bypasses declarations with initialization. A program that jumps from a point where a local variable with automatic storage duration is not in scope to a point where it is in scope is ill-formed **unless** the** variable has POD type and is declared without an initializer**


意思就是程序从A跳转到B的时候，如果 B有申明局部变量(没有初始化， 而且只有默认的构造函数和析构函数)这样是允许的，除此外，不允许.

一个简单的例子：


<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
    switch (value)
    {
       // int a = 22227;  // 这边的初始化是不允许的  
       // 注意： 这里 c标准是允许的， 但是下面printf还是一个garbage值，
       // 具体什么原因，我需要再深究进去
	   int a; //声明变量是可以的

       case 1:
           printf("%d\n", a);
           break;
       default:
           printf("%d\n", a * a);
    }
]]></script>



# TO BE CONTINUE #
为何不允许初始化呢？ 今天就写这么多吧


# 附录：  #
关于POD, C++11标准是这么定义的：


> An aggregate is an array or a class (Clause 9) with no user-provided constructors (12.1), no brace-or-equal-initializers for non-static data members (9.2), no private or protected non-static data members (Clause 11), no base classes (Clause 10), and no virtual functions (10.3).


引用:

[http://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special/7189821#7189821](http://blog.eastmoney.com/lxb9721/blog_140643695.html "http://blog.eastmoney.com/lxb9721/blog_140643695.html")

[http://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special/4178176#4178176](http://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special/4178176#4178176 "http://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special/4178176#4178176")

[http://blog.eastmoney.com/lxb9721/blog_140643695.html](http://blog.eastmoney.com/lxb9721/blog_140643695.html "http://blog.eastmoney.com/lxb9721/blog_140643695.html")

c++ standard 6.7 