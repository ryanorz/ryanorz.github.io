---
layout:     post
title:      "值得一用的C++11特性"
subtitle:   "那些增强代码可读性，性能，简化代码逻辑的新特性，你值得拥有"
date:       2016-09-27
author:     "Ryan"
header-img: "img/post-bg-2015.jpg"
tags:
    - Programming
    - C/C++
---

下面这些特性需要编译器支持哦，`gcc`的编译选项要加上`-std=c++0x`或`-std=c++11`.

# 1. {}初始化

与以前的语法相比，以前所有的`()`初始化都可换成`{}`初始化．
加入这个语法的目的是统一初始化方法．
示例代码如下：

```cpp
C c {0, 0}; // C++11 only, same as: C c(0, 0);

int *a = new int[3] {1, 2, 3}; // c++11 only

class X
{
public:
	X(): a{1, 2, 3, 4} {} // c++11, member array initializer
private:
	int a[4];
};

// c++11 container initializer
vector<string> vs = {"first", "second", "third"};
map singer = {
	{"Lady Gaga", "+1 (212) 555-7890"},
	{"Beyonce Knowles", "+1 (212) 555-0987"}
};

Class C
{
public:
	C();
private:
	int a = 7; // c++11 only
};
```
# 2. 自动类型推导`auto`与`decltype`

## 2.1 `auto`

```cpp
auto x = 0;   // x is int
auto c = 'c'; // char
auto d = 0.5; // double
auto l = 14400000000000LL; //long long
```

其最大好处就是让代码简洁，如:

```cpp
vector<int>::const_iterator ci = vi.begin();
auto ci = vi.begin();
```

但是`auto`的特性不能被滥用，否则会降低代码可读性．

```cpp
auto obj = ProcessData(vars);
```

上述代码完全让人不知道`obj`的类型，正确的使用姿势应该是这样的:

```cpp
auto pObject = new SomeType::VeryVeryLongType();
```

## 2.2 新的返回值语法

```cpp
int func(int x);
```

在`C++11`中，如果你愿意，你可以将返回值放在函数声明的最后，将`auto`放在返回值的位置:

```cpp
auto func(int x) -> int;
```

这么做法主要和`decltype`搭配起来使用，一般应用在泛型编程中．

## 2.3 `decltype`

`decltype`关键字用于查询表达式的类型。只是`查询`,不`求值`.
所有下面这段代码是不能编译通过的．

```cpp
cout << decltype(n);
```

正确的使用方式是这样：

```cpp
template<typename _Container>
auto begin(_Container& __cont) -> decltype(__cont.begin())
{
	return __cont.begin();
}
```

# 3. range `for` loop

为了在遍历容器时支持”foreach”用法，C++11扩展了for语句的语法。
用这个新的写法，可以遍历C类型的数组、初始化列表以及任何重载了非成员的`begin()`和`end()`函数的类型。

```cpp
std::map<std::string, std::vector<int>> map;
std::vector<int> v;
v.push_back(1);
v.push_back(2);
v.push_back(3);
map["one"] = v;

for(const auto& kvp : map) {
	std::cout << kvp.first << std::endl;

	for(auto v : kvp.second)
		std::cout << v << std::endl;
}

int arr[] = {1,2,3,4,5};
for(int& e : arr)
	e = e*e;
```

# 4. `nullptr`

`C++11`关键字`nullptr`是`std::nullptr_t`类型的值，用来指代空指针.
`C/C++`的`NULL`宏是个被有很多潜在BUG的宏。因为有的库把其定义成整数`0`，有的定义成 `(void*)0`。

```cpp
void f(int);       // #1
void f(char *);    // #2

//C++03
f(0);              // 二义性

//C++11
f(nullptr)         // 无二义性，调用f(char*)
```

所以`C++11`中使用`nullptr`初始化指针．

# 5. `override`和`final`标识符

在`C++`中并没有一个机制来确定一个虚函数是否会在派生类中被重写，着使得某些`C++`代码读起来很费劲，非常容易导致BUG.
因此，在某些公司的C++规范中甚至要求避免使用继承或是多重继承．
而在`C++11`中新加入了两个标识符使这类问题变得简单明了了许多：

* `override`表示函数应当重写基类中的虚函数
* `final`表示派生类不应该继续重写这个虚函数

```cpp
class A
{
public:
	void f0();
	virtual void f1() final;
	virtual void f2();
	virtual void f3();
};

class B: public A
{
public:
	void f0() override;                // error, A::f0 is not virtual
	virtual void f1() override;        // error, cannot override f1()
	virtual void f2() override final;  // OK
	virtual void f3() override;        // OK

};

class C: public B
{
public:
	virtual void f1() override;        // error, cannot override f1()
	virtual void f2() override;        // error, cannot override f2()
	virtual void f3() override final;  // OK
};
```

# 6. `Lvalues, Rvalues and References`

## 6.1 `Lvalues and Rvalues`

* An `lvalue` is an expression that identifies a non-temporary object.
* An `rvalue` is an expression that identifies a temporary object or is a value (such as a literal constant) not associated with any object.

简言之，`lvalue(左值)`就是有标记的变量，　`rvalue(右值)`无标记的临时变量．

```cpp
vector<string> arr( 3 );
const int x = 2;
int y;
...
int z = x + y;
string str = "foo";
vector<string> *ptr = &arr;
```

在上述例子中，`arr, str, arr[x], x, &x, y, z, ptr, *ptr, (*ptr)[x]` 均是左值．
`2, "foo", x+y, str.substr(0,1)`是右值．

左值引用，使用`&`　:

```cpp
string str = "hell";
string & rstr = str; // rstr is another name for str
rstr += ’o’;         // changes str to "hello"
bool cond = (&str == &rstr); // true; str and rstr are same object

string & bad1 = "hello";  // illegal: "hello" is not a modifiable lvalue
string & bad2 = str + ""; // illegal: str+"" is not an lvalue
string & sub = str.substr( 0, 4 ); // illegal: str.substr( 0, 4 ) is not an lvalue
```

右值引用，使用`&&` :

```cpp
string str = "hell";
string && bad1 = "hello";           // Legal
string && bad2 = str + "";          // Legal
string && sub = str.substr( 0, 4 ); // Legal
```

## 6.2 `lvalue references use`

### 6.2.1 `aliasing complicated names`

```cpp
auto & whichList = theLists[ myhash( x, theLists.size( ) ) ];
if( find(begin(whichList), end(whichList), x) != end(whichList) )
	return false;
whichList.push_back( x );
```

### 6.2.2 `range for loops`

```cpp
for( auto & x : arr )
	++x;
```

### 6.2.3 `avoiding a copy`

如果没必要再拷贝一遍返回值(尤其是返回值非常大的情况下，拷贝一遍是非常浪费时间的), 可以直接引用避免拷贝．

```cpp
LargeType randomItem1( const vector<LargeType> & arr )
{
	return arr[ randomInt( 0, arr.size( ) - 1 ) ];
}
const LargeType & randomItem2( const vector<LargeType> & arr )
{
	return arr[ randomInt( 0, arr.size( ) - 1 ) ];
}
vector<LargeType> vec;
...
LargeType item1 = randomItem1( vec );          // copy
LargeType item2 = randomItem2( vec );          // copy
const LargeType & item3 = randomItem2( vec );  // no copy
```

## 6.3 `move`语义

转移语义可以将资源 ( 堆，系统对象等 ) 从一个对象转移到另一个对象，这样能够减少不必要的临时对象的创建、拷贝以及销毁，能够大幅度提高 C++ 应用程序的性能。

在现有的 C++ 机制中，我们可以定义拷贝构造函数和赋值函数。要实现转移语义，需要定义转移构造函数，还可以定义转移赋值操作符。对于右值的拷贝和赋值会调用转移构造函数和转移赋值操作符。如果转移构造函数和转移拷贝操作符没有定义，那么就遵循现有的机制，拷贝构造函数和赋值操作符会被调用。

```cpp
vector<int> partialSum( const vector<int> & arr )
{
	vector<int> result( arr.size( ) );

	result[ 0 ] = arr[ 0 ];
	for( int i = 1; i < arr.size( ); ++i )
		result[ i ] = result[ i - 1 ] + arr[ i ];

	return result;
}

vector<int> vec;
...
vector<int> sums = partialSum( vec ); // Copy in old C++; move in C++11
```

## 6.4 `std::move`的使用

```cpp
void BadSwap( vector<string> & x, vector<string> & y )
{
	vector<string> tmp = x;  // copy the first time
	x = y;                   // copy the second time
	y = tmp;                 // OMG! copy the third time
}

void BetterSwap( vector<string> & x, vector<string> & y )
{
	vector<string> tmp = static_cast<vector<string> &&>( x );
	x = static_cast<vector<string> &&>( y );
	y = static_cast<vector<string> &&>( tmp );
}

// std::move 方法可以将左值（右值）转换成右值．
void BestSwap( vector<string> & x, vector<string> & y )
{
	vector<string> tmp = std::move( x );
	x = std::move( y );
	y = std::move( tmp );
}
```

## 6.5 `Perfect Forwarding`问题

右值引用是右值吗? C++11 中定义的 T&& 的推导规则为： 右值实参为右值引用，左值实参仍然为左值引用。

根据这个推导规则，右值引用能解决`Perfect Forwarding`问题．
`Perfect Forwarding`: 精确传递, 完美转发, 精准转发
精确传递适用于这样的场景：需要将一组参数原封不动的传递给另一个函数。
“原封不动”不仅仅是参数的值不变，在 C++ 中，除了参数值之外，还有一下两组属性：
左值／右值和 const/non-const。 精确传递就是在参数传递过程中，所有这些属性和参数值都不能改变。
