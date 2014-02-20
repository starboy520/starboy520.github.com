---
layout: post
title:  "Rvalue reference"
date:   2014-02-10 21:41:24
categories: [c++11]
---

#Lvalue And Rvalue

In c++11 standard expressions are not just categoried by lvalue and rvalue, but also gvalue, xvalue. Here I do not want to goto details. 

To simplify, use c++ 03 standard, an expression is a lvalue or a rvalue through not exactly, wh can distigush them in practice with follows:

1. If you can take address of an expression (maybe using & operators), then it's a lvalue.
2. If the type of an expression is a `lvalue reference(T& or const T&) then it's a lvalue`.
3. otherwise, the expression is a rvalue. Conceptually, rvalues correspond to temporary objects. such as returned from fuctions or created through implicit type conversion, most literal values are rvalues.

<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
int a = 12;
int b = 12;
a = b; // a, b are lvalues.
int c = a*b;
// a * b = 12; error! rvalues on left hand side of assignment

int foo_bar();
int j = 0;
j = foo_bar(); // ok foo_bar() is a rvalue.
// int* p2 = &foo_ar() // error ! can not take the address of a rvalue.
]]></script>

#Rvalue reference
`Rvalue reference` can be distincted with old c++ 03 refenerce(now we called `Lvalue reference`).
It's a new feature in c++11 standard, It's purpose to solve move semantics and it's a basis of perfect forwarding(Details below).

Rvalues reference is such as T&& (But T&& does not really means Rvalue reference below details), and Lvalue reference is T&. 

Here is the binding rules:

1.non-const lvalue-reference can only bind to non-const lvalues.

2.const lvalue-reference can bind to const-lvalue, non-const-lvalue, const-rvalue, non-const-rvalue.

3.non-const-rvalue-reference can only bind to non-const-rvalue.

4.const-rvalue-reference can bind to const or non-const rvalue.

OK Now Let's go into rvalue reference.

## Rvalue reference is a rvalue ?
Let's see an example:
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
void foo(X&& x)
{
  X anotherX = x;
}
]]></script>

Here When X anotherX = x are called, which overload of X's copy constructor are called?
Since X are declared as an rvalue reference, it's quite plausible to expect x itself should also bind a rvalue,
in other words, one expect that anything declared as a rvalue reference itself an rvalue.
But here is the rule:

>Things that are declared as rvalue reference can be lvalue or rvalue, 
depends on if it has a name, then it's a rvalue, otherwise it's a rvalue.

So the above code actually calls `X(X const& rhs);`

Here it's a example that declared rvalue reference and does not have a name,
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
X&& goo();
X x = goo(); // Calls X(X&& rhs); because right side does not have a name.
]]></script>


## T&& is a rvalue reference?

Actually '&&' in a type declaration sometimes means rvalue reference, but sometime it means a rvalue reference or lvalue reference, which scott meyers calls it universal reference.
> ###If a variable or parameter is declared to have type T&& for some deduced type T, that variable or parameter is a universal reference.###

Here the requirement is that the universal reference are found involved type deduction. It's sayed that universal reference are only limited in `function templates and auto`, because auto declared variables are essentially the same as for templates)

Here's the universal reference rule of initialized :

1. If the expression initializing a universal reference is an lvalue, the universal reference becomes an lvalue reference.
2. If the expression initializing the universal reference is an rvalue, the universal reference becomes an rvalue reference.

now Consider 
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
Widget&& var1 = someOtherWidget();
auto&& var2 = var1;  // since var1 is a lvalue, var2's type is a lvalue reference.
// actually it's like Widget& var2 = var1;
]]></script>

Universial reference are most common in template function.
Consider:
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
template &lt;typename T&gt;
void f(T&& param); // && means rvalue reference.
]]></script>
Call
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
f(10) // 10 is a rvalue, so params becomes rvalue reference, in particular int&&
int i = 10;
f(i) //  params becomes lvalue reference, int& 
]]></script>

Remember that “&&” indicates a universal reference only where type deduction takes place
And should be T&&
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
template &lt;typename T&gt;
void f(std::vector&lt;T&gt;&& param);     // “&&” means rvalue reference
// also const qualifiers.
]]></script>

Here, we have both type deduction and a “&&”-declared function parameter, but the form of the parameter declaration is not “T&&”, it’s “std::vector&lt;t&gt;&&”.  As a result, the parameter is a normal rvalue reference, not a universal reference.  Universal references can only occur in the form “T&&”! Even the simple addition of a const qualifier is enough to disable the interpretation of “&&” as a universal reference

A final point is worth bearing in mind: the lvalueness or rvalueness of en expression is independent of its type.


## type deduction of an universal reference
Here is the reference-collapsing rule:
1. An rvalue reference to an rvalue reference becomes (“collapses into”) an rvalue reference.
2. All other references to references (i.e., all combinations involving an lvalue reference) collapse into an lvalue reference.



During type deduction for a template parameter that is a universal reference, lvalues and rvalues of the same type are deduced to have slightly different types.  In particular:

1. lvalues of type T are deduced to be of type T& (i.e., lvalue reference to T)
2. rvalues of type T are deduced to be simply of type T.

consider
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
template &lt;typename T&gt;
void f(T&& param);

int x;
... 
f(10);  // invoke f on rvalue.
f(x);   // invoke f on lvalue.
]]></script>

In the call to f with 10, T is ducution to be int, and instantiated f:
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
void f(int&& param);  // f instantiated from rvalue.
]]></script>

while x is lvalue, T id deduced to be int&, and f's instantiation :
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
void f(int& && params) -> void f(int& f); // 
]]></script>

This demonstrates the precise mechanism by which a universal reference can, after type deduction and reference collapsing, become an lvalue reference.

The truth is that a universal reference is really just an rvalue reference in a reference-collapsing context.



Things get subtler when deducing the type for a variable that is itself a reference. In that case, the reference part of the type is ignored.

For example:
<script type="syntaxhighlighter" class="brush: cpp"><![CDATA[
int x;
...
int&& r1 = 10;
int& r2 =x;

f(r1);
f(r2);
]]></script>
the deduced type for both r1 and r2 is int&. Why? First the reference parts of r1’s and r2’s types are stripped off (yielding int in both cases), then, because each is an lvalue, each is treated as int& during type deduction for the universal reference parameter in the call to f.




