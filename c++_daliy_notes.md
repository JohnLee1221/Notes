### && 是右值引用

左值的英文简写为“lvalue”，右值的英文简写为“rvalue”。很多人认为它们分别是"left value"、"right value" 的缩写，其实不然。lvalue 是“loactor value”的缩写，可意为存储在内存中、有明确存储地址（可寻址）的数据，而 rvalue 译为 "read value"，指的是那些可以提供数据值的数据（不一定可以寻址，例如存储于寄存器中的数据）

***************************



### enum class

限定作用域的枚举类型(scoped enumeration)。定义限定作用域的枚举类型的一般形式是：首先是关键字[enum](https://so.csdn.net/so/search?q=enum&spm=1001.2101.3001.7020) class(或者等价地使用enum struct)，随后是枚举类型名字以及用花括号括起来的以逗号分隔的枚举成员(enumerator)列表，最后是一个分号。

枚举作用域(enumeration scope)是指枚举类型的成员的名字的作用域，起自其声明之处，终止枚举定义结束之处。C语言规定，枚举类型的成员(enumerator)的可见范围被提升至该枚举类型所在的作用域内。这被认为有可能污染了外部的作用域，为此，C++11引入了枚举类(enum class)解决此问题。

*******************



### static成员

概念
声明为static的类成员称为类的静态成员，分为如下两类：

- 用static修饰的成员变量，称之为静态成员变量
- 用static修饰的成员函数，称之为静态成员函数

静态的成员变量一定要在类外进行初始化

特性

1. **静态成员为所有类对象所共享，不属于某个具体的实例**

2. **静态成员变量必须在类外定义，定义时不添加static关键字**
3. **静态成员函数没有隐藏的this指针，不能访问任何非静态成员**

```c++
class A
{
public:
	static void Func()
	{
		cout << ret << endl;  // err错误，访问了非静态成员，因为无this指针
		cout << _k << endl; //正确
	}
private:
	//声明
	int ret = 0;
	static int _k;
};
//定义
int A::_k = 0;
```

4. **访问静态成员变量的方式，当静态成员变量为公有时，可有如下三种进行访问：**

* 对象.静态成员来访问
* 类名::静态成员来行访问
* 匿名对象突破类域进行访问

```c++
class A
{
public:
	static int _k;
};
int A::_k = 0;
int main()
{
	A a;
	cout << a._k << endl;  //通过对象.静态成员来访问
	cout << A::_k << endl; //通过类名::静态成员来行访问
	cout << A()._k << endl;//通过匿名对象突破类域进行访问
	return 0;
}
```

***********************



### 原始字面量

定义方式为：` R"xxx(原始字符串)xxx" `   其中（）两边的字符串可以省略。原始字面量 R 可以直接表示字符串的实际含义，而不需要额外对字符串做转义或连接等操作。

********************





### final

**final 关键字来限制某个类不能被继承，或者某个虚函数不能被重写**

1. final 修饰函数，只能修饰虚函数，这样就能阻止子类重写父类的这个函数了
2. final 关键字修饰过的类是不允许被继承的，也就是说这个类不能有派生类。

```c++
class Base
{
public:
  virtual void test()
  {
      cout<<"Base"<<endl;
  }
};

class Son final : pubilc Base		//类名后面加入final后，禁止继承Son类，class GrandSon : public Son是错误的
{
public:
    void test() final				//虚函数加入final关键字后，禁止通过继承Son类重写虚函数
    {
        cout<<"Son"<<endl;
    }
};
```

******************



### override

**override 关键字确保在派生类中声明的重写函数与基类的虚函数有相同的签名，同时也明确表明将会重写基类的虚函数，这样就可以保证重写的虚函数的正确性，也提高了代码的可读性，和 final 一样这个关键字要写到方法的后面**

``` c++
class Base
{
public:
	virtual void test()
    {
        cout<<"Base"<<endl;
    }
};

class Son : public Base
{
public:
    void test() override			//override关键字
    {
        cout<<"Son"<<endl;
    }
}
```

******************





### g++ test.cpp -o test -lpthread  -std=c++11

通过c++11标准编译，并链接libpthread.so库

********************************



### assert

` assert(expression) ` ，这是一个宏，它的参数是一个表达式，这个表达式通常返回一个布尔类型的值，并且要求表达式必须为 true 程序才能继续向下执行，否则会直接中断。



**assert 是一个运行时断言，也就是说它只有在程序运行时才能起作用**

```c++
#include <cassert>
void func()
{
    int a = 10;
    assert(a > 5);
    cout<<"Success"<<endl;
}
```



**`static_assert`所谓静态就是在编译时就能够进行检查的断言，使用时不需要引用头文件**

* 参数1：断言表达式，这个表达式通常需要返回一个 bool值
* 参数2：警告信息，它通常就是一段字符串，在违反断言（表达式为false）时提示该信息

```c++
void func()
{
    const int a = 10
    static_assert(a > 5,"Error");
    cout<<"Success"<<endl;
}
```

***由于静态断言的表达式是在编译阶段进行检测，所以在它的表达式中不能出现变量，也就是说这个表达式必须是常量表达式***

*****************************



### noexcept

异常的基本语法

![2016314153429533.jpg (577×329)](https://subingwen.cn/cpp/noexcept/2016314153429533.jpg)

**异常被抛出后，从进入 try 块起，到异常被抛掷前，这期间在栈上构造的所有对象，都会被自动析构。析构的顺序与构造的顺序相反。这一过程称为栈的解旋**

noexcept 形如其名， 表示其修饰的函数不会抛出异常 。`在 C++11 中如果 noexcept 修饰的函数抛出了异常，编译器可以选择直接调用 std::terminate () 函数来终止程序的运行，这比基于异常机制的 throw () 在效率上会高一些`。这是因为异常机制会带来一些额外开销，比如函数抛出异常，会导致函数栈被依次地展开（栈解旋），并自动调用析构函数释放栈上的所有对象。



```c++
struct Msg
{
    Msg(sting str) : msg(str){}
    string msg;
};

void func()	noexcept			
{
    throw Msg("Hello World")
}

int main()
{
    try {func();}
    catch (Msg msg)
    {
        cout<<"throw success"<<endl;
	}
}
```

noexcept 修饰符有两种形式：

1. 简单地在函数声明后加上 noexcept 关键字

2. 可以接受一个常量表达式作为参数，如下所示∶

   ```c++
   double divisionMethod(int a, int b) noexcept(常量表达式);
   ```

   常量表达式的结果会被转换成一个 bool 类型的值：

   * 值为 true，表示函数不会抛出异常
   * 值为 false，表示有可能抛出异常这里
   * 不带常量表达式的 noexcept 相当于声明了 noexcept（true），即不会抛出异常

*******************************





