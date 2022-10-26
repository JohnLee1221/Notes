

# C++ daliy notes



## 1 R_value L_value



### 1.1 左值 右值

**左值：存储在内存中，可取地址**

**右值：提供数值的数据，不可取地址（例如存储于寄存器中的数据）**，右值可以分为两种：

* 纯右值：*非引用返回的临时变量，运算表达式产生的临时变量，原始字面量，lambda表达式等*

* 将亡值：*与右值引用相关的表达式，如：T&& 类型的返回值，std::move的返回值等*

  

  

### 1.2 右值引用 &&

右值引用就是对一个右值进行引用的类型，因为右值是匿名的（**无法取地址**），所以只能通过引用的方式找到它。

* 声明一个右值引用必须进行初始化
* 通过右值引用获得的右值引用变量，右值的生命周期与右值引用变量的声明周期相同，即延长了右值的生命周期

```c++
int&& value = 520;
class Test
{
public:
    Test()
    {
        cout << "construct: my name is jerry" << endl;
    }
    Test(const Test& a)
    {
        cout << "copy construct: my name is tom" << endl;
    }
};

Test getObj()
{
    return Test();
}

int main()
{
    int a1;
    //int &&a2 = a1;        		// error
    //Test& t = getObj();   		// error
    Test && t = getObj();
    const Test& t = getObj();
    return 0;
}
```



### 1.3 堆区浅拷贝优化



```c++
class Base
{
public:
    Base():p(new int(100))
    {
        cout<<*p<<endl;
    }

    Base(const Base & b) : p(new int(*b.p))
    {
        cout<<"copy construct function"<<endl;
    }

    Base(Base && b) : p(b.p)
    {
        b.p = nullptr;
        cout<<"move construct function"<<endl;
    }

    ~Base()
    {
        delete p;
        cout<<"distruct function"<<endl;
    }

public:
    int* p = nullptr;
};

int main()
{
    //Base b1;
    //Base b2 = b1;				//使用深拷贝，堆区重复开辟内存，耗费资源和效率
    Base b3 = Base();			//移动构造，通过右值引用，使b3使用匿名对象开辟的临时内存

    return 0;
}
```

移动构造中使用了右值引用，会将临时对象中的堆内存地址的所有权转移给对象b3，这块内存被成功续命，因此在b3对象中还可以继续使用这块内存。



* **右值引用具有移动语义，移动语义可以将资源（堆、系统对象等）通过浅拷贝从一个对象转移到另一个对象这样就能减少不必要的临时对象的创建、拷贝以及销毁，可以大幅提高 C++ 应用程序的性能。**
* **对于需要动态申请大量资源的类，应该设计移动构造函数，以提高程序效率。**
* **一般在提供移动构造函数的同时，也会提供常量左值引用的拷贝构造函数，以保证移动不成还可以使用拷贝构造函数。**





### 1.4 && 的注意事项

1. **模板参数需要指定为 T &&时，&&是未定义引用类型**
2. **auto &&时，&&是未定义引用类型**
3. **const T&& 表示的是一个右值引用**






***************************



## 2 enum class

限定作用域的枚举类型(scoped enumeration)。定义限定作用域的枚举类型的一般形式是：首先是关键字[enum](https://so.csdn.net/so/search?q=enum&spm=1001.2101.3001.7020) class(或者等价地使用enum struct)，随后是枚举类型名字以及用花括号括起来的以逗号分隔的枚举成员(enumerator)列表，最后是一个分号。

枚举作用域(enumeration scope)是指枚举类型的成员的名字的作用域，起自其声明之处，终止枚举定义结束之处。C语言规定，枚举类型的成员(enumerator)的可见范围被提升至该枚举类型所在的作用域内。这被认为有可能污染了外部的作用域，为此，C++11引入了枚举类(enum class)解决此问题。

*******************



## 3 static成员

### 3.1 概念

声明为static的类成员称为类的静态成员，分为如下两类：

- 用static修饰的成员变量，称之为静态成员变量
- 用static修饰的成员函数，称之为静态成员函数

静态的成员变量一定要在类外进行初始化



### 3.2 特性

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



### 3.3 访问

​	静态成员变量的方式，当静态成员变量为公有时，可有如下三种进行访问：

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



## 4 原始字面量

定义方式为：` R"xxx(原始字符串)xxx" `   其中（）两边的字符串可以省略。原始字面量 R 可以直接表示字符串的实际含义，而不需要额外对字符串做转义或连接等操作。

********************



## 5 final	override



### 5.1 **final 关键字**

用来限制某个类不能被继承，或者某个虚函数不能被重写



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



### 5.2 voerride 关键字



确保在派生类中声明的重写函数与基类的虚函数有相同的签名，同时也明确表明将会重写基类的虚函数，这样就可以保证重写的虚函数的正确性，也提高了代码的可读性，和 final 一样这个关键字要写到方法的后面



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



## 6 assert

### 6.1 assert

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



### 6.2 static_assert

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





## 7 noexcept



### 7.1 异常的基本语法

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





### 7.2 noexcept 修饰符

​	noexcept 修饰符有两种形式:

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





## 8 setw()	setfill()

### 

```c++
cout<<right<<setfill('')<<setw(7)的解释：
```

setfill是设置填充填充字符，setw设置输出的宽度，它们的抄只作用表现在紧接着输入的字符串上。这个宽度是填充后的宽度。righ为右对齐,left为左对齐,如果字符长度不够设定的宽度就用设定的字符补充。知
所以：1，先输出A, 然后输出道7位宽的B，不足七位用*补齐。未设置对齐方式默认为右对齐。
结果：A******B
2, 左对齐的方式输出7位的A,不足7位用*补齐，再输出B。结果：A*****B
3,先输出A,再右对齐的方式输出7位的B，不足7位用*补齐。结果：A******B
4, 右对齐的方式输出7位的A,不足7位用*补齐，再输出B。结果：******AB

******************************



## 9 #if、#ifdef、#ifndef

### 9.1 #if：

```
　　#if 整型常量表达式1
 　　  程序段1
　　#elif 整型常量表达式2
   　程序段2
　　#else
　　  程序段3
　　#endif
```

　　含义：如果常量表达式1的值为真，就对程序段1进行编译；否则就计算表达式2，若为真则对程序段2进行编译，否则编译程序段3

　　用这个来调试也是个好办法 

### 9.2 #ifdef：

```
　　#ifdef 宏名
　　  程序段1
　　#else
　　  程序段2
　　#endif
```

　　含义：如果当前的宏已被定义过，则对程序段1进行编译，否则对程序段2进行编译 

### 9.3 #ifndef：

```
　　#ifndef 宏名
　　  程序段1
　　#else
　　  程序段2
　　#endif
```

　　含义：和上一个相反

***************************





## 10 auto	decltype

### 10.1 auto

#### 10.1.1 **语法**

```
auto 变量名 = 变量值;
```



#### 10.1.2 auto特性

* auto 并不代表一种实际的数据类型，只是一个类型声明的 “占位符”
* auto声明的变量必须要进行初始化，以让编译器推导出它的实际类型，在编译时将auto占位符替换为真正的类型
* auto 还可以和指针、引用结合起来使用也可以带上 const、volatile 限定符
  * 当变量不是指针或者引用类型时，推导的结果中不会保留 const、volatile 关键字
  * 当变量是指针或者引用类型时，推导的结果中会保留 const、volatile 关键字

```
auto z = 'a';       		// z 是字符型 char
auto nb;            		// error，变量必须要初始化

int temp = 110;				
auto *a = &temp;			//auto为int
auto b = &temp;				//auto为int*
auto &c = temp;				//auto为int
auto d = temp;				//auto为int

int tmp = 250;			
const auto a1 = tmp;		//auto为int
auto a2 = a1;				//auto为int（a2的数据类型为int，但是a2没有声明为指针或引用因此const属性被去掉）
const auto &a3 = tmp;		//auto为int
auto &a4 = a3;				//auto为const int
```



#### 10.1.3 auto的限制

1. 使用 auto 的时候必须对变量进行初始化

2.  auto 不能在函数的参数中使用，只有在函数调用的时候才会给函数参数传递实参，auto 要求必须要给修饰的变量赋值，因此二者矛盾。

3. auto 不能作用于类的非静态成员变量（也就是没有 static 关键字修饰的成员变量）中

   ```c++
   class Test
   {
       auto v1 = 0;                    // error
       static auto v2 = 0;             // error,类的静态非常量成员不允许在类内部直接初始化
       static const auto v3 = 10;      // ok
   }
   ```

   

4. auto 关键字不能定义数组，但可以定义指针，比如下面的例子就是错误的

   ```c++
   int array[] = {1,2,3,4,5};  // 定义数组
       auto t1 = array;            // ok, t1被推导为 int* 类型
       auto t2[] = array;          // error, auto无法定义数组
       auto t3[] = {1,2,3,4,5};;   // error, auto无法定义数组
   ```

   

5.  auto无法推导出模板参数

   ```c++
   template <typename T>
   struct Test{}
   int func()
   {
       Test<double> t;
       Test<auto> t1 = t;           // error, 无法推导出模板类型
       return 0;
   }
   ```

   

#### 10.1.4 auto的应用

1. 用于STL的容器遍历

2. 用于泛型编程

   ```c++
   class T1
   {
   public:
       static int get() {return 10;}
   };
   
   class T2
   {
   public:
       static string get() {return "hello, world";}
   };
   
   #if false
   template <class A>
   void func1(void)
   {
       auto val = A::get();				//使用auto自动类型推导
       cout << "func1  val: " << val << endl;
   }
   
   int main()
   {
       func1<T1>();
       func1<T2>();
       return 0;
   }
   
   #else
   template <class A, typename B>        // 不使用auto，添加了模板参数 B
   void func2(void)
   {
       B val = A::get();
       cout << "func2  val: " << val << endl;
   }
   
   int main()
   {
       func2<T1,int>();
       func2<T2,string>();
       return 0;
   }
   #endif
   ```

   

### 10.2 decltype

#### 10.2.1 语法

decltype 是 “declare type” 的缩写，意思是 “声明类型”。decltype 的推导是在编译期完成的，它只是用于表达式类型的推导，并不会计算表达式的值

```
decltype (expression)
```



#### 10.2.2 decltype的特性

1. 表达式为普通变量或者普通表达式或者类表达式，在这种情况下，使用 decltype 推导出的类型和表达式的类型是一致的
2. 表达式是函数调用，使用 decltype 推导出的类型和函数返回值一致
3. 表达式是一个左值，或者被括号 ( ) 包围，使用 decltype 推导出的是表达式类型的引用（如果有 const、volatile 限定符不能忽略）





#### 10.2.3 decltype的应用

decltype 的应用多出现在泛型编程中。比如我们编写一个类模板，在里边添加遍历容器的函数

```c++
template <class T>
class Container
{
public:
    void func(T& c)
    {
        for (m_it = c.begin(); m_it != c.end(); ++m_it){cout << *m_it << " ";}
        cout << endl;
    }
private:
    ??? m_it;  // 这里不能确定迭代器类型
};

int main()
{
    const list<int> lst;
    Container<const list<int>> obj;
    obj.func(lst);
    return 0;
}
```

使用	**decltype**

```c++
template <class T>
class Container
{
public:
    void func(T& c)
    {
        for (m_it = c.begin(); m_it != c.end(); ++m_it){cout << *m_it << " ";}
        cout << endl;
    }
private:
    decltype(T().begin()) m_it;  // 这里不能确定迭代器类型
};

int main()
{
    const list<int> lst{ 1,2,3,4,5,6,7,8,9 };
    Container<const list<int>> obj;
    obj.func(lst);
    return 0;
}
```



### 10.3 返回值类型后置

#### 10.3.1 语法

返回类型后置语法，说明白一点就是将decltype和auto结合起来完成返回类型的推导

```c++
// 符号 -> 后边跟随的是函数返回值的类型
auto func(参数1, 参数2, ...) -> decltype(参数表达式)
```



#### 10.3..2 实例

```c++
template <typename T, typename U>
// 返回类型后置语法
auto add(T t, U u) -> decltype(t+u) 
{
    return t + u;
}

int main()
{
    int x = 520;
    double y = 13.14;
    // auto z = add<int, double>(x, y);
    auto z = add(x, y);		// 简化之后的写法
    cout << "z: " << z << endl;
    return 0;
}
```



## 11 函数指针和回调函数

### 11.1 函数指针

函数指针指向的是函数而非对象，和其他指针一样，函数指针指向某种特定类型



#### 11.1.1 **定义**

声明指针时，必须指定指针指向的数据类型，同样，声明指向函数的指针时，必须指定指针指向的函数类型，这意味着声明应当指定函数的返回类型以及函数的参数列表

```
double cal(int);   			// prototype
double (*pf)(int);   		// 指针pf指向的函数， 输入参数为int,返回值为double,函数指针的定义
pf = cal;    				// 指针赋值
```

**参数传递**

如果将指针作为函数的参数传递：

```c++
string test(int a)							//test函数
{
    return "Hello World";
}

void foo1(int num,string (*ptr)(int))
{
    cout<<num<<"\t"<<ptr(num)<<endl;
}

void foo2(string (*ptr)(int))
{
    int num = 2;
    cout<<num<<"\t"<<ptr(num)<<endl;
}

void func1()
{
    foo1(1,test);
    foo2(test);
}
```

**函数指针调用函数**

```c++
double y = cal(5);   	// 通过函数调用
double y = (*pf)(5);   	// 通过指针调用 推荐的写法 
double y = pf(5);     	// 这样也对， 但是不推荐这样写
```



#### 11.1.2 typedef或者using

```c++
    void test(int);					

    void (*ptr1)(int) = test;		//默认定义

    void (*ptr2)(int);
    ptr2 = test;

    typedef void(*Ptr_)(int);
    Ptr_ ptr3 = test;

    using Ptr = void(*)(int);
    Ptr ptr4 = test;
```



### 11.2 回调函数

应用场景很重要

```c++
ara::core::Result<size_t> GetNewSamples(new_samples_callback&& f,size_t maxNumberOfSamples = std::numeric_limits<size_t>::max()) override
{
    ...
	f(std::move(dataPtr));
    ...
}

auto callback = [&logMsg](auto sample){...};

m_proxy->brakeEvent.GetNewSamples(callback);
```



```c++
    auto callback = [&logMsg](auto sample) {
        logMsg << "Polling: BrakeEvent - Radar:";
        // always returns NotAvailable if E2E is disabled

        if (!sample->active) 
        {
            logMsg << "NOT";
        }
        logMsg << "active";

        auto const& l_dataVector = sample->objectVector;

        if (!l_dataVector.empty()) 
        {
            logMsg << " data: ";
            ara::core::Span<std::uint8_t const> sp_object_vector(l_dataVector);
            logMsg << ara::core::as_bytes(sp_object_vector);
        }

        auto e2eCheckStatus = ara::com::e2e::internal::GetProfileCheckStatus(sample);
        bool sampleCheckStatusResult = (e2eCheckStatus == ara::com::e2e::ProfileCheckStatus::kNotAvailable);
        logMsg << "E2E checkStatus:" << (sampleCheckStatusResult ? "ok (NotAvailable)" : "not ok");
        logMsg.Flush();
    };
    // execute callback for every samples in the context of GetNewSamples
    m_proxy->brakeEvent.GetNewSamples(callback);
```



```c++
    ara::core::Result<size_t> GetNewSamples(new_samples_callback&& f,
                                            size_t maxNumberOfSamples = std::numeric_limits<size_t>::max()) override
    {
        std::lock_guard<std::mutex> guard(lock_);

        if (!samplesCache_.AvailableSize()) {
            return ara::core::Result<size_t>::FromError(ara::com::ComErrorDomainErrc::kMaxSamplesExceeded);
        }

        size_t toReadNumber = std::min(samplesCache_.AvailableSize(), maxNumberOfSamples);
        size_t passedToUserSamplesNumber = 0;

        auto samples = reader_->Read(toReadNumber);
        for (auto& sample : samples) {
            auto data = ConvertFromIdl(sample.data);
            auto dataPtr = samplesCache_.Add(data, ara::com::e2e::ProfileCheckStatus::kCheckDisabled);
            if (dataPtr) {
                f(std::move(dataPtr));
                passedToUserSamplesNumber++;
            }
        }

        return passedToUserSamplesNumber;
    }
```



## 12 constexpr

### 12.1 const 关键字

**特点**

* 修饰常量
* 只读变量

```c++
const int a = 10;

void func(const int num);
```



### 12.2 constexpr 关键字

#### 12.2.1 定义

constexpr是用来修饰常量表达式的。常量表达式，就是指有多个常量(值不会变)组成的，并且在编译阶段就计算得到结果的表达式。

constexpr修饰的常量表达式，可以提高程序的执行效率，在使用中建议将const和constexpr的功能区分：



***只读***		使用		***const***

***常量***		使用		***constexpr***



#### 12.2.1 使用

* 在定义常量时，const和constexpr是等价的，都可以在编译阶段计算结果

  ```c++
  const int m = f();  	// 不是常量表达式，m的值只有在运行时才会获取。
  const int i=520;    	// 是一个常量表达式
  const int j=i+1;    	// 是一个常量表达式
  
  constexpr int i=520;    // 是一个常量表达式
  constexpr int j=i+1;    // 是一个常量表达式
  ```

* c++内置数据类型，可以直接用constexpr修饰；自定义数据类型（class或者struct），不可以直接用constexpr修饰

  ```c++
  // 此处的constexpr修饰是无效的
  constexpr struct Test				//error
  {
      int id;
      int num;
  };
  
  ```

  ```c++
  //通过实例化对象，并对实例化的对象进行constexpr限定
  struct Test
  {
  	int id;
  	int num;
  }
  
  void func()
  {
      constexpr Test t{1,2};
  }
  ```

  

### 12.3 常量表达式

​	在程序中，使用constexpr修饰函数的返回值，这种函数被称作为常量表达式

​	

**主要包括以下函数**

* 普通函数/类成员函数

* 类的构造函数

* 模板函数

  

#### 12.3.1 修饰普通函数/类成员函数

1. 函数必须要有返回值，并且return返回的表达式必须是常量表达式

2. 函数在使用之前，必须有对应的定义语句

3. 整个函数的函数提中，不能出现非常量表达式之外的语句（using、typedef、static_assert、return除外）

   ```c++
   constexpr int func()
   {
       using mytype = int;
       constexpr mytype a = 100;
       constexpr mytype b = 10;
       constexpr mytype c = a * b;
       return c - (a + b);
   }
   ```

   

#### 12.3.2 修饰模板函数

constexpr 修饰的模板函数实例化结果不满足常量表达式函数的要求，则 constexpr 会被自动忽略，即该函数就等同于一个普通函数。



#### 12.3.3 修饰构造函数

constexpr可以修饰构造函数，这样就得到一个常量构造函数

* 构造函数的函数体必须为空
* 必须采用初始化列表的方式为各个成员赋值

```c++
#include <iostream>
using namespace std;

struct Person {
    constexpr Person(const char* p, int age) :name(p), age(age){}
    const char* name;
    int age;
};

int main()
{
    constexpr struct Person p1("luffy", 19);
    cout << "luffy's name: " << p1.name << ", age: " << p1.age << endl;
    return 0;
}
```



## 13 typedef using

### 13.1 typedef

​	通过 typedef 重定义一个类型，也称作给一个类型起别名



​	**定义别名**

```c++
//typedef 旧的类型名 新的类型名

void typedef_test()
{
    typedef unsigned int uint_t;
    typedef int (*Ptr)(int);

    uint_t a = 10;
    Ptr p = [](int b){return b;};

    cout<<a<<endl;
    cout<<p(1000)<<endl;
}

int main()
{
    typedef_test();
    return 0;
}
```



**模板的别名**

typedef不能直接给模板起别名，需要添加一个外敷类

```c++
template <typename T>
typedef map<int,T> myMap;		//错误的
```

```c++
template typename T>				//使用typedef很复杂
struct MyMap
{
    typedef map<int,T> myMap;
};

MyMap<string>::myMap m;				//调用也很怪
```





### 13.2 using

* using 用于声明命名空间，使用命名空间也可以防止命名冲突
* 使用using（别名声明 (alias declaration)） 来定义类型的别名

```c++
using 新的类型 = 旧的类型
    
using uint_t = unsigned int;		
using Ptr = int (*)(int);

template <typename T>				//可以直接给模板起别名，使用using很简洁
using map<int,T> myMap;
```







## 14 委托构造	继承构造

### 14.1 委托构造

委托构造函数允许使用同一个类中的一个构造函数调用其它的构造函数，从而简化相关变量的初始化

```c++
class Test
{
public:
    Test() {};
    Test(int max)
    {
        this->m_max = max > 0 ? max : 100;
    }

    Test(int max, int min):Test(max)
    {
        this->m_min = min > 0 && min < max ? min : 1;
    }

    Test(int max, int min, int mid):Test(max, min)
    {
        this->m_middle = mid < max && mid > min ? mid : 50;
    }

    int m_min;
    int m_max;
    int m_middle;
};
```

* 这种链式的构造函数调用不能形成一个闭环（死循环），否则会在运行期抛异常。
* 如果要进行多层构造函数的链式调用，**建议将构造函数的调用的写在初始列表中**，而不是函数体内部，否则编译器会提示形参的重复定义。
* 在初始化列表中调用了代理构造函数初始化某个类成员变量之后，就不能在初始化列表中再次初始化这个变量了。

```c++
// 错误, 使用了委托构造函数就不能再次m_max初始化了
Test(int max, int min) : Test(max), m_max(max)
{
    this->m_min = min > 0 && min < max ? min : 1;
}
```





### 14.2 继承构造

继承构造函数可以让派生类直接使用基类的构造函数，而无需自己再写构造函数，尤其是在基类有很多构造函数的情况下，可以极大地简化派生类构造函数的编写



**定义**

` using 类名::构造函数名` （其实类名和构造函数名是一样的）来声明使用基类的构造函数，这样子类中就可以不定义相同的构造函数了，直接使用基类的构造函数来构造派生类对象



```c++
class Base
{
public:
    Base(string name,int age) : m_name(name),m_age(age)
    {
        cout<<"name: "<<m_name<<"\t"<<"age: "<<m_age<<endl;
    }
    
    void func(int num)
    {
        cout<<"the function of base"<<endl;
    }
        
public:
    string m_name;
    int m_age;
};

class Derived : public Base
{
public:
    Derived(string name,int age) : Base(name,age){}				//传统继承基类的构造函数写法
    //using Base::Base;											//使用using来声明使用基类的构造函
	
 	void func()
    {
        cout<<"the function of derived"<<endl;
    }
    using Base::func;											//使用using来声明使用基类的同名函数
};

int main()
{
  Base b("Tom",18);												//base类构造
  Derived d("Jerry",12);										//通过继承base类的构造来构造derive类
  
  b.func(1);													//通过base类的对象调用base类func方法
  d.func();														//通过derived类的对象调用derived类的方法
  
  d.Base::func(1);												//通过derived类的对象调用base类的方法
  d.func(1);													//使用using关键字后，通过derived类的对象调用base类的方法
    
  return 0;
}
```





## 15 列表初始化

### 15.1 {}

使用{}来列表初始化

~~~c++
class Test
{
public:
    Test (int a){cout<<"construct the Test , value is : "<<a<<endl;}

    Test (const Test &t)
    {
        cout<<"test"<<endl;
    }
};

Test func(int num)
{
    return {num};                   //直接返回了一个匿名对象
}

Test func2()
{
    Test t(333);
    return t;
}

int main()
{
    Test t1(1);                   //普通构造
    Test t2 = 2;                  
    Test t5 = t1;                   //拷贝构造

    //列表初始化
    Test t3 = { 3 };
    Test t4{ 4 };
    Test t6 = func(100);
    Test t7 = func2();

    int a1 = { 1314 };
    int a2{ 1314 };
    int arr1[] = { 1, 2, 3 };
    int arr2[]{ 1, 2, 3 };

    return 0;
}
~~~



### 15.2 std::initializer_list



```c++
//std::initializer_list<数据类型> 名称{...} 
std::initializer_list<int> init_list{1，2，3}；
```



在 C++ 的 STL 容器中，可以进行任意长度的数据的初始化，使用初始化列表也只能进行固定参数的初始化，如果想要做到和 STL 一样有任意长度初始化的能力，可以使用 std::initializer_list 这个轻量级的类模板来实现。



* std::initializer_list拥有一个无参构造函数，可以直接定义实例，此时将得到一个空的std::initializer_list
* 是一个轻量级的容器类型，内部定义了迭代器 iterator 等容器必须的概念，遍历时得到的迭代器是只读的。
* 对于 std::initializer_list<T> 而言，它可以接收任意长度的初始化列表，但是要求元素必须是同种类型 T
* 在 std::initializer_list 内部有三个成员接口：size(), begin(), end()。
* std::initializer_list 对象只能被整体初始化或者赋值
* ***std::initializer_list的效率是非常高的，它的内部并不负责保存初始化列表中元素的拷贝，仅仅存储了初始化列表中元素的引用。**



#### 15.2.1 作为普通函数参数

```c++
void func (std::initializer_list<int> a)
{
    ....
}

int main()
{
    func({1,2,3,4,5});
    
    std::initializer_list<int> b{5,4,3,2,1};
    func(b);
        
    return 0;
}
```



#### 15.2.2 作为构造函数参数

```c++
class Test
{
public:
    Test(std::initializer_list<string> list)
    {
	...
    }
};

int main()
{
    Test t({ "jack", "lucy", "tom" });
    Test t1({ "hello", "world", "nihao", "shijie" });
    return 0;
}
```





## 16 function bind



### 16.1 **可调用对象**

在C++语言中有几种可调用对象(c++ primer P511)

* ***函数***
* ***函数指针***
* ***lambda表达式***
* ***bind创建的对象***
* ***重载了函数调用运算符的类***



示例如下场景

```c++
//函数指针
//具有operator()成员函数的类对象（仿函数）
//可被转换成函数指针的类对象
//类成员函数指针或者类成员指针

#include <iostream>
using namespace std;

typedef void (*pf)(string,int);                                 //定义一个函数指针类型

class Base
{
public:
    static void test01(string name,int age)     //静态成员函数
    {
        cout<<name<<"\t"<<age<<endl;
    }

    void test02(int a)
    {
        cout<<a<<endl;
    }

    //仿函数
    void operator()(string str) {cout<<str<<endl;}                          //可调用对象，2.仿函数

    //把类对象转换成一个函数指针    被转换的函数要求是静态函数
    operator pf() {return test01;}                                          //可调用对象，3.转换成函数指针的类对象

public:
    int m_num = 10;
};

void func1()
{
    pf p = [](string a,int b){cout<<"没有捕获列表的函数指针"<<endl;};          //可调用对象，1.函数指针
    p("hahaha",1);
}

void func2()
{  
    //可调用对象，4.类成员函数指针或者类成员变量指针
    Base b;
    void (Base::*pf_01)(int) = &Base::test02;                                 //4.1 类成员函数指针
    (b.*pf_01)(1);                                                            //调用

    using Pf_02 = void (Base::*)(int);                                       //使用using        
    Pf_02 pf_02 = &Base::test02;
    (b.*pf_02)(2);

    int Base::*pf_03 = &Base::m_num;                                          //4.2 类成员变量指针
    cout<<b.*pf_03<<endl;
}

int main()
{
    func1();
    func2();

    return 0;
}
```



### 16.2 function

**std::functinon是可调用对象包装器，它是一个模板类，可以容纳除了类成员指针（包括：*类成员指针，类函数指针*）以外的所有可调用对象**。通过指定模板参数，可以用统一的方式处理函数、函数指针、函数对象，丙允许保存和延迟执行。



#### 16.2.1 用法

```c++
#include <functional>
std::function<返回值类型（参数类型列表）> = 可调用对象；
```



```
#include <iostream>
#include <functional>
using namespace std;

typedef void (*Pf)(string,int);                                 //定义一个函数指针类型

void func1(string str,int num)                                  //可调用对象：普通函数
{
    cout<<str<<"\t"<<num<<endl;
}

Pf p_func1 = func1;                                                 //可调用对象：函数指针

class Base
{
public:
    static void test01(string name,int age)     //静态成员函数
    {
        cout<<name<<"\t"<<age<<endl;
    }

    void test02(int a)
    {
        cout<<a<<endl;
    }

    //仿函数
    void operator()(string str) {cout<<str<<endl;}                          //可调用对象，2.仿函数

    //把类对象转换成一个函数指针    被转换的函数要求是静态函数
    operator Pf() {return test01;}                                          //可调用对象，3.转换成函数指针的类对象

public:
    int m_num = 10;
};

int main()
{
    function<void(string,int)> function_01 = func1;
    function<void(string,int)> function_02 = p_func1;

    function_01("普通函数绑定",01);
    function_01("函数指针绑定",02);

    Base b;
    function<void(string)> function_03 = b;
    function<void(string,int)> function_04 = Base::test01;
    function_03("仿函数绑定");
    function_04("静态成员函数绑定",3);

    function<void(string,int)> function_05 = b;
    b("类转换成函数指针绑定",4);

    return 0;
}
```



### 16.3 bind

**std::bind用来将可调用对象与其参数一起进行绑定。绑定后的结果可以使用std::function进行保存，并延迟调用到任何我们需要的时候。**

**作用**

1. 将可调用对象与其参数绑定成一个***仿函数***
2. 将多元（参数个数为n，n>1）可调用对象转换为一元或者（n-1）元可调用对象，即只绑定部分参数

#### 16.3.1 用法

```c++
// 绑定非类成员函数/变量
auto f = std::bind(可调用对象地址, 绑定的参数/占位符);
// 绑定类成员函/变量
auto f = std::bind(类函数/成员地址, 类实例对象地址, 绑定的参数/占位符);
```





```c++
typedef void (*Pf)(string,int);                                 //定义一个函数指针类型

void func1(string str,int num)                                  //可调用对象：普通函数
{
    cout<<str<<"\t"<<num<<endl;
}

Pf p_func1 = func1;                                                 //可调用对象：函数指针

class Base
{
public:
    static void test01(string name,int age)     //静态成员函数
    {
        cout<<name<<"\t"<<age<<endl;
    }

    void test02(int a)
    {
        cout<<a<<endl;
    }

    //仿函数
    void operator()(string str) {cout<<str<<endl;}                          //可调用对象，2.仿函数

public:
    int m_num = 10;
};

int main()
{
    auto f1 = bind(func1,placeholders::_1,placeholders::_2);                                
    std::function<void(string,int)> f1_1 = bind(func1,placeholders::_1,placeholders::_2);     //这两种方法都可以
    f1("普通函数绑定调用",1);

    Base b;
    auto f2 = bind(b,placeholders::_1);                                        
    f2("仿函数绑定调用");
}
```



可调用对象包装器 std::function 是不能实现对类成员函数指针或者类成员指针的包装的，但是通过绑定器 std::bind 的配合之后，就可以完美的解决这个问题了

```c++
class Test
{
public:
    void output(int x, int y)
    {
        cout << "x: " << x << ", y: " << y << endl;
    }
    int m_number = 100;
};

int main(void)
{
    Test t;
    // 绑定类成员函数
    function<void(int, int)> f1 = 
        bind(&Test::output, &t, placeholders::_1, placeholders::_2);
    // 绑定类成员变量(公共)
    function<int&(void)> f2 = bind(&Test::m_number, &t);

    // 调用
    f1(520, 1314);
    f2() = 2333;
    cout << "t.m_number: " << t.m_number << endl;

    return 0;
}
```



## 17 拷贝构造 constructor



**拷贝构造调用时机** 

* 使用一个对象给另一个新建对象初始化
* 以值传递的方式，传递类的对象，生成一个临时对象进行拷贝
* 以返回值的方式，返回一个类对象，生成一个临时对象进行拷贝

实际过程中，以返回值的方式返回一个类对象，不会调用拷贝构造，***是因为RVO（return value optimization），被G++进行值返回的优化了***



**-fno-elide-constructors**

```c++
//关闭RVO
g++ test.cpp -o test -fno-elide-constructors
```

